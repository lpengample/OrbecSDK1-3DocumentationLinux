# 3D camera/depth camera
Measurements are made through different principles (structured light, iToF, stereo vision), and depth maps can be generated by processing these measured values. The depth map is a set of Z coordinate values of each pixel of the image, in millimeters.
# Gyroscope and acceleration sensor (IMU)
A device for measuring the three-axis attitude angle and acceleration of an object generally, includes a three-axis gyroscope and a three-axis accelerometer.
# Coordinate system
This article introduces 2D and 3D coordinate systems and the relationship between these coordinate systems.
## 2D coordinate system
The depth camera and the color camera are associated with a separate 2D coordinate system. The [x,y] coordinates are in pixels, where x ranges from 0 to width -1 and y ranges from 0 to height -1. The width and height depend on the selected depth camera and color camera working mode. The pixel coordinate [0,0] corresponds to the upper left pixel of the image.  

![1](./Orbbec%20SDK%20Basic%20Concepts%20img/1.png)

## Camera intrinsic parameters
The relationship between the two-dimensional and three-dimensional coordinate systems of the camera's data stream is described by its intrinsic parameters. Each device is different, image can be any size - the width and the height represent the number of columns and rows respectively in the image. The brief description follows below: 1. The field of view of the image can be changed respectively the fx and fy fields that describe the focal length of the image, which are multiples of the pixel width and height. The pixels of the image are not necessarily square, and fx and fy are allowed to be different (although they are usually similar). 2. Describes the pixel coordinates of the principal point (projection center)<img src="./Orbbec%20SDK%20Basic%20Concepts%20img/2.png" alt="2" style="zoom:50%;" />
## Camera extrinsic parameters
In general, the three-dimensional coordinate system of the data stream of different cameras is different. For example, the depth data stream generated by one or more infrared cameras, while the color stream is provided by a separate color camera. The relationship between the three-dimensional coordinate systems of different data streams is described by their extrinsic parameters. The brief description follows below: 1. Cameras may be located in different locations, but fixed on the same physical device. The displacement information describes the three-dimensional displacement conversion between the physical positions of cameras [x, y, z], in meters. 2. Cameras may have different directions, but fixed on the same physical device. The rotation information contains a 3x3 orthogonal rotation matrix that describes the physical direction between the cameras.
## Distortion model
The lens distortion is mainly divided into radial distortion and tangential distortion. Radial distortion: It is caused by the manufacturing process of the lens shape, and the radial distortion becomes more serious as it moves to the edge of the lens. In actual situations, we often use the first few terms of the Taylor series expansion at r=0 to estimate the radial distortion. Tangential distortion: It is caused by the installation position error of the lens and CMOS or CCD. Tangential distortion requires two additional distortion parameters to describe. The normalized coordinates after radial distortion and tangential distortion are collated to require a total of 5 distortion parameters<img src="./Orbbec%20SDK%20Basic%20Concepts%20img/3.png" alt="3" style="zoom:50%;" />
# Point cloud
The point cloud generated from the depth image is a set of points in the three-dimensional coordinate system of the depth stream. It is also possible to create point clouds with corresponding texture-mapped from both depth and color streams.
# Frame alignment
Usually when processing color and depth images, each pixel needs to be mapped from one image to another. To achieve image alignment, a set of frames with the same resolution is generated, and after processing, the pixels can be simply mapped. Our alignment methods are divided into hardware alignment and software alignment. Generally speaking, hardware alignment can get better performance than software alignment.
# Frame synchronization
Different data streams will cause data frames to be out of sync because of unsynchronized data collection and system processing. Through the time stamp of the frame data and corresponding data processing, it is possible to ensure as far as possible that the time difference between the two types of different data is within a certain controllable range, which is the frame synchronization function. However, it should be noted that the system often cannot guarantee a strict frame synchronization, i.e., the time difference between two frames is 0.
