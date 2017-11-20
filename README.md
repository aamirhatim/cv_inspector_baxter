## Inspector Baxter: Computer Vision Component
#### Aamir Husain
#### ME495 - Embedded Systems in Robotics

# Project Overview
The objective of this project is to implement a ROS program that trains Baxter to recognize an object among a number of objects on a table, and then retrieving it upon request. The user controls what object Baxter should pick up from simple speech input.

This repository contains the code which will distinguish objects on a plane and categorize them based on a user input.

# Computer Vision Component
### Packages Used
`perception_pcl`

### Overview
1. Use `perception_pcl` to compute point clouds of various objects on the table. The nodelets in this package allow for individual point cloud extraction of multiple objects as well as plane recognition.

2. A Kinect (or something similar) is used to view Baxter's environment. Point cloud locations in 3D space will be discovered and the centroid of each object is saved in a list that the grabbing component of this project will be able to access. Because the `perception_pcl` package allows for centroid identification of different 3D shapes, this feature can be used to assign Baxter a set of pre-programmed grasping methods for different object shapes.
