# Introduction
Orbbec Viewer is a tool developed based on Orbbec SDK to help developers quickly use Orbbec's 3D sensor products. The functional features of Orbbec 3D sensor products are realized, including functions such as reading serial numbers, obtaining device types, acquiring camera parameters, controlling cameras,etc.

This document describes these functions and demonstrates the basic usage process.
# Overview
## Target users
The OrbbecViewer tool is designed for end users/developers to use Orbbec SDK 3D cameras.
## Supported Devices and Platforms
| **SDK version** | **Product** | **Fireware version** |
| --- | --- | --- |
| v1.0.2 | Astra+ | 1.0.19 |
| v1.1.6 | Astra+ | 1.0.19/1.0.20 |
|  | Femto | 1.5.1 |
| v1.3.1 | Astra+ | 1.0.21/1.0.20/1.0.19 |
|  | Femto | 1.6.9/1.6.7 |

| **Operating system** | **Requirement** | **Description** |
| --- | --- | --- |
| Windows | - Windows 10 April 2018 (version 1803, operating system build 17134) release (x64) or higher<br />- 4 GB RAM<br />- USB2.0 and above ports<br /> | The generation of the VS project depends on the installation of the VS version and the cmake version, and supports VS2015/vs2017/vs2019 |
| Android | - Android 6/7/8/9/10<br /> |  |
| Linux | - Linux Ubuntu 16.04/18.04/20.04 (x64)<br />- 4 GB RAM<br />- USB2.0 and above ports<br /> |Support GCC 7.5|

# OrbbecViewer Usage
## 3.1 Main interface of the software
As shown in the figure, the main interface is marked with three areas separated by red boxes. The functions are:

| **Area** | **Name** | **Function** |
| --- | --- | --- |
| Area 1 | Equipment management area | Sensor list, device firmware update |
| Area 2 | Control panel area | Data stream switch and parameter configuration, image acquisition function control, D2C function control |
| Area 3 | Image preview area | Sensor image preview, frame rate, timestamp and other information viewing |

![1](./Orbbec%20Viewer%20Manual%20img/1.png)

<br />Among them, there are six buttons on the left side of the control panel area, the bottom button<br />

![2](./Orbbec%20Viewer%20Manual%20img/2.png) Used to switch between Chinese and English;![3](./Orbbec%20Viewer%20Manual%20img/3.png) Used to open the software details page.<br />![4](./Orbbec%20Viewer%20Manual%20img/4.png) Used to view device information and firmware upgrade; the remaining two buttons are used to switch the control panel TAB pages of different functions, from top to bottom, it is "Single Camera Mode" and "Advanced Mode".<br />Click the Log information at the bottom of area 3, and the specific log information will be displayed.

## 3.2 Data stream
OrbbecViewer allows the user to select and configure depth, infrared and color data streams. This section outlines the parameters that the user can customize in the tool. After the user completes the configuration, they must click the top ![5](./Orbbec%20Viewer%20Manual%20img/5.png)button to start data streaming.

The OrbbecViewer tool allows the user to select a combination of depth, infrared and color data streams. User can enable/disable the stream by checking/unchecking from the list of available streams. The viewer supports both USB2.0 and USB3.0, so the available configuration parameters will vary depending on the USB2.0/USB3.0 capabilities.<br />

![6](./Orbbec%20Viewer%20Manual%20img/6.png)

### Resolution and Frame Rate
The cameras provide users with a choice of resolutions to suit their solution needs. Each data stream provides a variety of optional frame formats for users to freely choose the appropriate combination of image size, image format and frame rate.

For example, the depth stream can support two resolutions of 320x240 and 640x480, and the range of FPS frame rate is 5 to 30.<br />![7](./Orbbec%20Viewer%20Manual%20img/7.png)<br />The color stream can support multiple resolutions from 640x480 to 1920x1080, the range of FPS frame rate ranges from 5 to 30, and supports MJPG, RGB, I420, and H265.

For the preview of the color stream, MJPG, H264, and H265 are the encoding formats, which need to be decoded first. The decoding usually takes a lot of time, resulting in dropped frames or rendering a "corrupted" picture. On some models of PC, hardware-accelerated decoding is available.<br />![8](./Orbbec%20Viewer%20Manual%20img/8.png)<br />The infrared stream can support two resolutions of 320x240 and 640x480, and the FPS frame rate ranges from 5 to 30.<br />![9](./Orbbec%20Viewer%20Manual%20img/9.png)<br />Users can choose the most suitable resolution according to the actual situation. Note that higher resolutions are more accurate, but increase computational intensity.
## 3.3 Device Information
Click the button“ ![10](./Orbbec%20Viewer%20Manual%20img/10.png) ”to view device information<br />The OrbbecViewer tool contains simple device information such as firmware version, product identification code, camera parameters, temperature, etc.<br />![11](./Orbbec%20Viewer%20Manual%20img/11.png)
## 3.4 Image preview area
Open stream will display the average frame rate, time frame, image frame number and other information by default, click on the upper right corner ![12](./Orbbec%20Viewer%20Manual%20img/12.png) to toggle information display\close.<br />Click ![13](./Orbbec%20Viewer%20Manual%20img/13.png) , the stream can be paused without being removed from the preview area.<br />Click ![14](./Orbbec%20Viewer%20Manual%20img/14.png) , streams that have been paused and not removed from the preview area can be reopened.<br />After the data stream is closed, the image preview window will not be closed, and the user needs to click ![15](./Orbbec%20Viewer%20Manual%20img/15.png) the upper right corner of the image preview window.<br />![16](./Orbbec%20Viewer%20Manual%20img\16.jpg)
## 3.5 Control Panel Area
### Sensors and Data Stream
#### Get camera name, serial number and USB protocol
After the device is powered on and the USB is connected to the PC host, open the OrbbecViewer tool software, and the software will automatically connect the device. Some devices may take a long time to initialize, please wait patiently for the device to connect successfully.

The parameter configuration items of each stream are hidden by default. You can click the four buttons in the device management area to view and set specific configuration items.<br />![17](./Orbbec%20Viewer%20Manual%20img/17.png)<br />After the connection is successful, the control panel area automatically displays the specific information of the device.<br />![18](./Orbbec%20Viewer%20Manual%20img/18.png)
#### Depth** stream module**
Including: mirroring, software filtering (range mode: off, medium, long distance), depth effective range acquisition (MinDepthValue/MaxDepthValue), exposure and other functions.<br />![19](./Orbbec%20Viewer%20Manual%20img/19.png)
#### Color stream module
Including: Mirror, Flip, Align, Exposure, White Balance, Gain, Brightness, Sharpness, Saturation, Contrast, Hue and other functions.<br />![20](./Orbbec%20Viewer%20Manual%20img/20.png)
#### Infrared stream module
Including mirroring and exposure functions.<br />![21](./Orbbec%20Viewer%20Manual%20img/21.png)
#### **Device data management**
Users can select corresponding images and captured frames for recording, and the captured frames are saved in the "OrbbecViewer/output" directory by default.<br />![22](./Orbbec%20Viewer%20Manual%20img/22.png)
## 3.6 D2C function (support hardware D2C and software D2C)
The "D2C" function in advanced mode can control the synchronization function of depth stream and color stream:<br />1. Click the button![23](./Orbbec%20Viewer%20Manual%20img/23.png)You can open the depth stream and color stream synchronously by default;<br />2. Available via toggle button![24](./Orbbec%20Viewer%20Manual%20img/24.png)Enables or disables the frame synchronization function;<br />3. Click the button![25](./Orbbec%20Viewer%20Manual%20img/25.png)You can preview the rendering display effect of depth overlay to color;<br />4. Click the button![26](./Orbbec%20Viewer%20Manual%20img/26.png)Depth and color frame sync capture can be turned on or off (Femto specific);<br />5. The recording file is saved in the "OrbbecViewer/output/RecordFile" directory by default.<br />![27](./Orbbec%20Viewer%20Manual%20img/27.png)
## 3.7 **Point Cloud**
click![28](./Orbbec%20Viewer%20Manual%20img/28.png)The point cloud is enabled.<br />The zoom of the point cloud can be controlled by the mouse wheel, and the movement of the point cloud can be controlled by the movement of the mouse.<br />You can choose to export the depth point cloud (xyz) or RGBD point cloud (xyzrgb) and save it as a ply format file, which is saved in the "OrbbecViewer/output/PointCloud" directory by default.<br />![29](./Orbbec%20Viewer%20Manual%20img/29.png)
## 3.8 **Firmware update**
When the device is connected, click![30](./Orbbec%20Viewer%20Manual%20img/30.png)Access the firmware update page.

1. Femto device firmware includes system firmware and onboard MCU firmware. You can select the upgrade firmware type in the "Type" drop-down box.
1. After selecting the firmware type, enter the existing updated firmware image file (“.img” suffix) in the “Firmware” input box, and click the “Upgrade” button to start the update.
1. The device firmware update adopts the A/B dual partition scheme. If the update fails unexpectedly, it will not affect the operation of the original firmware, just reconnect the device to update. The device update time is relatively long, and the device will automatically restart once, please pay attention to the prompt information and wait patiently.

Astra+ interface:<br />![31](./Orbbec%20Viewer%20Manual%20img/31.png)<br />Femto interface:<br />![32](./Orbbec%20Viewer%20Manual%20img/32.png)
## 3.9 **Log Information**
By default, the log information area is displayed in a single-folded state. By clicking the button on the far right.![33](./Orbbec%20Viewer%20Manual%20img/33.png)Expand to view full log information.  By clicking the button![34](./Orbbec%20Viewer%20Manual%20img/34.png)will empty the log. By clicking the button![35](./Orbbec%20Viewer%20Manual%20img/35.png)restores the collapsed single bar display state.<br />![36](./Orbbec%20Viewer%20Manual%20img/36.png)
# Features
Demonstrates the use of the main API
## 4.1 Get serial number
![37](./Orbbec%20Viewer%20Manual%20img/37.png)
### C++ code
```
DeviceInfo deviceInfo = device->getDeviceInfo();
std::string serialNumber = deviceInfo.serialNumber();
```
### Android code
```
DeviceInfo deviceInfo = device.getInfo();
String serialNumber = deviceInfo.getSerialNumber();
```
## 4.2 Get device name
![38](./Orbbec%20Viewer%20Manual%20img/38.png)
### C++ code
```
std::shared_ptr< DeviceInfo > deviceInfo = device->getDeviceInfo();
std::string deviceName = deviceInfo->name();
```
### Android code
```
DeviceInfo deviceInfo = device.getInfo();
String name = deviceInfo.getName();
```
## 4.3 Get camera parameters
![39](./Orbbec%20Viewer%20Manual%20img/39.png)
### C++ code
```
//Get the internal parameters of the depth camera
OBCameraIntrinsic colorCameraIntrinsic = device->getCameraIntrinsic(OB_SENSOR_DEPTH);
//Get the internal parameters of the color camera
OBCameraIntrinsic depthCameraIntrinsic = device->getCameraIntrinsic(OB_SENSOR_COLOR);
//Get the distortion parameters of the depth camera 
OBCameraDistortion depthCameraDistortion = device->getCameraDistortion(OB_SENSOR_DEPTH);
//Get the distortion parameters of the color camera 
OBCameraDistortion colorCameraDistortion = device->getCameraDistortion(OB_SENSOR_color);
//Get the rotation matrix
OBD2CTransform d2cTransform = device->getD2CTransform();
```
### Android code
```
CameraParams object = new CameraParams();
boolean isSupport = device.isPropertySupported(DeviceProperty.CAMERA_PARA);
if (!isSupport) {
    return;
}
device.getPropertyValueDataType(DeviceProperty.CAMERA_PARA, object);
//Get the internal parameters of the depth camera
float[] depthParams = object.getDepthInternalParams();
//Get the internal parameters of the color camera
float[] colorParams = object.getColorInternalParams();
//Get the distortion parameters of the depth camera 
float[] depthCoeffs = object.getDepthCoeffs();
//Get the distortion parameters of the color camera 
float[] colorCoeffs = object.getColorCoeffs();
```
## 4.4 Get and set infrared camera exposure values
![40](./Orbbec%20Viewer%20Manual%20img/40.png)
### C++ code
```
std::shared_ptr<ob::Sensor> irSensor = device->getSensorList()->getSensor(OB_SENSOR_IR);
if(!irSensor->isPropertySupported(OB_SENSOR_PROPERTY_EXPOSURE_INT))
    return;
//Get the exposure value of the infrared camera
int32_t exposure = irSensor->getIntProperty(OB_SENSOR_PROPERTY_EXPOSURE_INT);
//Set the exposure value of the infrared camera
irSensor->setIntProperty(OB_SENSOR_PROPERTY_EXPOSURE_INT, exposure / 2);
```
### Android code
```
Sensor irSensor = device.getSensor(SensorType.IR);
boolean isSupport = irSensor.isPropertySupported(SensorProperty.EXPOSURE_INT);
if (!isSupport) {
    return;
}
//Get the exposure value of the infrared camera
int exposure = irSensor.getPropertyValueI(SensorProperty.EXPOSURE_INT);
//Set the exposure value of the infrared camera
irSensor.setPropertyValueI(SensorProperty.EXPOSURE_INT, exposure / 2);
```
## 4.5 Color camera automatic exposure
![41](./Orbbec%20Viewer%20Manual%20img/41.png)
### C++ code
```
bool isOpen;
std::shared_ptr<ob::Sensor> colorSensor = device->getSensorList()->getSensor(OB_SENSOR_COLOR);
if(!colorSensor->isPropertySupported(OB_SENSOR_PROPERTY_ENABLE_AUTO_EXPOSURE_BOOL))
    return;

colorSensor->setBoolProperty(OB_SENSOR_PROPERTY_ENABLE_AUTO_EXPOSURE_BOOL, isOpen);
```
### Android code
```
boolean isOpen;
Sensor colorSensor = device.getSensor(SensorType.COLOR);
boolean isSupport = colorSensor.isPropertySupported(SensorProperty.ENABLE_AUTO_EXPOSURE_BOOL);
if (!isSupport) {
    return;
}
colorSensor.setPropertyValueB(SensorProperty.ENABLE_AUTO_EXPOSURE_BOOL, isOpen);
```
## 4.6 Get and set color camera exposure values
![42](./Orbbec%20Viewer%20Manual%20img/42.png)
### C++ code
```
std::shared_ptr<ob::Sensor> colorSensor = device->getSensorList()->getSensor(OB_SENSOR_COLOR);
if(!colorSensor->isPropertySupported(OB_SENSOR_PROPERTY_EXPOSURE_INT))
    return;
//Get the exposure value of the color camera
int32_t exposure = colorSensor->getIntProperty(OB_SENSOR_PROPERTY_EXPOSURE_INT);
//Set the exposure value of the color camera
colorSensor->setIntProperty(OB_SENSOR_PROPERTY_EXPOSURE_INT, exposure / 2);
```
### Android code
```
Sensor colorSensor = device.getSensor(SensorType.COLOR);
boolean isSupport = colorSensor.isPropertySupported(SensorProperty.EXPOSURE_INT);
if (!isSupport) {
    return;
}
//Get the exposure value of the color camera
int exposure = colorSensor.getPropertyValueI(SensorProperty.EXPOSURE_INT);
//Set the exposure value of the color camera
colorSensor.setPropertyValueI(SensorProperty.EXPOSURE_INT, exposure / 2);
```
## 4.7 Color camera automatic white balance
### ![43](./Orbbec%20Viewer%20Manual%20img/43.png)
### C++ code
```
bool isOpen;
std::shared_ptr<ob::Sensor> colorSensor = device->getSensorList()->getSensor(OB_SENSOR_COLOR);
if(!colorSensor->isPropertySupported(OB_SENSOR_PROPERTY_ENABLE_AUTO_WHITE_BALANCE_BOOL))
    return;

colorSensor->setBoolProperty(OB_SENSOR_PROPERTY_ENABLE_AUTO_WHITE_BALANCE_BOOL, isOpen);
```
### Android code
```
boolean isOpen;
Sensor colorSensor = device.getSensor(SensorType.COLOR);
boolean isSupport = colorSensor.isPropertySupported(SensorProperty.ENABLE_AUTO_WHITE_BALANCE_BOOL);
if (!isSupport) {
    return;
}
colorSensor.setPropertyValueB(SensorProperty.ENABLE_AUTO_WHITE_BALANCE_BOOL, isOpen);
```
## 4.8 Get and set color camera gain
![44](./Orbbec%20Viewer%20Manual%20img/44.png)
### C++ code
```
std::shared_ptr<ob::Sensor> colorSensor = device->getSensorList()->getSensor(OB_SENSOR_COLOR);
if(!colorSensor->isPropertySupported(OB_SENSOR_PROPERTY_GAIN_INT))
    return;
//Get the color camera gain value
int32_t gain = colorSensor->getIntProperty(OB_SENSOR_PROPERTY_GAIN_INT);
//Set the color camera gain value
colorSensor->setIntProperty(OB_SENSOR_PROPERTY_GAIN_INT, gain / 2);
```
### Android code
```
boolean isExposure;
Sensor colorSensor = device.getSensor(SensorType.COLOR);
boolean isSupport = colorSensor.isPropertySupported(SensorProperty.GAIN_INT);
if (!isSupport) {
    return;
}
//Get the color camera gain value
int gain = colorSensor.getPropertyValueI(SensorProperty.GAIN_INT);
//Set the color camera gain value
colorSensor.setPropertyValueI(SensorProperty.GAIN_INT, gain / 2);
```
## 4.9 Color camera data stream mirroring
![45](./Orbbec%20Viewer%20Manual%20img/45.png)
### C++ code
```
std::shared_ptr<ob::Sensor> colorSensor = device->getSensorList()->getSensor(OB_SENSOR_COLOR);
if(!colorSensor->isPropertySupported(OB_SENSOR_PROPERTY_ROLL_INT))
    return;
//1 - Set Mirror； 0 - No Mirror
colorSensor->setBoolProperty(OB_SENSOR_PROPERTY_ROLL_INT, 1);
```
### Android code
```
Sensor colorSensor = device.getSensor(SensorType.COLOR);
boolean isSupport = colorSensor.isPropertySupported(SensorProperty.ROLL_INT);
if (!isSupport) {
    return;
}
//1 - Set Mirror； 0 - No Mirror
colorSensor.setPropertyValueI(SensorProperty.ROLL_INT, 1);
```
## 4.10 Depth camera data stream mirroring
### ![46](./Orbbec%20Viewer%20Manual%20img/46.png)
### C++ code
```
bool isMirror;
std::shared_ptr<ob::Sensor> depthSensor = device->getSensorList()->getSensor(OB_SENSOR_COLOR);
if(!depthSensor->isPropertySupported(OB_DEVICE_PROPERTY_DEPTH_MIRROR_BOOL))
    return;

depthSensor->setBoolProperty(OB_DEVICE_PROPERTY_DEPTH_MIRROR_BOOL, isMirror);
```
### Android code
```
boolean isMirror;
boolean isSupport = device.isPropertySupported(DeviceProperty.DEPTH_MIRROR_BOOL);
if (!isSupport) {
    return;
}
device.setPropertyValueB(DeviceProperty.DEPTH_MIRROR_BOOL, isMirror);
```
## 4.11 Infrared camera data stream mirroring
![47](./Orbbec%20Viewer%20Manual%20img/47.png)
### C++ code
```
bool isMirror;
std::shared_ptr<ob::Sensor> irSensor = device->getSensorList()->getSensor(OB_SENSOR_IR);
if(!irSensor->isPropertySupported(OB_SDK_PROPERTY_IR_MIRROR_BOOL))
    return;

irSensor->setBoolProperty(OB_SDK_PROPERTY_IR_MIRROR_BOOL, isMirror);
```
### Android code
```
boolean isMirror;
boolean isSupport = device.isPropertySupported(DeviceProperty.IR_MIRROR_BOOL);
if (!isSupport) {
    return;
}
device.setPropertyValueB(DeviceProperty.IR_MIRROR_BOOL, isMirror);
```



