# Windows10
The Orbbec SDK is compatible with the standard UVC protocol, and the supported hardware devices comply with the standard UVC specifications, so no additional drivers are required.
## 1. Verify device status

- Connect the device to the host
- Navigate to "Control Panel" -> "Device Manager"
- Browse to find the Orbbec device, as shown in the figure below, the device is successfully connected. (Different devices may have different channel numbers.

![1](./Orbbec%20SDK%20Environment%20Configuration%20img/1.png)
## 2. Configure OpenCV (Examples dependency)
Data rendering relies on the third-party library OpenCV. Here we take OpenCV 4.3.0 as an example to demonstrate the installation configuration<br />1) Execute the OpenCV installation file, select the directory where opencv is to be installed, and click extract to execute the installation;<br />

![2](./Orbbec%20SDK%20Environment%20Configuration%20img/2.png)

2)Add the path of OpenCV in the environment variables of the system, enter OpenCV_DIR for the variable name, pay attention to the capitalization of the letters, and the variable value is the path to the build folder of the OpenCV installation directory；<br />

![3](./Orbbec%20SDK%20Environment%20Configuration%20img/3.png)

![4](./Orbbec%20SDK%20Environment%20Configuration%20img/4.png)

## 3. Generate your first application
Software dependency: VisualStudio2019, cmake 3.10 and above<br />1) Download/obtain our SDK software package. The storage location is assumed to be the root directory of Disk D: "D:/V1.0.0". The directory structure is as follows:<br />

![5](./Orbbec%20SDK%20Environment%20Configuration%20img/5.png)

2)Open Cmake and set the "Examples" folder as the code path, and the "build" folder under "Examples" as the path to generate the binary file, as shown in the figure below. If there is no build under Examples, you need to create a new folder.<br />

![6](./Orbbec%20SDK%20Environment%20Configuration%20img/6.png)

3)Click "Configure" and select the corresponding Visual Studio version and platform version, then click "Finish", as shown below:<br />

![7](./Orbbec%20SDK%20Environment%20Configuration%20img/7.png)

4)Click "Generate", as shown below:<br />

![8](./Orbbec%20SDK%20Environment%20Configuration%20img/8.png)

5)The Sample project can be opened in the following two ways<br />Method 1: With cmake, click the "Open Project" button to open the Visual Studio project<br />

![9](./Orbbec%20SDK%20Environment%20Configuration%20img/9.png)

Method 2: In the folder, directly start the Visual Studio project in Examples/build, as shown in the figure below.

![10](./Orbbec%20SDK%20Environment%20Configuration%20img/10.png)

6)Open the Examples project interface as shown below:<br />

![11](./Orbbec%20SDK%20Environment%20Configuration%20img/11.png)

(7) Select the project you want to run, right click and "set as startup project", select release and 64-bit version at the run option；<br /><br />

![12](./Orbbec%20SDK%20Environment%20Configuration%20img/12.png)

![13](./Orbbec%20SDK%20Environment%20Configuration%20img/13.png)

<br />8) Connect the device to the host<br />9) Copy the dll files from the bin directory of the Examples folder to the build folder where the generated executables located at , and then run the project.

![14](./Orbbec%20SDK%20Environment%20Configuration%20img/14.png)

![22](./Orbbec%20SDK%20Environment%20Configuration%20img/22.png)

# Android
## 1. Verify device status
1)Prepare a set of P1 development board (A311D), the environment is Android 9<br />2) Prepare ADB tools and configure environment variables locally on the PC<br />3) Prepare USB cable for the connection of the A311D and the PC. At the same time, A311D system can be accessed for debugging through ADB tool (identify A311D as drive-free), and prepare USB 3.0 Type-C cable to connect A311D and Astra+ camera.<br />

![15](./Orbbec%20SDK%20Environment%20Configuration%20img/15.png)

![16](./Orbbec%20SDK%20Environment%20Configuration%20img/16.png)

<br />4) Take ASTRA+ as an example to determine whether A311D identifies as a camera normally<br />Judging by the lsusb command:<br />VID: 2bc5 PID: 0532 (color camera)<br />VID: 2bc5 PID: 0636 (depth camera) <br />

![17](./Orbbec%20SDK%20Environment%20Configuration%20img/17.png)

<br />If the system can recognize the camera normally, the device PID will be enumerated on the USB bus (as shown in the picture above), indicating that the A311D has recognized the camera and can do subsequent operations.

## 2. AAR file import
We have encapsulated the corresponding file into an aar file (OrbbecSDK_v1.0.0_release.aar), you can directly import it into the project.<br />1) Put "obsensor_xxxx_release.aar" in the libs directory under module;<br />2) Add local dependencies in the dependencies of the module's build.gradle;<br />3) Click File --> Sync Project with Gradle Files of AS and wait for the update to complete;<br />![18](./Orbbec%20SDK%20Environment%20Configuration%20img/18.png)

<br />4) Configure AndroidManifest.xml permissions, the SDK needs to monitor the permissions of USB devices and cameras, apply statically in AndroidManifest.xml, and dynamically apply when Java/kotlin is running;<br />5) In our corresponding Java file, "import com.orbbec.obsensor.*", add the corresponding code, as shown below:<br />![19](./Orbbec%20SDK%20Environment%20Configuration%20img/19.png)<br />If the corresponding SDK version information is printed out normally, the SDK integration is successful.

## 3. Example project import
In addition to the aar file, we also provide an example project, you can directly open the corresponding Android project for viewing and development.
## 4. Obfuscated configuration
If the project has Proguard configuration, then it is necessary to keep the OrbbecSDK Jave class.

```
-keep class com.orbbec.obsensor.**
-dontwarn com.orbbec.obsensor.**
-keepclassmembers class com.orbbec.obsensor.** { *; }
-keepclasseswithmembernames,includedescriptorclasses class * {
    native <methods>;
}
```
# Linux
## 1. System environment configuration
1)Install libudev library: sudo apt install libudev-dev<br />2) Install libusb library: sudo apt install libusb-dev

## 2. USB access rights configuration
By default, direct access to USB devices in Linux systems requires root privileges, which can be resolved through the rules configuration file. After the files released by OrbbecSDK are decompressed, there will be a "99-obsensor-libusb.rules" configuration file and "install.sh" installation script in the root directory. Run the "install.sh" script through the sudo command to complete the rules Installation of configuration files. In addition, if the "install.sh" installation script does not have execution permission, it can be solved by changing the command: "sudo chmod +x ./install.sh". After the installation script is successfully executed, it will take effect when the device is connected again (the connected device needs to be re-plugged).<br />![20](./Orbbec%20SDK%20Environment%20Configuration%20img/20.png)

## 3. Verify device status
1)Environment preparation: ubuntu18.04 x64 desktop<br />2) Take Astra+ as an example, use USB 3.0 Type-C data cable to connect with PC.<br />3) Use the lsusb command to check if the PC system correctly recognizes the camera<br />

![21](./Orbbec%20SDK%20Environment%20Configuration%20img/21.png)

<br />4) Judge whether the camera is recognized normally by PID&VID<br />VID: 2bc5 PID: 0532 (color camera)<br />VID: 2bc5 PID: 0636 (depth camera)
