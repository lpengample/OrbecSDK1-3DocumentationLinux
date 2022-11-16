# Overview
Welcome to the Orbbec SDK (hereinafter referred to as "SDK") tutorial! The SDK not only provides a concise high-level API, but also a flexible and comprehensive low-level API to help you use and quickly understand Orbbec 3D cameras in detail. 

# Features
Orbbec SDK is a cross-platform (Windows, Android, Linux) software development kit that provides device parameter configuration, data stream reading and stream processing for 3D sensing cameras such as Orbbec structured light, binocular, and iToF.

**Core functions: **

- Depth camera access and related parameter settings 
- RGB camera access and related parameter settings (eg exposure and white balance) 
- Sensor access and related parameter settings (eg gyroscope and accelerometer) 
- Frame synchronization and alignment control 
- Point cloud data 
- Algorithmic capabilities such as filtering 
- Multi-OS and Wrapper support. 

**Highlights:**

SDK design goals: thin + flexible + high scalability. 

- "Thin": Provides the ability to obtain device data at the minimum level and high performance 
- "Flexible": Modular sensor function, flexible combination of different devices 
- "Highly Scalable": Supports increasingly diversified devices and systems, and plug-in algorithms for different scenarios 

What's included in the SDK:

| Content | Description                                                  |
| --- | --- |
| Code example | These simple examples demonstrate how to easily use the SDK to include code snippets that access the camera into your application. Includes color flow, depth flow, point cloud, alignment, recording, and more.  |
| tool | OrbbecViewer: A tool that demonstrates the main basic functions and parameter configuration of the 3D sensing camera using the SDK to help developers quickly understand and verify the capabilities of the SDK and the 3D sensing camera. With this application, you can quickly access your depth camera to view the depth stream, visualize point clouds, record and playback data streams, configure your camera settings.  |

# Hardware products supported by the SDK
| SDK version | Product | Firmware version |
| --- | --- | --- |
| v1.0.2 | Astra+ | 1.0.19 |
| v1.1.6 | Astra+ | 1.0.19/1.0.20 |
|  | Femto | 1.5.1 |
| v1.3.1 | Astra+ | 1.0.21/1.0.20/1.0.19 |
|  | Femto | 1.6.9/1.6.7 |

# SDK system requirements
| Operating system | Requirement                                                  | Description |
| --- | --- | --- |
| Windows | - Windows 10 April 2018 (version 1803, operating system build 17134) release (x64) or higher<br />- 4 GB RAM<br />- USB2.0 and above ports<br /> | The generation of the VS project depends on the installation of the VS version and the cmake version, and supports VS2015/vs2017/vs2019 |
| Android | - Android 6/7/8/9/10 |  |
| Linux | - Linux Ubuntu 16.04/18.04/20.04 (x64)<br />- support ARM/ARM64<br />-4 GB RAM<br />- USB2.0 and above ports<br /> |Support GCC 7.5|

# Supported Languages & Wrapper


![1](./Orbbec%20SDK%20Overview%20Document%20img/1.png)





# SDK Architecture
![2](./Orbbec%20SDK%20Overview%20Document%20img/2.png)

**Low-level API/High-level API/Wrappers**

The API layer encapsulates core business components and provides APIs for different wrappers.

**Basic business layer**

The realization of the core business logic framework realizes the functions of Low Level and High Level.

**Platform abstraction layer**

Cross-platform components shield the implementation of different operating systems and provide a unified access method. 

**Platform implementation layer**

The driver implementation of each platform.
