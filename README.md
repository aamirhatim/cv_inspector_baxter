## Inspector Baxter: Computer Vision Component
#### Aamir Husain
#### ME495 - Embedded Systems in Robotics

# Project Overview
The objective of this project is to implement a ROS program that trains Baxter to recognize an object among a number of objects on a table, sort them, and then retrieving an item upon request. The user controls what object Baxter should pick up from simple speech input.

This repository contains the code which will distinguish objects on a plane and categorize them based on a user input.

#Computer Vision Component
### Packages Required
* [`perception_pcl`](https://github.com/ros-perception/perception_pcl.git) (kinetic-devel branch)
* [`openni2_camera`](https://github.com/ros-drivers/openni2_camera) (indigo-devel branch)

*This [Github repository](https://github.com/NU-MSR/nodelet_pcl_demo) was used as a framework to build the CV components of this project.*

### Overview
1. Use `perception_pcl` to compute point clouds of various objects on the table. The nodelets in this package allow for individual point cloud extraction of multiple objects as well as plane recognition.

2. An [ASUS XtionPRO LIVE](https://www.asus.com/us/3D-Sensor/Xtion_PRO_LIVE/) is used to view Baxter's environment. Point cloud locations in 3D space will be discovered and the centroid of each object is saved in a list that the grabbing component of this project will be able to access. Because the `perception_pcl` package allows for centroid identification of different 3D shapes, this feature can be used to assign Baxter a set of pre-programmed grasping methods for different object shapes.

### Setting Up the XtionPRO LIVE
1. Clone `openni2_camera` package (link is above) and run `catkin_make` from your workspace directory.
2. Connect the device and start publishing to the `/camera` topics:
```
roslaunch openni2_launch openni2.launch
roslaunch rviz rviz
```
3. In `rviz`, add a new **PointCloud2** display. Within the display properties, set your topic (either `/camera/depth/points` or `/camera/depth_registered/points`). This should display a depth image of all the camera's field of view. Change your view parameters to the following:
```
Yaw: Pi (or something close enough)
Pitch: 0
Focal Point (X,Y,Z): (0,0,0)
```

*See this [tutorial](https://github.com/IntelligentRoboticsLab/KukaYouBot/wiki/Pattern-recognition-with-3D-Cameras-Microsoft-Kinect-&-ASUS-Xtion-PRO-LIVE) for more information on setting up the sensor.*

### Extracting Point Clouds and Their Locations
To filter out unwanted points, `perception_pcl` makes use of several *nodelets* that can be easily implemented in a launch file.

* **CropBox** filters out any points that are not within a specified volume. This filter was used to remove any points outside of the table with objects on it.

* **VoxelGrid** down-samples a set of points by averaging the points within a specified unit volume into a single point. This filter reduces the resolution of the data, making data processing less computationally intensive.

* **SACSegmentation** extracts models from a set of points. This filter is used to identify the table surface.

* **StatisticalOutlierRemoval** removes any random stray points.

The filtered data is then passed to a node that extracts clusters of point clouds.
