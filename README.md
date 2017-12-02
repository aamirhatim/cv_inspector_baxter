## Inspector Baxter: Computer Vision Component
#### Aamir Husain, Peng Peng
#### ME495 - Embedded Systems in Robotics

# Project Overview
The objective of this project is to implement a ROS program that trains Baxter to recognize an object among a number of objects on a table, sort them, and then retrieving an item upon request. The user controls what object Baxter should pick up via voice commands.

This repository contains the code which will distinguish objects on a plane and categorize them based on a user input.

# Computer Vision Component
### Packages Used
* [`perception_pcl`](https://github.com/ros-perception/perception_pcl.git) (kinetic-devel branch)
* [`openni2_camera`](https://github.com/ros-drivers/openni2_camera) (indigo-devel branch)
* [`rqt_reconfigure`](https://github.com/ros-visualization/rqt_reconfigure.git) (optional, used mostly for debugging)

*This [Github repository](https://github.com/NU-MSR/nodelet_pcl_demo) was used as a framework to build the CV components of this project.*

### Overview
1. Use `perception_pcl` to compute point clouds of various objects on the table. The nodelets in this package allow for individual point cloud extraction of multiple objects as well as plane recognition.

2. An [ASUS XtionPRO LIVE](https://www.asus.com/us/3D-Sensor/Xtion_PRO_LIVE/) is used to view Baxter's environment. This sensor was chosen over other depth sensing devices like the Kinect because of its relative ease of use. Point cloud locations in 3D space will be discovered and the centroid of each object is saved in a list that the grabbing component of this project will be able to access. Because the `perception_pcl` package allows for centroid identification of different 3D shapes, this feature can be used to assign Baxter a set of pre-programmed grasping methods for different object shapes.

### Running the Code
*Make sure all required packages are properly installed and the XtionPRO LIVE is pluged in!*
To run the code, use the following command:
```
$ roslaunch openni2_launch openni2.launch
```
In another terminal, run the following command:
```
$ roslaunch cv_inspector_baxter pcl_extract.launch
```
**NOTE:** The second command takes two arguments: `show_pcl` and `gui` which both have default value of "false." To view the extracted point clouds, set `show_pcl` to "true". Set `gui` to "true" to change point cloud filtering parameters in real time.

### Setting Up the XtionPRO LIVE
1. Clone `openni2_camera` package (link is above) and run `catkin_make` from your workspace directory.
2. Connect the device to start publishing to the `/camera` topics:
```
roslaunch openni2_launch openni2.launch
rosrun rviz rviz
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

* **StatisticalOutlierRemoval** removes any random stray points to produce cleaner data.

The filtered data is then passed to a [node](src/cluster_extractor.cpp) that extracts clusters of point clouds. The node is hard coded to look for a maximum of 4 clusters (objects). *See this [tutorial](http://pointclouds.org/documentation/tutorials/cluster_extraction.php#cluster-extraction) for information on the cluster extraction method.*

Once the point clouds are extracted, the centroids and point cloud indexes are posted on two separate topics for each object. Centroids are published to `/cluster_<OBJECT NUMBER>_point` and point cloud indexes are published to `/cluster_<OBJECT NUMBER>_cloud`.

Cluster centroid example ([**PointStamped**](http://docs.ros.org/api/geometry_msgs/html/msg/PointStamped.html) message type):
```
header:
  seq: 1065
  stamp:
    secs: 1512177347
    nsecs: 991435043
  frame_id: camera_depth_optical_frame
point:
  x: 0.229352176189
  y: -0.0985339954495
  z: 0.861864626408
```

Cluster point cloud example ([**PointCloud2**](http://docs.ros.org/jade/api/sensor_msgs/html/msg/PointCloud2.html) message type):
```
header:
  seq: 4287
  stamp:
    secs: 0
    nsecs: 0
  frame_id: camera_depth_optical_frame
height: 1
width: 812
fields:
  -
    name: x
    offset: 0
    datatype: 7
    count: 1
  -
    name: y
    offset: 4
    datatype: 7
    count: 1
  -
    name: z
    offset: 8
    datatype: 7
    count: 1
is_bigendian: False
point_step: 16
row_step: 12992
data: []
```

### Working with Extracted Point Cloud Data
Once the objects have been identified, another [node]() creates an averaged approximation of each object's height, width, and centroid. This reduces the chance of using an inaccurate data point when Baxter goes to reach for an object.
