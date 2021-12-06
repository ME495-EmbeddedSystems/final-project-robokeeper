## RoboKeeper!
#### Jonny Bosnich, Joshua Cho, Lio Liang, Marco Morales, Cody Nichoson 
****
<p align="center">
<img src="https://user-images.githubusercontent.com/62906322/144802674-e79eaead-e5f2-4155-9ca0-5d0346366a89.gif" alt="Robokeeper being a boss height="270" width="480" />

 </p>

### Equipment
##### Hardware:
- HDT Global Adroit Manipulator Arm
- Intel RealSense Camera
##### Software:
- Robot Operating System (ROS)
- MoveIt!
- OpenCV

### Dependencies

### Quickstart Guide
0. Install ROS Noetic on Ubuntu 20.04
1. Create catkin workspace
    ```
    $ source /opt/ros/noetic/setup.bash
    $ mkdir -p ~/catkin_ws/src
    $ cd ~/catkin_ws/
    $ catkin_make
    ```
2. Copy this repository into `src` folder
    ```
    $ cd ~/catkin_ws/src
    $ git clone git@github.com:ME495-EmbeddedSystems/final-project-robokeeper.git
    ```
3. Install required packages and build
    ```
    $ source devel/setup.bash
    $ rosdep install --from-paths src --ignore-src -r -y
    $ catkin_make
    ```
****
### Launchfiles
#### robokeeper_go.launch
This is the main launchfile used to operate robokeeper. It starts by launching `robokeeper_moveit.launch` which loads the necessary urdf file and hardware configuration, as well as the main `MoveIt!` executable. It then launches `intel_cam.launch` which starts the Intel Realsense camera. It also starts a `transforms` node which handles the calculation of transformation between various frames within the world. Finally, the launchfile starts a `motion_control` node that publishes appropriate joint state messages to actuate the arm. 

#### robokeeper_moveit.launch
This launchfile loads robot description for the Adroit 6-dof manipulator arm, as well as its hardware and controller configuration from the `hdt_6dof_a24_pincer_description` package. It also includes `move_group.launch` from the `hdt_6dof_a24_pincer_moveit` package, which starts the move group that `MoveIt!` uses to plan the motion of the arm.

#### intel_cam.launch
This launchfile starts the Intel Realsense camera by launching `rs_camera.launch` from the `realsense2_camera` package. It then launches `AprilTag_detection.launch` for AprilTag integration.

#### AprilTag_detection.launch
This launchfile loads parameters necessary for integrating AprilTag detection, which is crucial for detecting the position of the robot relative to the camera. It starts `apriltag_ros_continuous_node` from the `apriltag_ros` package.

## Nodes
#### perception
The perception node is responsible for handling the data collected from the Intel RealSense camera utilized to identify and locate the objects that our robot is tasked with blocking. It contains a CV bridge to enable OpenCV integration with ROS, subscribes to the RealSense's camera data, and ultimately publishes 3-dimensional coordinate data of the centroid of the object of interest (a red ball for our purposes).

In order to identify the ball, video frames are iteratively thresholded for a range of HSV values that closely match those of our ball. Once the area of interest is located, a contour is created around its edges and the centroid of the contour located. This centroid can then be treated as the location of the ball in the camera frame and published appropriately.

#### transforms
Knowing where the ball is relative to the camera is great, but it doesn't help the robot locate the ball. In order to accomplish this, transformations between the camera frame and the robot frame are necessary. This node subscribes to both the ball coordinates from the perception node and AprilTag detections, and publishes the transformed ball coordinates in the robot frame.

In order to complete the relationship between the two frames, an AprilTag with a known transformation between itself and the baselink of the robot (positioned on the floor next to the robot) was used. Using the RealSense, the transformation between the camera frame and the AprilTag can then also be determined. Using these three frames and their relationships, the transformation between coordinates in the camera frame and coordinates in the robot frame can finally be determined.

#### motion_control
This node provides the core functionality of the robokeeper.Primarily, it subscribes to the topic containing the ball coordinates in the robot frame and contains a number of services utilized to interact with its environment in several ways.

The main service used is /start_keeping. As the name suggests, this service allows the robot to begin interpreting the ball coordinates and attempting to intersect it at the goal line. Appropriate joint trajectory commands are sent to the robot through a mix of MoveIt! and direct joint publishing (depending on the service called) in order to accomplish the task. This node also keeps track of goals scored by determining if the ball has entered the net.

### System Architecture

