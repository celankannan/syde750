# Vision-Based Speaker Tracking on a REEM-C Humanoid

A humanoid robot that works out **who is speaking** from vision alone and turns its head and torso toward them.

**Course:** SYDE 750 / ME 780 Humanoid Robotics, University of Waterloo, Winter 2022
**Team:** Pranav Barot, Chandrasekar Elankannan
**Full write-up:** [project_report.pdf](./project_report.pdf)

---

## The problem

Robots that talk with people need to know who is talking and face them, even when people move or a second person joins. Audio-only methods (sound source localization) struggle when people move around. This project adds a vision-based answer on the REEM-C humanoid.

## What we built

1. **Active speaker detection from the face.** dlib's 68-point face landmarks locate the inner mouth. From an RGB-D stream we compute four features per frame: mouth opening **height**, mouth **area**, **lightness** inside the mouth (HSL space) and mouth **depth**. Each speaker's features are compared against their own resting baseline to decide whether they are speaking.
2. **Face yaw estimation.** The nose landmark's offset from the image centre, scaled to millimetres using face height and combined with depth, gives the yaw angle between the robot and the person.
3. **Whole-body response.** The yaw angle is split across the head, torso and legs, so small turns use only the head and larger turns bring in the torso and legs. With two people in view, the robot tracks the midpoint between their faces and raises a separate arm for each person it detects speaking.

Tested in two scenarios, in Gazebo simulation: one person in view, and two people in view.

## Files

| File | What it is |
|---|---|
| `face_yaw_detection_chandra.py` | Face landmark detection on a webcam + OpenNI depth stream, and yaw/pitch estimation from the nose position and depth |
| `whole_body_move.py` | ROS node that splits a target yaw across head and torso joints (≤10°: head only; 10–45°: head 60% / torso 40%; 45–90°: head 50% / torso 30% / legs 20%) and publishes `JointTrajectory` commands to `/head_controller/command` and `/torso_controller/command` |
| `algo_exp.ipynb` | Experiments on the mouth features (height, area, lightness, dark-pixel count) used to classify speech |
| `project_report.pdf` | Full report: method, control design, results |
| `A11_3_Right_leg_rviz.png` | REEM-C right-leg visualisation in RViz from course work |

## Running it

**Face yaw detection** (Python 3):

```bash
pip install numpy scipy opencv-python dlib openni
# Download the landmark model and unzip it next to the script:
#   http://dlib.net/files/shape_predictor_68_face_landmarks.dat.bz2
# or point to it with:  export LANDMARK_MODEL=/path/to/shape_predictor_68_face_landmarks.dat
python face_yaw_detection_chandra.py
```

Needs a webcam and an OpenNI-compatible depth camera. Press `Esc` to quit.

**Head and torso motion** (ROS 1 with the REEM-C simulation from PAL Robotics):

```bash
rosrun <your_package> whole_body_move.py
# enter the target yaw angle in degrees when prompted
```

## Results

From the report: speech identification worked accurately in both the single-person and the two-person scenarios, and REEM-C turned toward the speaker in simulation. The robot's response is best seen on video. No quantitative accuracy or latency figures were recorded.

## Limitations and next steps

- Tested in simulation only.
- Speech thresholds depend on each person's baseline and on frontal faces with reasonable lighting.
- Next step from the report: fuse this with audio sound-source localization so vision can correct audio results and check sound sources outside the camera's view.

## Acknowledgments

SYDE 750 / ME 780 course staff, University of Waterloo; PAL Robotics for the REEM-C model.
