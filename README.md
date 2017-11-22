## Inspector Baxter: Computer Vision Component
#### Aamir Husain
#### ME495 - Embedded Systems in Robotics

# Project Overview
The objective of this project is to implement a ROS program that trains Baxter to recognize an object among a number of objects on a table, and then retrieving it upon request. The user controls what object Baxter should pick up from simple speech input.

This repository contains the code which will distinguish objects on a plane and categorize them based on a user input.

# Computer Vision Component
### Packages Used
* [`perception_pcl`](https://github.com/ros-perception/perception_pcl.git) (kinetic-devel branch)
* [`OpenNi`](https://github.com/OpenNI/OpenNI.git)
* [`SensorKinect`](https://github.com/avin2/SensorKinect.git)

### Overview
1. Use `perception_pcl` to compute point clouds of various objects on the table. The nodelets in this package allow for individual point cloud extraction of multiple objects as well as plane recognition.

2. A Kinect (or something similar) is used to view Baxter's environment. Point cloud locations in 3D space will be discovered and the centroid of each object is saved in a list that the grabbing component of this project will be able to access. Because the `perception_pcl` package allows for centroid identification of different 3D shapes, this feature can be used to assign Baxter a set of pre-programmed grasping methods for different object shapes.

### Kinect-ing ;)
1. Clone the OpenNI repository. Make sure you clone from the **unstable** branch.

2. To set up the Kinect with the system, the following installs are required (my system already had them installed):
```
$ sudo apt-get install g++
$ sudo apt-get install libusb-1.0-0-dev
$ sudo apt-get install freeglut3-dev
$ sudo apt-get install doxygen
```
3. Once these programs are installed, run the following commands to build OpenNI.
Checkout the unstable 1.5.4.0 branch using the command: `git checkout Unstable-1.5.4.0`. Then, from the package directory:
```
$ cd Platform/Linux/CreateRedist
$ ./RedistMaker
```
This creates a redist package in `Platform/Linux/Redist`. It also creates a distribution in `Platform/Linux/CreateRedist/Final`.

4. From `Platform/Linux/Redist`, run the following commands:
```
$ cd OpenNI-Bin-Dev-Linux-x64-v1.5.4.0/
$ sudo ./install.sh
```

5. Next, install the Kinect sensor module. Make sure you clone the **unstable** branch.
Once the package is installed, run the following commands from the `SensorKinect` directory:
```
$ cd Platform/Linux/CreateRedist
$ ./RedistBuild

$ cd ../Redist/Sensor-Bin-Linux-x64-v5.1.2.1/
$ sudo ./install.sh
```

6. Test the installation by running the following commands from the `OpenNI` package installed in step 1:
```
$ cd Platform/Linux/Bin/x64-Release
$ ./Sample-NiSimpleViewer
```
