All examples can be found in the project's OrbbecSdkExamples directory.

| Name | Language | Description |
| --- | --- | --- |
| HelloOrbbec | Java | Demonstrate connect to device to get SDK version and device information |
| DepthViewer | Java | Open the specified configuration depth stream through the Pipeline and render it |
| ColorViewer | Java | Turn on the specified configuration color stream through the Pipeline and render it |
| InfraredViewer | Java | Open the specified configuration infrared stream through Pipeline and render it |
| SyncAlignViewer | Java | Demonstrate operations on sensor data stream alignment |
| HotPlugin | Java | Demonstrate the settings of the device plug and unplug callback, and get the stream processed after plugging and unplugging |
| IMUReader | Java | Get IMU data and output display |
| MultiDevice | Java | Demonstrate operation on multiple devices |
| PointCloud | Java | Demonstrate the generation of depth point cloud or RGBD point cloud and save it as ply format file |
| SaveToDisk | Java | Get color storage in png format, depth storage in raw format and format conversion through filter |
| SensorControl | Java | Demonstrate manipulation of device and sensor control commands |
| Record and playback example | Java | Connect the device to open the stream, record the current video stream to a file, and load the video file for playback. |

# Android
## HelloOrbbec
Function description: Used to demonstrate SDK initialization, get SDK version, get device model, get device serial number, get firmware version number, and SDK release resources.

First, you need to create a Context to obtain a list of device information and create a device.
```
private OBContext mOBContext;
```
Initialize the SDK and monitor device changes
```
// 1.Initialize the SDK, and listen for device changes
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
    @Override
    public void onDeviceAttach(DeviceList deviceList) {
        try {
            StringBuilder builder = new StringBuilder();
            // 2.Check SDK version
            builder.append("SDK Version = " + OBContext.getVersionName() + "\n");
            
            // 3.Get the number of devices
            int deviceCount = deviceList.getDeviceCount();
            
            for (int i = 0; i < deviceCount; ++i) {
                // 4.Create device from device index
                Device device = deviceList.getDevice(i);
                
                // 5.Get device information
                DeviceInfo info = device.getInfo();
                
                // 6.Get device version information
                builder.append("Name: " + info.getName() + "\n");
                builder.append("Vid: " + LocalUtils.formatHex04(info.getVid()) + "\n");
                builder.append("Pid: " + LocalUtils.formatHex04(info.getPid()) + "\n");
                builder.append("Uid: " + info.getUid() + "\n");
                builder.append("SN: " + info.getSerialNumber() + "\n");
                String firmwareVersion = info.getFirmwareVersion();
                builder.append("FirmwareVersion: " + firmwareVersion + "\n");
                String hardwareVersion = info.getHardwareVersion();
                builder.append("HardwareVersion: " + hardwareVersion + "\n");
                
                // 7.Iterate over the sensors of the current device
                for (Sensor sensor : device.querySensors()) {
                    // 8.Query sensor type
                    builder.append("Sensor : type = " + sensor.getType() + "\n");
                }
                runOnUiThread(() -> txtInfo.setText(builder.toString()));
                
                // 9.Release device information
                info.close();
                
                // 10.Free up device resources
                device.close();
            }
            // 11.Release device list resources
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onDeviceDetach(DeviceList deviceList) {
        try {
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```
View SDK version number
```java
// Get SDK version name
String version = OBContext.getVersionName();
// Get SDK kernel version name (version of Orbbec SDK so)
String coreVersion = OBContext.getCoreVersionName();
```
The device index can be obtained through the callback, and the device is created according to the device index. Here we take the creation of the first device as an example.
```
Device device = deviceList.getDevice(0);
```
Next, we can get the information from this device, the following information can be obtained through DeviceInfo.
```
// 5.Get device information
DeviceInfo info = device.getInfo();

// 6.Get device version information
builder.append("Name: " + info.getName() + "\n");
builder.append("Vid: " + LocalUtils.formatHex04(info.getVid()) + "\n");
builder.append("Pid: " + LocalUtils.formatHex04(info.getPid()) + "\n");
builder.append("Uid: " + info.getUid() + "\n");
builder.append("SN: " + info.getSerialNumber() + "\n");
String firmwareVersion = info.getFirmwareVersion();
builder.append("FirmwareVersion: " + firmwareVersion + "\n");
String hardwareVersion = info.getHardwareVersion();
builder.append("HardwareVersion: " + hardwareVersion + "\n");

// 7.Iterate over the sensors of the current device
for (Sensor sensor : device.querySensors()) {
    // 8.Query sensor type
    builder.append("Sensor : type = " + sensor.getType() + "\n");
}
runOnUiThread(() -> txtInfo.setText(builder.toString()));

// 9.release device information
info.close();
```
Iterate sensor type
```
for (Sensor sensor : device.querySensors()) {
    // Query sensor type
    Log.d(TAG,"Sensor : type = " + sensor.getType() + "\n");
}
```
Resource release
```
// Turn off the device
device.close();

// Release SDK
if (null != mOBContext) {
    mOBContext.close();
    mOBContext = null;
}
```
## ColorViewer
Function description: This example mainly demonstrates the initialization of SDK, device creation, Pipeline initialization and configuration, as well as enabling the specified configuration for color stream and rendering through the Pipeline.

First, you need to create a Context to obtain a list of device information and create a device.
```
private OBContext mOBContext;
```
Initialize the SDK and monitor device changes
```
// 1.Initialize the SDK, and listen for device changes
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
    @Override
    public void onDeviceAttach(DeviceList deviceList) {
        try {
            if (null == mPipeline) {
                // 2.Create Device and initialize Pipeline through Device
                mDevice = deviceList.getDevice(0);
                mPipeline = new Pipeline(mDevice);
                
                // 3.Create Pipeline Configuration
                Config config = new Config();
                
                // 4.Get the color stream configuration and configure it to Config, where the configuration is 640x480 YUYV format or 640x480 I420 format
                StreamProfileList colorProfileList = mPipeline.getStreamProfileList(SensorType.COLOR);
                StreamProfile streamProfile = null;
                if (null != colorProfileList) {
                    streamProfile = colorProfileList.getVideoStreamProfile(640, 480, Format.YUYV, 0);
                    
                    if (null == streamProfile) {
                        streamProfile = colorProfileList.getVideoStreamProfile(640, 480, Format.I420, 0);
                    }
                    
                    colorProfileList.close();
                }
                
                if (streamProfile == null) {
                    return;
                }
                
                // 5. If the configuration format is I420, create a format conversion filter to convert the I420 format to RGB888 format for rendering
                if (streamProfile.getFormat() == Format.I420) {
                    mFormatConvertFilter = new FormatConvertFilter();
                    mFormatConvertFilter.setFormatType(FormatConvertType.FORMAT_I420_TO_RGB888);
                }
                
                // 6.Enable Color Configuration
                config.enableStream(streamProfile);
                
                // 7.Set mirror
                if (mDevice.isPropertySupported(DeviceProperty.OB_PROP_COLOR_MIRROR_BOOL, PermissionType.OB_PERMISSION_WRITE)) {
                    mDevice.setPropertyValueB(DeviceProperty.OB_PROP_COLOR_MIRROR_BOOL, true);
                }
                
                // 8.Open flow
                mPipeline.start(config);
                
                // 9.Release config
                config.close();
                
                // 10.Free streamProfile
                streamProfile.close();
                
                // 11.Create a thread to get Pipeline data
                start();
            }
            
            // 12.Release device list resources
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onDeviceDetach(DeviceList deviceList) {
        try {
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```
Create a device and create a Pipeline through the device
```
// 2. Create Device and initialize Pipeline through Device
mDevice = deviceList.getDevice(0);
mPipeline = new Pipeline(mDevice);
```
Create a pipeline configuration
```
// 3.Create Pipeline Configuration
Config config = new Config();
```
Get the color stream profile and configure it to Config, where the configuration is 640x480 YUYV format or 640x480 I420 format.
```
//4.Get the color stream configuration and configure it to Config, where the configuration is 640x480 YUYV format or 640x480 I420 format
StreamProfileList colorProfileList = mPipeline.getStreamProfileList(SensorType.COLOR);
StreamProfile streamProfile = null;
if (null != colorProfileList) {
    streamProfile = colorProfileList.getVideoStreamProfile(640, 480, Format.YUYV, 0);
    
    if (null == streamProfile) {
        streamProfile = colorProfileList.getVideoStreamProfile(640, 480, Format.I420, 0);
    }
    
    colorProfileList.close();
}

if (streamProfile == null) {
    return;
}

// 5. If the configuration format is I420, create a format conversion filter to convert the I420 format to RGB888 format for rendering
if (streamProfile.getFormat() == Format.I420) {
    mFormatConvertFilter = new FormatConvertFilter();
    mFormatConvertFilter.setFormatType(FormatConvertType.FORMAT_I420_TO_RGB888);
}

// 6. Enable Color Configuration
config.enableStream(streamProfile);
```
Setting mirroring through the Device attribute interface.
```
// 7. Set mirror
if (mDevice.isPropertySupported(DeviceProperty.OB_PROP_COLOR_MIRROR_BOOL, PermissionType.OB_PERMISSION_WRITE)) {
    mDevice.setPropertyValueB(DeviceProperty.OB_PROP_COLOR_MIRROR_BOOL, true);
}
```
Use Config configuration to open flow through Pipeline and release local resources
```
// 8.Open flow
mPipeline.start(config);

// 9.Release config
config.close();

// 10.Release streamProfile
streamProfile.close();
```
Create a thread to get Pipeline data.
```java
private void start() {
    mIsStreamRunning = true;
    if (null == mStreamThread) {
        mStreamThread = new Thread(mStreamRunnable);
        mStreamThread.start();
    }
}
```
Obtain the frameset in blocking mode, if it cannot be obtained after 100ms, it will time out.
```
FrameSet frameSet = mPipeline.waitForFrames(100);
```
Data processing and rendering
```
// Get color stream data
ColorFrame colorFrame = frameSet.getColorFrame();

// Data processing and rendering
if (null != colorFrame) {
    if (colorFrame.getFormat() == Format.I420) {
        // Convert the color data frame in I420 format to the data frame in RGB888 format through filter
        Frame frame = mFormatConvertFilter.process(colorFrame);
        if (null != frame) {
            // Get data and render
            byte[] frameData = new byte[colorFrame.getWidth() * colorFrame.getHeight() * 3];
            frame.getData(frameData);
            mColorView.update(frameData, colorFrame.getWidth(), colorFrame.getHeight(), StreamType.COLOR, Format.RGB888);
            frame.close();
        }
    } else {
        // Get data and render
        byte[] frameData = new byte[colorFrame.getDataSize()];
        colorFrame.getData(frameData);
        mColorView.update(frameData, colorFrame.getWidth(), colorFrame.getHeight(), StreamType.COLOR, colorFrame.getFormat());
    }
    
    // Free color data frame
    colorFrame.close();
}
// Free dataset
frameSet.close();
```
Stop obtaining pipeline data
```
private void stop() {
    mIsStreamRunning = false;
    if (null != mStreamThread) {
        try {
            mStreamThread.join(300);
        } catch (InterruptedException e) {
        }
        mStreamThread = null;
    }
}
```
Resource release
```
// Release the frameset
frameSet.close();

// Release filter resources
if (null != mFormatConvertFilter) {
    try {
        mFormatConvertFilter.close();
    } catch (Exception e) {
        e.printStackTrace();
    }
}

// Stop and close pipeline
if (null != mPipeline) {
    mPipeline.stop();
    mPipeline.close();
}

// Release Device
if (mDevice != null){
    mDevice.close();
}

// Release SDK resources
if (null != mOBContext) {
    mOBContext.close();
}

// Release rendering window resources
if (mColorView != null){
    mColorView.release();
}
```
## DepthViewer
Function description: This example mainly demonstrates the initialization of SDK, device creation, Pipeline initialization and configuration, and opening the specified configuration depth stream and rendering through the Pipeline.

First, you need to create a Context to obtain a list of device information and create a device.
```
private OBContext mOBContext;
```
Initialize the SDK and monitor device changes
```
// 1.Initialize the SDK, and listen for device changes
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
    @Override
    public void onDeviceAttach(DeviceList deviceList) {
        try {
            if (null == mPipeline) {
                // 2.Create Device, and initialize Pipeline through Device
                mDevice = deviceList.getDevice(0);
                mPipeline = new Pipeline(mDevice);
                
                // 3.Create Pipeline Configuration
                Config config = new Config();
                
                // 4.Get the depth stream configuration and configure it to Config, the configuration here is 640x480 Y16 format
                StreamProfileList depthProfileList = mPipeline.getStreamProfileList(SensorType.DEPTH);
                StreamProfile streamProfile = null;
                
                if (null != depthProfileList) {
                    streamProfile = depthProfileList.getVideoStreamProfile(640, 480, Format.Y16, 30);
                    depthProfileList.close();
                }
                
                if (streamProfile == null) {
                    return;
                }
                
                // 5.Enable depth stream configuration
                config.enableStream(streamProfile);
                
                // 6.Set mirror
                if (mDevice.isPropertySupported(DeviceProperty.OB_PROP_DEPTH_MIRROR_BOOL, PermissionType.OB_PERMISSION_WRITE)) {
                    mDevice.setPropertyValueB(DeviceProperty.OB_PROP_DEPTH_MIRROR_BOOL, true);
                }
                
                // 7.Open flow
                mPipeline.start(config);
                
                // 8.Release config
                config.close();
                
                // 9.Release streamProfile
                streamProfile.close();
                
                // 10.Create a thread to get Pipeline data
                start();
            }
            // 11.Release device list resources
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onDeviceDetach(DeviceList deviceList) {
        try {
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```
Create a device and create a Pipeline through the device
```
// 2.Create Device, and initialize Pipeline through Device
mDevice = deviceList.getDevice(0);
mPipeline = new Pipeline(mDevice);
```
Create a pipeline configuration
```
// 3.Create a pipeline configuration
Config config = new Config();
```
Get the depth stream profile and configure it to Config, where the configuration is in 640x480 Y16 format.
```
// 4.Get the depth stream configuration and configure it to Config, the configuration here is 640x480 Y16 format
StreamProfileList depthProfileList = mPipeline.getStreamProfileList(SensorType.DEPTH);
StreamProfile streamProfile = null;

if (null != depthProfileList) {
    streamProfile = depthProfileList.getVideoStreamProfile(640, 480, Format.Y16, 30);
    depthProfileList.close();
}

if (streamProfile == null) {
    return;
}

// 5.Enable depth stream configuration
config.enableStream(streamProfile);
```
Setting mirroring through the Device attribute interface
```
// 6.Set mirror
if (mDevice.isPropertySupported(DeviceProperty.OB_PROP_DEPTH_MIRROR_BOOL, PermissionType.OB_PERMISSION_WRITE)) {
    mDevice.setPropertyValueB(DeviceProperty.OB_PROP_DEPTH_MIRROR_BOOL, true);
}
```
Use Config configuration to open flow through Pipeline and release local resources
```java
mPipeline.start(config);

// 8.Release config
config.close();

// 9.Release streamProfile
streamProfile.close();
```
Create and get the pipeline data thread
```
private void start() {
    mIsStreamRunning = true;
    if (null == mStreamThread) {
        mStreamThread = new Thread(mStreamRunnable);
        mStreamThread.start();
    }
}
```
Obtain the frameset in blocking mode, if it cannot be obtained after 100ms, it will time out
```
// if it cannot be obtained after 100ms, it will time out
FrameSet frameSet = mPipeline.waitForFrames(100);
```
Data processing and rendering
```
// Get depth stream data
DepthFrame frame = frameSet.getDepthFrame();

if (frame != null) {
    // Get depth data and render
    byte[] frameData = new byte[frame.getDataSize()];
    frame.getData(frameData);
    mDepthView.update(frameData, frame.getWidth(), frame.getHeight(), StreamType.DEPTH, frame.getFormat());
    
    // Release depth data frame
    frame.close();
}

// Release dataset
frameSet.close();
```
Stop obtaining pipeline data
```
private void stop() {
    mIsStreamRunning = false;
    if (null != mStreamThread) {
        try {
            mStreamThread.join(300);
        } catch (InterruptedException e) {
        }
        mStreamThread = null;
    }
}
```
Resource release
```
// Release the frameset
frameSet.close();

// Stop and close pipeline
if (null != mPipeline) {
    mPipeline.stop();
    mPipeline.close();
}

// Release Device
if (mDevice != null){
    mDevice.close();
}

// Release SDK resources
if (null != mOBContext) {
    mOBContext.close();
}

// Release rendering window resources
if (mDepthView != null) {
    mDepthView.release();
}
```
## InfraredViewer
Function description: This example mainly demonstrates SDK initialization, device creation, Pipeline initialization and configuration, and opening and rendering the specified configured infrared stream through the Pipeline.

First, you need to create a Context to obtain a list of device information and create a device.
```
private OBContext mOBContext;
```
Initialize the SDK and monitor device changes
```
// 1.Initialize the SDK, and listen for device changes
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
    @Override
    public void onDeviceAttach(DeviceList deviceList) {
        try {
            if (null == mPipeline) {
                // 2.Create Device, and initialize Pipeline through Device
                mDevice = deviceList.getDevice(0);
                mPipeline = new Pipeline(mDevice);
                
                // 3.Create Pipeline Configuration
                Config config = new Config();
                
                // 4.Get the depth stream configuration and configure it to Config, the configuration here is 640x480 Y16 format
                StreamProfileList depthProfileList = mPipeline.getStreamProfileList(SensorType.DEPTH);
                StreamProfile streamProfile = null;
                
                if (null != depthProfileList) {
                    streamProfile = depthProfileList.getVideoStreamProfile(640, 480, Format.Y16, 30);
                    depthProfileList.close();
                }
                
                if (streamProfile == null) {
                    return;
                }
                
                // 5.Enable depth stream configuration
                config.enableStream(streamProfile);
                
                // 6.Set mirror
                if (mDevice.isPropertySupported(DeviceProperty.OB_PROP_DEPTH_MIRROR_BOOL, PermissionType.OB_PERMISSION_WRITE)) {
                    mDevice.setPropertyValueB(DeviceProperty.OB_PROP_DEPTH_MIRROR_BOOL, true);
                }
                
                // 7.Open flow
                mPipeline.start(config);
                
                // 8.Release config
                config.close();
                
                // 9.Release streamProfile
                streamProfile.close();
                
                // 10.Create a thread to get Pipeline data
                start();
            }
            // 11.Release device list resources
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onDeviceDetach(DeviceList deviceList) {
        try {
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```
Create a device and create a Pipeline through the device
```
// 2. Create Device, and initialize Pipeline through Device
mDevice = deviceList.getDevice(0);
mPipeline = new Pipeline(mDevice);
```
Create a pipeline configuration
```
//3. Create a Pipeline configuration
Config config = new Config();
```
Get the infrared stream profile and configure it to Config, where the configuration is in 640x480 Y16 format
```
// 4.Get the flow configuration and configure it to Config
StreamProfileList irProfileList = mPipeline.getStreamProfileList(SensorType.IR);
StreamProfile streamProfile = null;
if (null != irProfileList) {
    streamProfile = irProfileList.getVideoStreamProfile(640, 480, Format.Y16, 30);
    irProfileList.close();
}

if (streamProfile == null) {
    return;
}

// 5.Enable IR Streaming Configuration
config.enableStream(streamProfile);
```
Setting mirroring through the Device attribute interface
```java
// 6.Set mirror
if (mDevice.isPropertySupported(DeviceProperty.OB_PROP_IR_MIRROR_BOOL, PermissionType.OB_PERMISSION_READ)) {
    mDevice.setPropertyValueB(DeviceProperty.OB_PROP_IR_MIRROR_BOOL, true);
}
```
Use Config to configure open streaming through Pipeline
```
// 7.Open flow
mPipeline.start(config);

// 8.Release config
config.close();

// 9.Release streamProfile
streamProfile.close();
```
Create a thread for obtaining Pipeline data
```
private void start() {
    mIsStreamRunning = true;
    if (null == mStreamThread) {
        mStreamThread = new Thread(mStreamRunnable);
        mStreamThread.start();
    }
}
```
Obtain the frameset in blocking mode, if it cannot be obtained after 100ms, it will time out
```
// If it cannot be obtained after 100ms, it will time out
FrameSet frameSet = mPipeline.waitForFrames(100);
```
Data processing and rendering
```
// Get infrared stream data
IRFrame frame = frameSet.getIrFrame();

if (frame != null) {
    // Get infrared data and render
    byte[] frameData = new byte[frame.getDataSize()];
    frame.getData(frameData);
    mIrView.update(frameData, frame.getWidth(), frame.getHeight(), StreamType.IR, frame.getFormat());
    
    // Release infrared data frame
    frame.close();
}

// Release dataset
frameSet.close();
```
Stop obtaining pipeline data
```
private void stop() {
    mIsStreamRunning = false;
    if (null != mStreamThread) {
        try {
            mStreamThread.join(300);
        } catch (InterruptedException e) {
        }
        mStreamThread = null;
    }
}
```
Resource release
```
// Release the frameset
frameSet.close();

// Stop and close pipeline
if (null != mPipeline) {
    mPipeline.stop();
    mPipeline.close();
}

// Release Device
if (mDevice != null){
    mDevice.close();
}

// Release SDK resources
if (null != mOBContext) {
    mOBContext.close();
}

// Release rendering window resources
if (null != mIrView) {
    mIrView.release();
}
```
## SyncAlignViewer
Function description: This example mainly demonstrates the operation of sensor data stream alignment.

First, you need to create a Context to obtain a list of device information and create a device.
```
private OBContext mOBContext;
```
Initialize the SDK and monitor device changes
```
// 1.Initialize the SDK, and listen for device changes
mOBContext = new OBContext(this, new DeviceChangedCallback() {
    @Override
    public void onDeviceAttach(DeviceList deviceList) {
        try {
            if (null == mPipeline) {
                // 2.Get Device and create Pipeline through Device
                mDevice = deviceList.getDevice(0);
                mPipeline = new Pipeline(mDevice);
                
                // 3.Create a format conversion filter to convert data in I420 format to RGB888 format
                mFormatConvertFilter = new FormatConvertFilter();
                mFormatConvertFilter.setFormatType(FormatConvertType.FORMAT_I420_TO_RGB888);
                
                // 4.Create Pipeline Configuration
                mConfig = new Config();
                
                // 5.D2C is turned off by default
                mConfig.setAlignMode(AlignMode.ALIGN_D2C_DISABLE);
                
                // 6.Get the color stream configuration in the specified format
                StreamProfileList colorProfileList = mPipeline.getStreamProfileList(SensorType.COLOR);
                StreamProfile colorStreamProfile = null;
                if (null != colorProfileList) {
                    colorStreamProfile = colorProfileList.getVideoStreamProfile(640, 480, Format.YUYV, 0);
                    if (null == colorStreamProfile) {
                        colorStreamProfile = colorProfileList.getVideoStreamProfile(640, 480, Format.I420, 0);
                    }
                    colorProfileList.close();
                }
                
                if (colorStreamProfile == null) {
                    return;
                }
                
                // 7.Enable color stream configuration
                mConfig.enableStream(colorStreamProfile);
                
                // 8.Get depth stream configuration
                StreamProfileList depthProfileList = mPipeline.getStreamProfileList(SensorType.DEPTH);
                StreamProfile depthStreamProfile = null;
                if (null != depthProfileList) {
                    depthStreamProfile = depthProfileList.getVideoStreamProfile(640, 480, Format.Y16, 30);
                    depthProfileList.close();
                }
                
                if (depthStreamProfile == null) {
                    return;
                }
                
                // 9.Enable depth stream configuration
                mConfig.enableStream(depthStreamProfile);
                
                // 10.Open flow through config
                mPipeline.start(mConfig);
                
                // 11.Release colorStreamProfile
                colorStreamProfile.close();
                
                // 12.Release depthStreamProfile
                depthStreamProfile.close();
                
                // 13.Create a thread to get Pipeline data
                start();
            }
            
            // 14.Release device list resources
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onDeviceDetach(DeviceList deviceList) {
        try {
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```
Create a device and a Pipeline through the device
```
// 2.Get Device and create Pipeline through Device
mDevice = deviceList.getDevice(0);
mPipeline = new Pipeline(mDevice);
```
Create Pipeline Configuration
```java
mConfig = new Config();
```
Create a format conversion filter to convert data in I420 format to RGB888 format
```java
// 3.Create a format conversion filter to convert data in I420 format to RGB888 format
mFormatConvertFilter = new FormatConvertFilter();
mFormatConvertFilter.setFormatType(FormatConvertType.FORMAT_I420_TO_RGB888);
```
Create Pipeline Configuration
```java
// 4.Create Pipeline Configuration
mConfig = new Config();
```
D2C is turned off by default
```java
// 5.D2C is turned off by default
mConfig.setAlignMode(AlignMode.ALIGN_D2C_DISABLE);
```
Get the color stream configuration and configure it to Config, where the configuration is 640x480 YUYV format or 640x480 I420 format
```java
// 6.Get the color stream configuration in the specified format
StreamProfileList colorProfileList = mPipeline.getStreamProfileList(SensorType.COLOR);
StreamProfile colorStreamProfile = null;
if (null != colorProfileList) {
    colorStreamProfile = colorProfileList.getVideoStreamProfile(640, 480, Format.YUYV, 0);
    if (null == colorStreamProfile) {
        colorStreamProfile = colorProfileList.getVideoStreamProfile(640, 480, Format.I420, 0);
    }
    colorProfileList.close();
}

if (colorStreamProfile == null) {
    return;
}

// 7.Enable color stream configuration
mConfig.enableStream(colorStreamProfile);
```
Get the depth stream profile and configure it to Config, where the configuration is in 640x480 Y16 format
```
// 8.Get depth stream configuration
StreamProfileList depthProfileList = mPipeline.getStreamProfileList(SensorType.DEPTH);
StreamProfile depthStreamProfile = null;
if (null != depthProfileList) {
    depthStreamProfile = depthProfileList.getVideoStreamProfile(640, 480, Format.Y16, 30);
    depthProfileList.close();
}

if (depthStreamProfile == null) {
    return;
}

// 9.Enable depth stream configuration
mConfig.enableStream(depthStreamProfile);
```
Use Config to configure the stream through Pipeline
```
// 10.Open flow through config
mPipeline.start(mConfig);

// 11.Rlease colorStreamProfile
colorStreamProfile.close();

// 12.Rlease depthStreamProfile
depthStreamProfile.close();
```
Create a thread for obtaining Pipeline data
```
private void start() {
    mIsStreamRunning = true;
    if (null == mStreamThread) {
        mStreamThread = new Thread(mStreamRunnable);
        mStreamThread.start();
    }
}
```
Obtain the frameset in blocking mode, if it cannot be obtained after 100ms, it will time out
```
FrameSet frameSet = mPipeline.waitForFrames(100);
```
Get color frame and depth frame
```
// Get color frame
ColorFrame colorFrame = frameSet.getColorFrame();

// Get depth frame
DepthFrame depthFrame = frameSet.getDepthFrame();
```
Data processing and rendering
```
if (null != depthFrame && null != colorFrame) {
    // Transcode depth frame to RGB888 format
    ByteBuffer depthSrcBytebuffer = ByteBuffer.allocateDirect(depthFrame.getDataSize());
    depthSrcBytebuffer.rewind();
    byte[] depthSrcBytes = new byte[depthFrame.getDataSize()];
    
    // Get depth data
    depthFrame.getData(depthSrcBytes);
    depthSrcBytebuffer.put(depthSrcBytes);
    ByteBuffer depthDstBytebuffer = ByteBuffer.allocateDirect(depthFrame.getWidth() * depthFrame.getHeight() * 3);
    
    // Transcode depth frame to RGB888 format
    YuvUtil.depth2RGB888(depthSrcBytebuffer, depthDstBytebuffer, depthFrame.getWidth(), depthFrame.getHeight(), 1);
    depthData = new byte[depthFrame.getWidth() * depthFrame.getHeight() * 3];
    depthDstBytebuffer.get(depthData, 0, depthData.length);
    
    // Transcode color frame to RGB888 format
    if (colorFrame.getFormat() == Format.I420) {
        // Transcode I420 format data to RGB888 format
        Frame frame = mFormatConvertFilter.processFrame(colorFrame);
        if (null != frame) {
            colorData = new byte[colorFrame.getWidth() * colorFrame.getHeight() * 3];
            frame.getData(colorData);
            // Release frame
            frame.close();
        }
    } else {
        // Transcode YUYV format data to RGB888 format
        ByteBuffer colorSrcBytebuffer = ByteBuffer.allocateDirect(colorFrame.getDataSize());
        colorSrcBytebuffer.rewind();
        byte[] colorSrcBytes = new byte[colorFrame.getDataSize()];
        colorFrame.getData(colorSrcBytes);
        colorSrcBytebuffer.put(colorSrcBytes);
        ByteBuffer colorDstBytebuffer = ByteBuffer.allocateDirect(colorFrame.getWidth() * colorFrame.getHeight() * 3);
        YuvUtil.yuyv2Rgb888(colorSrcBytebuffer, colorDstBytebuffer, colorFrame.getWidth() * colorFrame.getHeight() * 3);
        colorData = new byte[colorFrame.getWidth() * colorFrame.getHeight() * 3];
        colorDstBytebuffer.get(colorData, 0, colorData.length);
    }
}
if (depthData != null && colorData != null) {
		// Superimpose depth and color data, mAlpha is the transparency value of the depth map
    byte[] depthColorData = depthToColor(depthData, colorData, 640, 480, mAlpha);
    
    // Render the superimposed depth and color data
    mColorView.update(depthColorData, 640, 480, StreamType.COLOR, Format.RGB888);
}

// Release depth frame
if (null != depthFrame) {
    depthFrame.close();
}

// Release color frame
if (null != colorFrame) {
    colorFrame.close();
}

// Release frameset
frameSet.close();
```
Set frame synchronization
```
// Set frame synchronization
private void setSync(boolean isChecked) {
    try {
        if (isChecked) {
            mPipeline.enableFrameSync();
        } else {
            mPipeline.disableFrameSync();
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Set up D2C
```
// Set up D2C
private void setAlignToColor(boolean isChecked, boolean isHardware) {
    try {
        if (mPipeline == null || mDevice == null) {
            return;
        }
        try {
            if (isChecked) {
                mConfig.setAlignMode((isHardware ? AlignMode.ALIGN_D2C_HW_ENABLE : AlignMode.ALIGN_D2C_SW_ENABLE));
            } else {
                mConfig.setAlignMode(AlignMode.ALIGN_D2C_DISABLE);
            }
            mPipeline.switchConfig(mConfig);
        } catch (Exception e) {
            Log.w(TAG, "setAlignToColor: " + e.getMessage());
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Stop obtaining pipeline data
```
private void stop() {
    mIsStreamRunning = false;
    if (null != mStreamThread) {
        try {
            mStreamThread.join(300);
        } catch (InterruptedException e) {
        }
    }
}
```
Resource release
```
// Stop and close pipeline
if (null != mPipeline) {
    mPipeline.stop();
    mPipeline.close();
}
// Release FormatConvertFilter
if (null != mFormatConvertFilter) {
    try {
        mFormatConvertFilter.close();
    } catch (Exception e) {
    }
    mFormatConvertFilter = null;
}
// Release Config
if (null != mConfig) {
    mConfig.close();
}
// Release Device
if (null != mDevice) {
    mDevice.close();
}
// Release SDK resources
if (null != mOBContext) {
    mOBContext.close();
}
// Release window rending resources
if (null != mColorView) {
    mColorView.release();
}
```
## HotPlugin
Function description: This example mainly demonstrates the settings of the device plug-in callback, and processes the stream after plug-in.

First, you need to create a Context to obtain a list of device information and create a device.
```
private OBContext mOBContext;
```
FrameCallback that defines Color
```java
private FrameCallback mColorFrameCallback = frame -> {
	printFrameInfo(frame.as(FrameType.COLOR), mColorFps);

	// Release frame resources
	frame.close();
};

```
Define the FrameCallback of Depth
```java
private FrameCallback mColorFrameCallback = frame -> {
    printFrameInfo(frame.as(FrameType.COLOR), mColorFps);

    // Release frame resources
    frame.close();
};
```
FrameCallback that defines IR
```java
private FrameCallback mIrFrameCallback = frame -> {
    printFrameInfo(frame.as(FrameType.IR), mIrFps);

    // Release frame resources
    frame.close();
};
```
Print data frame information
```java
private void printFrameInfo(VideoFrame frame, int fps) {
	try {
		String frameInfo = "FrameType:" + frame.getStreamType()
				+ ", index:" + frame.getFrameIndex()
				+ ", width:" + frame.getWidth()
				+ ", height:" + frame.getHeight()
				+ ", format:" + frame.getFormat()
				+ ", fps:" + fps
				+ ", timeStampUs:" + frame.getTimeStampUs();
		if (frame.getStreamType() == FrameType.DEPTH) {
			frameInfo += ", middlePixelValue:" + getMiddlePixelValue(frame);
		}
		Log.i(TAG, frameInfo);
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```
Initialize the SDK and monitor device changes
```
// 1.Initialize the SDK, and listen for device changes
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
    @Override
    public void onDeviceAttach(DeviceList deviceList) {
        try {
            if (deviceList == null || deviceList.getDeviceCount() <= 0) {
                setText(mNameTv, "No device connected !");
            }
            
            // 2.Create a device and get the device name
            mDevice = deviceList.getDevice(0);
            DeviceInfo devInfo = mDevice.getInfo();
            String deviceName = devInfo.getName();
            setText(mNameTv, deviceName);
            devInfo.close();
            
            // 3.Get a depth sensor
            mDepthSensor = mDevice.getSensor(SensorType.DEPTH);
            
            // 4.Open the depth stream, and pass in null to the profile, which means to use the parameters configured in the configuration file to open the stream,
            // If the configuration does not exist in the device, or there is no configuration file, it means that the first configuration in the Profile list is used
            if (null != mDepthSensor) {
                mDepthSensor.start(null, mDepthFrameCallback);
            }
            
            // 5.Get color sensor
            mColorSensor = mDevice.getSensor(SensorType.COLOR);
            
            // 6.Open the color stream, and pass in null in the profile, indicating that the stream is opened using the parameters configured in the configuration file.
            // If the configuration does not exist in the device, or there is no configuration file, it means that the first configuration in the Profile list is used
            if (null != mColorSensor) {
                mColorSensor.start(null, mColorFrameCallback);
            }
            
            // 7.Get IR sensor
            mIrSensor = mDevice.getSensor(SensorType.IR);
            
            // 8.Open the infrared stream, and pass in null into the profile, which means that the stream is opened using the parameters configured in the configuration file.
            // If the configuration does not exist in the device, or there is no configuration file, it means that the first configuration in the Profile list is used
            if (null != mIrSensor) {
                mIrSensor.start(null, mIrFrameCallback);
            }
            
            // 9.Update OpenFlow configuration information
            setText(mProfileInfoTv, formatProfileInfo());
            
            // 10.Release deviceList resources
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onDeviceDetach(DeviceList deviceList) {
        try {
            setText(mNameTv, "No device connected !");
            setText(mProfileInfoTv, "");
            
            mDepthFps = 0;
            mColorFps = 0;
            mIrFps = 0;
            
            // Stop depth flow
            if (null != mDepthSensor) {
                mDepthSensor.stop();
            }
            
            // Stop color flow
            if (null != mColorSensor) {
                mColorSensor.stop();
            }
            
            // Stop infrared stream
            if (null != mIrSensor) {
                mIrSensor.stop();
            }
            
            // Release Device
            if (null != mDevice) {
                mDevice.close();
                mDevice = null;
            }
            
            // Release deviceList
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```
Processing in the device attach callback (onDeviceAttach)
```
try {
    setText(mNameTv, "No device connected !");
    setText(mProfileInfoTv, "");

    mDepthFps = 0;
    mColorFps = 0;
    mIrFps = 0;

    // Stop depth flow
    if (null != mDepthSensor) {
        mDepthSensor.stop();
    }

    // Stop color flow
    if (null != mColorSensor) {
        mColorSensor.stop();
    }

    // Stop infrared stream
    if (null != mIrSensor) {
        mIrSensor.stop();
    }

    // Release Device
    if (null != mDevice) {
        mDevice.close();
        mDevice = null;
    }

    // Release deviceList
    deviceList.close();
} catch (Exception e) {
    e.printStackTrace();
}
```
Resource release
```
try {
    // Stop depth flow
    if (null != mDepthSensor) {
        mDepthSensor.stop();
    }
    
    // Stop color flow
    if (null != mColorSensor) {
        mColorSensor.stop();
    }
    
    // Stop infrared stream
    if (null != mIrSensor) {
        mIrSensor.stop();
    }
    
    //  Release Device
    if (null != mDevice) {
        mDevice.close();
    }
    
    //Release SDK
    if (null != mOBContext) {
        mOBContext.close();
    }
} catch (Exception e) {
    e.printStackTrace();
}
}
```
## IMU
Function description: This example mainly demonstrates the use of SDK to obtain IMU data and display the output.

First, you need to create a Context to obtain a list of device information and create a device.
```
private OBContext mOBContext;
```
Define IMU related sensors
```java
// Accelerometer sensor
private AccelFrame mAccelFrame;
// Gyro sensor
private GyroFrame mGyroFrame;
```
Initialize the SDK and monitor device changes
```
// 1.Initialize the SDK, and listen for device changes
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
    @Override
    public void onDeviceAttach(DeviceList deviceList) {
        try {
            if (deviceList == null || deviceList.getDeviceCount() == 0) {
                showToast("Please access the device");
            } else {
                // 2.Create Device
                mDevice = deviceList.getDevice(0);
                
                // 3.Get acceleration Sensor through Device
                mSensorAccel = mDevice.getSensor(SensorType.ACCEL);
                
                // 4.Get the gyroscope Sensor through Device
                mSensorGyro = mDevice.getSensor(SensorType.GYRO);
                
                if (mSensorAccel == null || mSensorGyro == null) {
                    showToast("This device does not support IMU");
                    deviceList.close();
                    return;
                } else {
                    runOnUiThread(() -> {
                        mSurfaceViewImu.setVisibility(View.VISIBLE);
                    });
                }
                
                if (mSensorAccel != null && mSensorGyro != null) {
                    // 5.Get accelerometer configuration
                    StreamProfileList accelProfileList = mSensorAccel.getStreamProfileList();
                    if (null != accelProfileList) {
                        mAccelStreamProfile = accelProfileList.getStreamProfile(0).as(StreamType.ACCEL);
                        accelProfileList.close();
                    }
                    
                    // 6.Get gyroscope configuration
                    StreamProfileList gyroProfileList = mSensorGyro.getStreamProfileList();
                    if (null != gyroProfileList) {
                        mGyroStreamProfile = gyroProfileList.getStreamProfile(0).as(StreamType.GYRO);
                        gyroProfileList.close();
                    }
                }
            }
            // 8.Release device list resources
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onDeviceDetach(DeviceList deviceList) {
        try {
            showToast("Please access the device");
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```
Create a device, and get accelerometer sensor and gyroscope sensor through the device
```
// 2.Create a device
mDevice = deviceList.getDevice(0);

// 3.get accelerometer sensor through the device
mSensorAccel = mDevice.getSensor(SensorType.ACCEL);

// 4.get gyroscope sensor through the device
mSensorGyro = mDevice.getSensor(SensorType.GYRO);

if (mSensorAccel == null || mSensorGyro == null) {
    showToast("This device does not support IMU");
    deviceList.close();
    return;
} else {
    runOnUiThread(() -> {
        mSurfaceViewImu.setVisibility(View.VISIBLE);
    });
}
```
Get the accelerometer and gyroscope stream configuration
```
if (mSensorAccel != null && mSensorGyro != null) {
    // 5.Get accelerometer configuration
    StreamProfileList accelProfileList = mSensorAccel.getStreamProfileList();
    if (null != accelProfileList) {
        mAccelStreamProfile = accelProfileList.getStreamProfile(0).as(StreamType.ACCEL);
        accelProfileList.close();
    }

    // 6.Get gyroscope configuration
    StreamProfileList gyroProfileList = mSensorGyro.getStreamProfileList();
    if (null != gyroProfileList) {
        mGyroStreamProfile = gyroProfileList.getStreamProfile(0).as(StreamType.GYRO);
        gyroProfileList.close();
    }
}
```
Open stream by specifying configuration
```
private void startIMU() {
    // 7.1.Initialize the IMU data, refresh thread
    mIsRefreshIMUDataRunning = true;
    mRefreshIMUDataThread = new Thread(mRefreshIMUDataRunnable);
    mRefreshIMUDataThread.setName("RefreshIMUDataThread");
    
    // 7.2.Start the IMU data, refresh thread
    mRefreshIMUDataThread.start();
    
    // 7.3.Start gyroscope sampling
    startGyroStream();
    
    // 7.4.Start accelerometer sampling
    startAccelStream();
}
```
Start accelerometer sampling
```
private void startAccelStream() {
    try {
        // Start accelerometer sampling
        if (null != mAccelStreamProfile) {
            mSensorAccel.start(mAccelStreamProfile, new FrameCallback() {
                @Override
                public void onFrame(Frame frame) {
                    AccelFrame accelFrame = frame.as(FrameType.ACCEL);
                    
                    if (null == mAccelFrame) {
                        mAccelFrame = accelFrame;
                        return;
                    }
                    
                    frame.close();
                }
            });
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Start gyroscope sampling
```
private void startGyroStream() {
    try {
        // Start gyroscope sampling
        if (null != mGyroStreamProfile) {
            mSensorGyro.start(mGyroStreamProfile, new FrameCallback() {
                @Override
                public void onFrame(Frame frame) {
                    GyroFrame gyroFrame = frame.as(FrameType.GYRO);
                    
                    if (null == mGyroFrame) {
                        mGyroFrame = gyroFrame;
                        return;
                    }
                    
                    frame.close();
                }
            });
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Create AccelFrame and GyroFrame data drawing methods, and draw in real-time on the data refresh thread
```
private void drawImuInfo() {
    try {
        Canvas canvas = mSurfaceViewImu.getHolder().lockCanvas();
        if (null != canvas) {
            mPaintIMU.setTextSize(25.0f);
            Paint.FontMetrics fm = mPaintIMU.getFontMetrics();
            float offsetY = fm.descent - fm.ascent;
            float x = canvas.getWidth() / 8;
            float y = canvas.getHeight() / 8;
            canvas.drawColor(Color.WHITE);
            if (mAccelFrame != null) {
                canvas.drawText("AccelTimestamp:" + mAccelFrame.getTimeStamp(), x, y, mPaintIMU);
                canvas.drawText("AccelTemperature:" + mAccelFrame.getTemperature() + "dC", x, (y += offsetY), mPaintIMU);
                
                canvas.drawText("Accel.x:" + mAccelFrame.getAccelData()[0] + "g", x, (y += 2 * offsetY), mPaintIMU);
                canvas.drawText("Accel.y:" + mAccelFrame.getAccelData()[1] + "g", x, (y += offsetY), mPaintIMU);
                canvas.drawText("Accel.z:" + mAccelFrame.getAccelData()[2] + "g", x, (y += offsetY), mPaintIMU);
                
                // Release AccelFrame resources
                mAccelFrame.close();
                mAccelFrame = null;
            }
            
            if (mGyroFrame != null) {
                canvas.drawText("GyroTimestamp:" + mGyroFrame.getTimeStamp(), x, (y += 2 * offsetY), mPaintIMU);
                canvas.drawText("GyroTemperature:" + mGyroFrame.getTemperature() + "dC", x, (y += offsetY), mPaintIMU);
                
                canvas.drawText("Gyro.x:" + mGyroFrame.getGyroData()[0] + "dps", x, (y += 2 * offsetY), mPaintIMU);
                canvas.drawText("Gyro.y:" + mGyroFrame.getGyroData()[1] + "dps", x, (y += offsetY), mPaintIMU);
                canvas.drawText("Gyro.z:" + mGyroFrame.getGyroData()[2] + "dps", x, (y += offsetY), mPaintIMU);
                
                // Free GyroFrame resources
                mGyroFrame.close();
                mGyroFrame = null;
            }
            mSurfaceViewImu.getHolder().unlockCanvasAndPost(canvas);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Stop data sampling
```
private void stopIMU() {
    try {
        // Stop accelerometer sampling
        if (null != mSensorAccel) {
            mSensorAccel.stop();
        }
        
        // Stop gyro sampling
        if (null != mSensorGyro) {
            mSensorGyro.stop();
        }
        
        // Stop IMU data refresh thread and release
        mIsRefreshIMUDataRunning = false;
        if (null != mRefreshIMUDataThread) {
            try {
                mRefreshIMUDataThread.join(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            mRefreshIMUDataThread = null;
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Resource release
```
try {
	// Release the accelerometer configuration
	if (null != mAccelStreamProfile) {
		mAccelStreamProfile.close();
		mAccelStreamProfile = null;
	}

	// Release gyro configuration
	if (null != mGyroStreamProfile) {
		mGyroStreamProfile.close();
		mGyroStreamProfile = null;
	}

	// Release Device
	if (null != mDevice) {
		mDevice.close();
		mDevice = null;
	}

	// Release SDK
	if (null != mOBContext) {
		mOBContext.close();
		mOBContext = null;
	}
} catch (Exception e) {
	e.printStackTrace();
}
```
## MultiDevice
Function description: This example mainly demonstrates the operation of multiple devices.

First, you need to create a Context to obtain a list of device information and create a device.
```
private OBContext mOBContext;
```
Initialize the SDK and monitor device changes
```
// Initialize SDK
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
    @Override
    public void onDeviceAttach(DeviceList deviceList) {
        try {
            int count = deviceList.getDeviceCount();
            for (int i = 0; i < count; i++) {
                // Create device
                Device device = deviceList.getDevice(i);
                // Get DeviceInfo
                DeviceInfo devInfo = device.getInfo();
                // Get device name
                String name = devInfo.getName();
                // Get device uid
                String uid = devInfo.getUid();
                // Get device usb interface type
                String connectionType = devInfo.getConnectionType();
                // Release DeviceInfo resources
                devInfo.close();
                runOnUiThread(() -> {
                    mDeviceControllerAdapter.addItem(new DeviceBean(name, uid, connectionType, device));
                });
                
            }
            
            // Release device list resources
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onDeviceDetach(DeviceList deviceList) {
        try {
            for (DeviceBean deviceBean : mDeviceBeanList) {
                //Judging offline devices by uid
                if (deviceBean.getDeviceUid().equals(deviceList.getUid(0))) {
                    // Release offline device resources
                    deviceBean.getDevice().close();
                    runOnUiThread(() -> {
                        mDeviceControllerAdapter.deleteItem(deviceBean);
                    });
                }
            }
            
            // Release device list resources
            deviceList.close();
        } catch (Exception e) {
            Log.w(TAG, "onDeviceDetach: " + e.getMessage());
        }
    }
});
```
Create a device list in the device connection callback method
```
int count = deviceList.getDeviceCount();
for (int i = 0; i < count; i++) {
    // Create device
    Device device = deviceList.getDevice(i);
    // Get DeviceInfo
    DeviceInfo devInfo = device.getInfo();
    // Get device name
    String name = devInfo.getName();
    // Get device uid
    String uid = devInfo.getUid();
    // Get device usb interface type
    String connectionType = devInfo.getConnectionType();
    // Release DeviceInfo resources
    devInfo.close();
    runOnUiThread(() -> {
        mDeviceControllerAdapter.addItem(new DeviceBean(name, uid, connectionType, device));
    });
}
```
Select the corresponding device to open the stream
```
private void startStream(Sensor sensor, GLView glView) {
	try {
		// Get a list of flow configurations for a sensor
		StreamProfileList profileList = sensor.getStreamProfileList();
		if (null == profileList) {
			Log.w(TAG, "start stream failed, profileList is null !");
			return;
		}
		switch (sensor.getType()) {
			case DEPTH:
				GLView depthGLView = glView;
				// Get the open flow configuration through StreamProfileList
				StreamProfile depthProfile = profileList.getVideoStreamProfile(640, 480, Format.Y16, 0);
				if (null != depthProfile) {
					// Open current by specifying configuration
					sensor.start(depthProfile, frame -> {
						DepthFrame depthFrame = frame.as(FrameType.DEPTH);
						byte[] bytes = new byte[depthFrame.getDataSize()];
						depthFrame.getData(bytes);
						// Render data
						depthGLView.update(bytes, depthFrame.getWidth(), depthFrame.getHeight(),
								StreamType.DEPTH, depthFrame.getFormat());
						// Release frame resources
						frame.close();
					});
					// Release profile resources
					depthProfile.close();
				} else {
					Log.w(TAG, "start depth stream failed, depthProfile is null!");
				}
				break;
			case COLOR:
				GLView colorGLView = glView;
				// Get the open flow configuration through StreamProfileList
				StreamProfile colorProfile = profileList.getVideoStreamProfile(640, 480, Format.RGB888, 0);

				if (null != colorProfile) {
					// Open current by specifying configuration
					sensor.start(colorProfile, frame -> {
						ColorFrame colorFrame = frame.as(FrameType.COLOR);
						byte[] bytes = new byte[colorFrame.getDataSize()];
						// Get frame data
						colorFrame.getData(bytes);
						// Render data
						colorGLView.update(bytes, colorFrame.getWidth(), colorFrame.getHeight(), StreamType.COLOR, Format.RGB888);
						// Release frame resources
						frame.close();
					});
					// Release profile resources
					colorProfile.close();
				} else {
					Log.w(TAG, "start color stream failed, colorProfile is null!");
				}
				break;
			case IR:
				GLView irGLView = glView;
				// Get the open flow configuration through StreamProfileList
				StreamProfile irProfile = profileList.getVideoStreamProfile(640, 480, Format.Y16, 0);
				if (null != irProfile) {
					// Open current by specifying configuration
					sensor.start(irProfile, frame -> {
						IRFrame irFrame = frame.as(FrameType.IR);
						byte[] bytes = new byte[irFrame.getDataSize()];
						// Get frame data
						irFrame.getData(bytes);
						// Render data
						irGLView.update(bytes, irFrame.getWidth(), irFrame.getHeight(),
								StreamType.IR, irFrame.getFormat());
						// Frame resource
						frame.close();
					});
					// Release profile resources
					irProfile.close();
				} else {
					Log.w(TAG, "start ir stream failed, irProfile is null!");
				}
				break;
		}

		// Release profileList resources
		profileList.close();
	} catch (Exception e) {
		Log.w(TAG, "startStream: " + e.getMessage());
	}
}
```
Shut down the specified sensor stream to the specified device
```java
private void stopStream(Sensor sensor) {
    try {
        sensor.stop();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Do the corresponding resource release in the device disconnect callback and refresh the device list
```java
try {
    for (DeviceBean deviceBean : mDeviceBeanList) {
        // Judging offline devices by uid
        if (deviceBean.getDeviceUid().equals(deviceList.getUid(0))) {
            // Release offline device resources
            deviceBean.getDevice().close();
            runOnUiThread(() -> {
                mDeviceControllerAdapter.deleteItem(deviceBean);
            });
        }
    }
} catch (Exception e) {
    Log.w(TAG, "onDeviceDetach: " + e.getMessage());
}
```
Resource release
```
try {
    // Release resources
    for (DeviceBean deviceBean : mDeviceBeanList) {
        try {
            // Release device resources
            deviceBean.getDevice().close();
        } catch (Exception e) {
            Log.w(TAG, "onDestroy: " + e.getMessage());
        }
    }
    mDeviceBeanList.clear();
    
    // Release SDK
    if (null != mOBContext) {
        mOBContext.close();
    }
} catch (Exception e) {
    e.printStackTrace();
  }
```
## PointCloud
Function description: This example mainly demonstrates the stream of connected devices, generating depth point cloud or RGBD point cloud and saving it as a ply format file.

First, you need to create a Context to obtain a list of device information and create a device.
```
private OBContext mOBContext;
```
Initialize the SDK and monitor device changes
```
// 1.Initialize SDK
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
	@Override
	public void onDeviceAttach(DeviceList deviceList) {
		try {
			if (null == mPipeline) {
				// 2.Get Device through deviceList
				mDevice = deviceList.getDevice(0);

				// 3.Create Pipeline through Device
				mPipeline = new Pipeline(mDevice);

				// 4.Create Config for configuring pipeline open flow
				Config config = new Config();

				// 5.Get the color configuration list, and get the 640x480 YUYV or I420 format configuration open flow
				StreamProfileList colorProfileList = mPipeline.getStreamProfileList(SensorType.COLOR);
				VideoStreamProfile colorProfileTarget = null;
				if (null != colorProfileList) {
					colorProfileTarget = colorProfileList.getVideoStreamProfile(640, 480, Format.YUYV, 0);
					if (null == colorProfileTarget) {
						colorProfileTarget = colorProfileList.getVideoStreamProfile(640, 480, Format.I420, 0);
					}
					colorProfileList.close();
				}

				// 6.Enable the configuration after obtaining the specified color configuration
				if (null != colorProfileTarget) {
					config.enableStream(colorProfileTarget);
					colorProfileTarget.close();
				}

				// 7.Get in-depth configuration list, and get 640x480 YUYV or Y16 format configuration open current
				VideoStreamProfile depthProfileTarget = null;
				StreamProfileList depthProfileList = mPipeline.getStreamProfileList(SensorType.DEPTH);
				if (null != depthProfileList) {
					depthProfileTarget = depthProfileList.getVideoStreamProfile(640, 480, Format.Y16, 0);
					depthProfileList.close();
				}

				// 8.Enable configuration after obtaining the specified depth configuration
				if (null != depthProfileTarget) {
					config.enableStream(depthProfileTarget);
					depthProfileTarget.close();
				}


				// 9.Enable hardware D2C
				config.setAlignMode(AlignMode.ALIGN_D2C_HW_ENABLE);

				// 10.Open flow using callback method
				mPipeline.start(config, mPointCloudFrameSetCallback);

				// 11.Start the point cloud asynchronous processing thread
				start();

				// 12.Create point cloud filter through Pipeline
				mPointCloudFilter = new PointCloudFilter();

				// 13.Set the format of the point cloud filter
				mPointCloudFilter.setPointFormat(mPointFormat);

				// 14.Get the camera internal parameters and set the parameters to the point cloud filter
				CameraParamList cameraParamList = mDevice.getCalibrationCameraParamList();
				CameraParam cameraParam = null;
				if (null != cameraParamList) {
					int count = cameraParamList.getCameraParamCount();
					for (int i = 0; i < count; i++) {
						cameraParam = cameraParamList.getCameraParam(i);
						int depthW = cameraParam.getDepthIntrinsic().getWidth();
						int depthH = cameraParam.getDepthIntrinsic().getHeight();
						int colorW = cameraParam.getColorIntrinsic().getWidth();
						int colorH = cameraParam.getColorIntrinsic().getHeight();
						// 14.1.It is necessary to obtain the internal reference that the calibrated resolution ratio is consistent with the open-current resolution ratio
						if ((depthW / depthH == 640 / 480) && (colorW / colorH == 640 / 480)) {
							break;
						}
					}
					cameraParamList.close();
				}
				mPointCloudFilter.setCameraParam(cameraParam);

				// 15.Release config resources
				config.close();
			}

			// 16.Release device list resources
			deviceList.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	@Override
	public void onDeviceDetach(DeviceList deviceList) {
		try {
			deviceList.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
});
```
Create a device and create a Pipeline through the device
```
// 2.Get Device through deviceList
mDevice = deviceList.getDevice(0);

// 3.Create Pipeline through Device
mPipeline = new Pipeline(mDevice);
```
Create a pipeline configuration
```
// 4.Create Config for configuring pipeline open flow
Config config = new Config();
```
Get the color stream profiles and configure it to Config, where the configuration is 640x480 YUYV format or 640x480 I420 format
```
// 5.Get the color configuration list, and get the 640x480 YUYV or I420 format configuration open flow
StreamProfileList colorProfileList = mPipeline.getStreamProfileList(SensorType.COLOR);
VideoStreamProfile colorProfileTarget = null;
if (null != colorProfileList) {
    colorProfileTarget = colorProfileList.getVideoStreamProfile(640, 480, Format.YUYV, 0);
    if (null == colorProfileTarget) {
        colorProfileTarget = colorProfileList.getVideoStreamProfile(640, 480, Format.I420, 0);
    }
    colorProfileList.close();
}

// 6.Enable the configuration after obtaining the specified color configuration
if (null != colorProfileTarget) {
    config.enableStream(colorProfileTarget);
    colorProfileTarget.close();
}
```
Get the depth stream profile, and configure it to Config, where the configuration is 640x480 YUYV format or Y16 format
```
// 7.Get depth configuration list, and get 640x480 YUYV or Y16 format configuration open current
VideoStreamProfile depthProfileTarget = null;
StreamProfileList depthProfileList = mPipeline.getStreamProfileList(SensorType.DEPTH);
if (null != depthProfileList) {
    depthProfileTarget = depthProfileList.getVideoStreamProfile(640, 480, Format.Y16, 0);
    depthProfileList.close();
}

// 8.Enable configuration after obtaining the specified depth configuration
if (null != depthProfileTarget) {
    config.enableStream(depthProfileTarget);
    depthProfileTarget.close();
}
```
Determine whether the device supports hardware D2C and turn it on
```
// 9.Enable hardware D2C
config.setAlignMode(AlignMode.ALIGN_D2C_HW_ENABLE);
```
Callback method for creating pipeline stream
```
//Pipeline open flow callback
private FrameSetCallback mPointCloudFrameSetCallback = frameSet -> {
    if (null != frameSet) {
        if (mIsPointCloudRunning) {
            if (null == mPointFrameSet) {
                mPointFrameSet = frameSet;
                return;
            }
        }
        frameSet.close();
    }
};
```
Use Config to configure the stream through the Pipeline callback method
```
// 10.Use Config to configure the stream through the Pipeline callback method
mPipeline.start(config, mPointCloudFrameSetCallback);
```
Start point cloud asynchronous processing thread
```
private void start() {
    mIsPointCloudRunning = true;
    if (null == mPointFilterThread) {
        mPointFilterThread = new Thread(mPointFilterRunnable);
        mPointFilterThread.start();
    }
}
```
Create a point cloud filter through Pipeline, and set the format of the point cloud filter (if you want to save the depth point cloud, you need to set the format to Format.POINT; if you want to save the RGBD point cloud, you need to set the format to Format.RGB_POINT)
```
// 12.Create point cloud filter through Pipeline
mPointCloudFilter = new PointCloudFilter();

// 13.Set the format of the point cloud filter
mPointCloudFilter.setPointFormat(mPointFormat);
```
Get the camera internal parameters and set the parameters to the point cloud filter
```
// 14.Get the camera internal parameters and set the parameters to the point cloud filter
CameraParamList cameraParamList = mDevice.getCalibrationCameraParamList();
CameraParam cameraParam = null;
if (null != cameraParamList) {
    int count = cameraParamList.getCameraParamCount();
    for (int i = 0; i < count; i++) {
        cameraParam = cameraParamList.getCameraParam(i);
        int depthW = cameraParam.getDepthIntrinsic().getWidth();
        int depthH = cameraParam.getDepthIntrinsic().getHeight();
        int colorW = cameraParam.getColorIntrinsic().getWidth();
        int colorH = cameraParam.getColorIntrinsic().getHeight();
        // 14.1.It is necessary to obtain the internal reference that the calibrated resolution ratio is consistent with the open-current resolution ratio
        if ((depthW / depthH == 640 / 480) && (colorW / colorH == 640 / 480)) {
            break;
        }
    }
    cameraParamList.close();
}
mPointCloudFilter.setCameraParam(cameraParam);
```
Point cloud filter thread data processing
```
while (mIsPointCloudRunning) {
    try {
        if (null != mPointFrameSet) {
            Frame frame = null;
            if (mPointFormat == Format.POINT) {
                // Set the save format to depth point cloud
                mPointCloudFilter.setPointFormat(Format.POINT);
            } else {
                // Set the save format to color point cloud
                mPointCloudFilter.setPointFormat(Format.RGB_POINT);
            }
            // Point cloud filter processing generates corresponding point cloud data
            frame = mPointCloudFilter.process(mPointFrameSet);
            
            if (null != frame) {
                // Get point cloud frame
                PointFrame pointFrame = frame.as(FrameType.POINTS);
                
                if (mIsSavePoints) {
                    if (mPointFormat == Format.POINT) {
                        // Obtain and save the depth point cloud data, the data size of the depth point cloud is w * h * 3
                        float[] depthPoints = new float[pointFrame.getDataSize() / Float.BYTES];
                        pointFrame.getPointCloudData(depthPoints);
                        String depthPointsPath = mSdcardDir.toString() + "/points.ply";
                        FileUtils.savePointCloud(depthPointsPath, depthPoints);
                        runOnUiThread(new Runnable() {
                            @Override
                            public void run() {
                                mInfoTv.append("Save Path:" + depthPointsPath + "\n");
                            }
                        });
                    } else {
                        // Get the color point cloud data and save it, the data size of the color point cloud is w * h * 6
                        float[] colorPoints = new float[pointFrame.getDataSize() / Float.BYTES];
                        pointFrame.getPointCloudData(colorPoints);
                        String colorPointsPath = mSdcardDir.toString() + "/rgb_points.ply";
                        FileUtils.saveRGBPointCloud(colorPointsPath, colorPoints);
                        runOnUiThread(new Runnable() {
                            @Override
                            public void run() {
                                mInfoTv.append("Save Path:" + colorPointsPath + "\n");
                            }
                        });
                    }
                    
                    mIsSavePoints = false;
                }
                
                // Release the newly generated frame
                frame.close();
            }
            
            // Release the original data frameSet
            mPointFrameSet.close();
            mPointFrameSet = null;
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Save depth point cloud data
```java
public static void savePointCloud(String fileName, float[] data) {
	try {
		File file = new File(fileName);
		FileOutputStream fos = new FileOutputStream(file);
		PrintWriter writer = new PrintWriter(fos);
		writer.write("ply\n");
		writer.write("format ascii 1.0\n");
		writer.write("element vertex 307200\n");
		writer.write("property float x\n");
		writer.write("property float y\n");
		writer.write("property float z\n");
		writer.write("end_header\n");
		writer.flush();

		for (int i = 0; i < data.length; i += 3) {
			writer.print(data[i]);
			writer.print(" ");
			writer.print(data[i + 1]);
			writer.print(" ");
			writer.print(data[i + 2]);
			writer.print("\n");
		}
		writer.close();
		fos.close();
	} catch (Exception e) {
		Log.e(TAG, "exception: " + e.getMessage());
	}
}
```
Save color point cloud data
```java
public static void saveRGBPointCloud(String fileName, float[] data) {
	try {
		File file = new File(fileName);
		FileOutputStream fos = new FileOutputStream(file);
		PrintWriter writer = new PrintWriter(fos);
		writer.write("ply\n");
		writer.write("format ascii 1.0\n");
		writer.write("element vertex 307200\n");
		writer.write("property float x\n");
		writer.write("property float y\n");
		writer.write("property float z\n");
		writer.write("property uchar red\n");
		writer.write("property uchar green\n");
		writer.write("property uchar blue\n");
		writer.write("end_header\n");
		writer.flush();

		for (int i = 0; i < data.length; i += 6) {
			writer.print(data[i]);
			writer.print(" ");
			writer.print(data[i + 1]);
			writer.print(" ");
			writer.print(data[i + 2]);
			writer.print(" ");
			writer.print((int) data[i + 3]);
			writer.print(" ");
			writer.print((int) data[i + 4]);
			writer.print(" ");
			writer.print((int) data[i + 5]);
			writer.print("\n");
		}
		writer.close();
		fos.close();
	} catch (Exception e) {
		Log.e(TAG, "exception: " + e.getMessage());
	}
}
```
Exit the point cloud filter processing thread
```
private void stop() {
    mIsPointCloudRunning = false;
    if (null != mPointFilterThread) {
        try {
            mPointFilterThread.join(300);
        } catch (InterruptedException e) {
        }
        mPointFilterThread = null;
    }
}
```
Resource release
```
// Stop and close pipeline
if (null != mPipeline) {
    mPipeline.stop();
    mPipeline.close();
    mPipeline = null;
}
// Release point cloud filter
if (null != mPointCloudFilter) {
    try {
        mPointCloudFilter.close();
    } catch (Exception e) {
    }
    mPointCloudFilter = null;
}
// Release Device
if (mDevice != null) {
    mDevice.close();
    mDevice = null;
}
// Release SDK resources
if (null != mOBContext) {
        mOBContext.close();
        mOBContext = null;
    }
} catch (Exception e) {
    e.printStackTrace();
}
```
## SaveToDisk
Function description: This example demonstrate the stream of connected devices, obtain color and depth maps and save them in png format (depth saves in raw format) and format conversion through filters.

First, you need to create a Context to obtain a list of device information and create a device.
```
private OBContext mOBContext;
```
Initialize the SDK and monitor device changes
```
// 1.Initialize the SDK, and listen for device changes
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
    @Override
    public void onDeviceAttach(DeviceList deviceList) {
        try {
            if (null == mPipeline) {
                // 2.Create Device, and initialize Pipeline through Device
                mDevice = deviceList.getDevice(0);
                mPipeline = new Pipeline(mDevice);
                
                // 3.Initialize the format conversion filter
                mFormatConvertFilter = new FormatConvertFilter();
                
                // 4.Create Pipeline Configuration
                Config config = new Config();
                
                // 5.Get the color stream configuration and configure it to Config
                StreamProfileList colorProfileList = mPipeline.getStreamProfileList(SensorType.COLOR);
                StreamProfile colorStreamProfile = null;
                if (null != colorProfileList) {
                    colorStreamProfile = colorProfileList.getVideoStreamProfile(640, 480, Format.MJPG, 0);
                    colorProfileList.close();
                }
                
                if (null == colorStreamProfile) {
                    Log.e(TAG, "onDeviceAttach: get color stream profile failed !");
                    deviceList.close();
                    return;
                }
                
                // 6.Enable color flow through the obtained color flow configuration
                config.enableStream(colorStreamProfile);
                
                // 7.Get the depth flow configuration and configure it to Config
                StreamProfileList depthProfileList = mPipeline.getStreamProfileList(SensorType.DEPTH);
                StreamProfile depthStreamProfile = null;
                if (null != depthProfileList) {
                    depthStreamProfile = depthProfileList.getVideoStreamProfile(640, 480, Format.Y16, 0);
                    depthProfileList.close();
                }
                
                if (null == depthStreamProfile) {
                    Log.e(TAG, "onDeviceAttach: get depth stream profile failed !");
                    deviceList.close();
                    return;
                }
                
                // 8.Enable depth streaming through the acquired depth streaming configuration
                config.enableStream(depthStreamProfile);
                
                // 9.Use Config to start Pipeline
                mPipeline.start(config);
                
                // 10.Release colorStreamProfile
                colorStreamProfile.close();
                
                // 11.Release depthStreamProfile
                depthStreamProfile.close();
                
                // 12.Release config resources
                config.close();
                
                // 13.Create a thread for getting Pipeline data and a thread for saving pictures
                start();
            }
            
            // 14.Release device list resources
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onDeviceDetach(DeviceList deviceList) {
        try {
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```
Create a device and create a Pipeline through the device
```
// 2.Create a device and create a Pipeline through the device
mDevice = deviceList.getDevice(0);
mPipeline = new Pipeline(mDevice);
```
Initialize format conversion filter through Pipeline
```
// 3.Initialize format conversion filter
mFormatConvertFilter = new FormatConvertFilter();
```
Create a pipeline configuration
```
// 4.Create a pipeline configuration
Config config = new Config();
```
Get the color stream profile, and configure it to Config, where the configuration is 640x480 MJPG format
```
// 5.Get the color stream profile list
StreamProfileList colorProfileList = mPipeline.getStreamProfileList(SensorType.COLOR);
StreamProfile colorStreamProfile = null;
if (null != colorProfileList) {
    colorStreamProfile = colorProfileList.getVideoStreamProfile(640, 480, Format.MJPG, 0);
    colorProfileList.close();
}

if (null == colorStreamProfile) {
    Log.e(TAG, "onDeviceAttach: get color stream profile failed !");
    deviceList.close();
    return;
}

// 6. Enable profile after obtaining the specified color profile
config.enableStream(colorStreamProfile);
```
Get the depth stream profile and configure it to Config, where the configuration is in 640x480 Y16 format
```
// 7.Get the depth flow configuration and configure it to Config
StreamProfileList depthProfileList = mPipeline.getStreamProfileList(SensorType.DEPTH);
StreamProfile depthStreamProfile = null;
if (null != depthProfileList) {
    depthStreamProfile = depthProfileList.getVideoStreamProfile(640, 480, Format.Y16, 0);
    depthProfileList.close();
}

if (null == depthStreamProfile) {
    Log.e(TAG, "onDeviceAttach: get depth stream profile failed !");
    deviceList.close();
    return;
}

//8. Enable profile after obtaining the specified depth profile
config.enableStream(depthStreamProfile);
```
Use Config to configure the stream through Pipeline
```
mPipeline.start(config);
```
Create a thread for obtaining Pipeline data and a thread for saving pictures
```
private void start() {
    colorCount = 0;
    depthCount = 0;
    mIsStreamRunning = true;
    mIsPicSavingRunning = true;
    if (null == mStreamThread) {
        mStreamThread = new Thread(mStreamRunnable);
        mStreamThread.start();
    }
    if (null == mPicSavingThread) {
        mPicSavingThread = new Thread(mPicSavingRunnable);
        mPicSavingThread.start();
    }
}
```
Data stream processing
```
while (mIsStreamRunning) {
    // After waiting for 100ms, if it cannot be obtained, it will time out
    FrameSet frameSet = mPipeline.waitForFrames(100);
    if (null == frameSet) {
        continue;
    }
    if (count < 5) {
        frameSet.close();
        count++;
        continue;
    }
    // Get color frame
     ColorFrame colorFrame = frameSet.getColorFrame();
        if (null != colorFrame) {
            mFormatConvertFilter.setFormatType(FormatConvertType.FORMAT_MJPEG_TO_RGB888);
            Frame rgbFrame = mFormatConvertFilter.process(colorFrame);
            
            FrameCopy frameT = copyToFrameT(rgbFrame.as(FrameType.VIDEO));
            
            mFrameSaveQueue.offer(frameT);
            colorFrame.close();
            rgbFrame.close();
        }
        
    // Get depth stream data
  DepthFrame depthFrame = frameSet.getDepthFrame();
        if (null != depthFrame) {
            FrameCopy frameT = copyToFrameT(depthFrame);
            mFrameSaveQueue.offer(frameT);
            depthFrame.close();
        }
    
    // Release frame set
           frameSet.close();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Data preservation
```
while (mIsPicSavingRunning) {
    try {
        FrameCopy frameT = mFrameSaveQueue.poll(300, TimeUnit.MILLISECONDS);
        if (null != frameT) {
            Log.d(TAG, "colorCount :" + colorCount);
            if (frameT.getStreamType() == FrameType.COLOR && colorCount < 5) {
                FileUtils.saveImage(frameT);
                colorCount++;
            }
            
            Log.d(TAG, "depthCount :" + depthCount);
            if (frameT.getStreamType() == FrameType.DEPTH && depthCount < 5) {
                FileUtils.saveImage(frameT);
                depthCount++;
            }
        }
    } catch (Exception e) {
    }
    
    if (colorCount == 5 && depthCount == 5) {
        mIsPicSavingRunning = false;
        break;
    }
}

mFrameSaveQueue.clear();
```
Exit data processing thread and image storage thread
```
private void stop() {
    mPipeline.stop();
    mIsStreamRunning = false;
    mIsPicSavingRunning = false;
    if (null != mStreamThread) {
        try {
            mStreamThread.join(1000);
        } catch (InterruptedException e) {
        }
        mStreamThread = null;
    }
    if (null != mPicSavingRunnable) {
        try {
            mPicSavingThread.join(1000);
        } catch (InterruptedException e) {
        }
        mPicSavingThread = null;
    }
}
```
Resource release
```
// Release format convert filter
if (null != mFormatConvertFilter) {
    try {
        mFormatConvertFilter.close();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
// Stop and close pipeline
if (null != mPipeline) {
    mPipeline.close();
    mPipeline = null;
}
// Release device
if (null != mDevice) {
    mDevice.close();
    mDevice = null;
}
// Release SDK resources
if (null != mOBContext) {
    mOBContext.close();
    mOBContext = null;
}
```
## SensorControl
Function description: This example mainly demonstrates the operation of device control commands and the operation of Sensor control commands.

First, you need to create a Context to obtain a list of device information and create a device.
```
private OBContext mOBContext;
```
Initialize the SDK and monitor device changes
```
// 2.Initialize SDK
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
    @Override
    public void onDeviceAttach(DeviceList deviceList) {
        try {
            // 3.Add the acquired device to the device list
            mDeviceList.add(deviceList.getDevice(0));
            
            // 4.Update device list
            updateDeviceSpinnerList();
            
            // 5.Release device list resources
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onDeviceDetach(DeviceList deviceList) {
        try {
            // A device is disconnected, release the device list resource
            for (Device device : mDeviceList) {
                device.close();
            }
            mDeviceList.clear();
            
            // Reacquire devices and update device list
            DeviceList curDeviceList = mOBContext.queryDevices();
            for (int i = 0; i < curDeviceList.getDeviceCount(); i++) {
                mDeviceList.add(curDeviceList.getDevice(i));
            }
            curDeviceList.close();
            updateDeviceSpinnerList();
            
            // No device connected, clear sensor list and property list
            if (mDeviceList.size() <= 0) {
                clearPropertySpinnerList();
            }
            
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```
Device connection callback method (onDeviceAttach) processing
```
try {
    // 3.Add the acquired device to the device list
    mDeviceList.add(deviceList.getDevice(0));
    
    // 4.Update device list
    updateDeviceSpinnerList();
    
    // 5.Release device list resources
    deviceList.close();
} catch (Exception e) {
    e.printStackTrace();
}
```
Device disconnection callback method (onDeviceDetach) processing
```
try {
    // A device is disconnected, release the device list resource
    for (Device device : mDeviceList) {
        device.close();
    }
    mDeviceList.clear();
    
    // Reacquire devices and update device list
    DeviceList curDeviceList = mOBContext.queryDevices();
    for (int i = 0; i < curDeviceList.getDeviceCount(); i++) {
        mDeviceList.add(curDeviceList.getDevice(i));
    }
    curDeviceList.close();
    updateDeviceSpinnerList();
    
    // No device connected, clear sensor list and property list
    if (mDeviceList.size() <= 0) {
        clearPropertySpinnerList();
    }
    
    deviceList.close();
} catch (Exception e) {
    e.printStackTrace();
}
```
Update property list based on selected device
```java
@Override
public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
    switch (parent.getId()) {
        case R.id.spi_device:
            // Device switching, update the transfer attribute list
            mSelectDevice = mDeviceList.get(position);
            updatePropertySpinnerList();
            
            String deviceName = mDeviceSp.getSelectedItem().toString();
            addNewMessage("select device name：" + deviceName);
            addNewMessage(getVersionInfo(mDeviceList.get(position)));
            // 其他case
    }
}

// Update property list
private void updatePropertySpinnerList() {
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            mDevicePropertyMap.clear();
            mPropertyNameList.clear();
            try {
                List<DevicePropertyInfo> devicePropertyList = mSelectDevice.getSupportedPropertyList();
                for (DevicePropertyInfo deviceProperty : devicePropertyList) {
                    if (deviceProperty.getPropertyType() != PropertyType.STRUCT_PROPERTY
                        && deviceProperty.getPermissionType() != PermissionType.OB_PERMISSION_DENY) {
                        mPropertyNameList.add(deviceProperty.getPropertyName());
                        mDevicePropertyMap.put(deviceProperty.getPropertyName(), deviceProperty);
                    }
                }
                addNewMessage("Select device command");
                mPropertyAdapter.clear();
                mPropertyAdapter.addAll(mPropertyNameList);
                mPropertyAdapter.notifyDataSetChanged();
                
                mPropertySp.setSelection(0, true);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    });
}
```
Update the set scope prompt based on the selected attribute
```java
@Override
public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
    switch (parent.getId()) {
        case R.id.spi_instructions:
            // command switch
            String instructionTypeName = mPropertySp.getSelectedItem().toString();
            PermissionType permissionType = mDevicePropertyMap.get(instructionTypeName).getPermissionType();
            addNewMessage("Select an operation command：" + instructionTypeName + " Read and write permissions：" + permissionType);
            updateControlPanel(permissionType);
            updateSetEditTextHint(instructionTypeName);
            break;
            // other cases
    }
}
```
Method for updating the property list based on the selected device
```java
private void updatePropertySpinnerList() {
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            mDevicePropertyMap.clear();
            mPropertyNameList.clear();
            try {
                List<DevicePropertyInfo> devicePropertyList = mSelectDevice.getSupportedPropertyList();
                for (DevicePropertyInfo deviceProperty : devicePropertyList) {
                    if (deviceProperty.getPropertyType() != PropertyType.STRUCT_PROPERTY
                        && deviceProperty.getPermissionType() != PermissionType.OB_PERMISSION_DENY) {
                        mPropertyNameList.add(deviceProperty.getPropertyName());
                        mDevicePropertyMap.put(deviceProperty.getPropertyName(), deviceProperty);
                    }
                }
                addNewMessage("select device command");
                mPropertyAdapter.clear();
                mPropertyAdapter.addAll(mPropertyNameList);
                mPropertyAdapter.notifyDataSetChanged();
                
                mPropertySp.setSelection(0, true);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    });
}
```
Updates the setting scope prompt for the attribute based on the selected attribute
```java
private void updateSetEditTextHint(String instructionTypeName) {
    mSetEt.setText("");
    mGetTv.setText("");
    try {
        switch (mDevicePropertyMap.get(instructionTypeName).getPropertyType()) {
            case INT_PROPERTY:
                int minI = mSelectDevice.getMinRangeI(mDevicePropertyMap.get(instructionTypeName).getProperty());
                int maxI = mSelectDevice.getMaxRangeI(mDevicePropertyMap.get(instructionTypeName).getProperty());
                mSetEt.setHint("[" + minI + "-" + maxI + "]");
                break;
            case BOOL_PROPERTY:
                mSetEt.setHint("[" + 0 + "-" + 1 + "]");
                break;
            case FLOAT_PROPERTY:
                float minF = mSelectDevice.getMinRangeF(mDevicePropertyMap.get(instructionTypeName).getProperty());
                float maxF = mSelectDevice.getMaxRangeF(mDevicePropertyMap.get(instructionTypeName).getProperty());
                mSetEt.setHint("[" + minF + "-" + maxF + "]");
                break;
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Set properties
```java
try {
    String setValue = mSetEt.getText().toString();
    // set device directive
    DevicePropertyInfo devProperty = mDevicePropertyMap.get(mPropertySp.getSelectedItem().toString());
    switch (devProperty.getPropertyType()) {
        case INT_PROPERTY:
            //Exposure and gain need to be turned off to adjust AE, brightness needs to be turned on AE to be adjusted, white balance (color temperature) needs to be turned off automatic white balance to adjust
            if (devProperty.getProperty() == DeviceProperty.OB_PROP_COLOR_EXPOSURE_INT
                || devProperty.getProperty() == DeviceProperty.OB_PROP_COLOR_GAIN_INT) { //曝光或增益
                //Get auto exposure status
                boolean propertyExposureBool = false;
                try {
                    propertyExposureBool = mSelectDevice.getPropertyValueB(DeviceProperty.OB_PROP_COLOR_AUTO_EXPOSURE_BOOL);
                } catch (Exception e) {
                    
                }
                Log.d(TAG, "propertyExposureBool:" + propertyExposureBool);
                if (propertyExposureBool) {
                    showToast("Auto exposure is not turned off, exposure or gain cannot be set");
                    return;
                }
            }
            
            if (devProperty.getProperty() == DeviceProperty.OB_PROP_COLOR_BRIGHTNESS_INT) { //brightness
                //Get auto exposure status
                boolean propertyExposureBool = mSelectDevice.getPropertyValueB(DeviceProperty.OB_PROP_COLOR_AUTO_EXPOSURE_BOOL);
                Log.d(TAG, "propertyExposureBool:" + propertyExposureBool);
                if (!propertyExposureBool) {
                    showToast("Auto exposure is not turned on, and brightness cannot be set");
                    return;
                }
            }
            
            if (devProperty.getProperty() == DeviceProperty.OB_PROP_COLOR_WHITE_BALANCE_INT) { //白平衡
                //Get auto white balance status
                boolean propertyWhiteBool = mSelectDevice.getPropertyValueB(DeviceProperty.OB_PROP_COLOR_AUTO_WHITE_BALANCE_BOOL);
                Log.d(TAG, "propertyWhiteBool:" + propertyWhiteBool);
                if (propertyWhiteBool) {
                    showToast("Auto white balance is not turned off, cannot set white balance");
                    return;
                }
            }
            mSelectDevice.setPropertyValueI(devProperty.getProperty(), Integer.parseInt(setValue));
            break;
        case BOOL_PROPERTY:
            // 0:false 1:true
            mSelectDevice.setPropertyValueB(devProperty.getProperty(), ("1".equals(setValue) ? true : false));
            break;
        case FLOAT_PROPERTY:
            mSelectDevice.setPropertyValueF(devProperty.getProperty(), Float.parseFloat(setValue));
            break;
    }
} catch (Exception e) {
    e.printStackTrace();
}
```
Get attribute
```java
try {
    // Get device instructions
    DevicePropertyInfo devProperty = mDevicePropertyMap.get(mPropertySp.getSelectedItem().toString());
    switch (devProperty.getPropertyType()) {
        case INT_PROPERTY:
            int valueI = mSelectDevice.getPropertyValueI(devProperty.getProperty());
            mGetTv.setText(Integer.toString(valueI));
            break;
        case BOOL_PROPERTY:
            boolean valueB = mSelectDevice.getPropertyValueB(devProperty.getProperty());
            mGetTv.setText(Boolean.toString(valueB));
            break;
        case FLOAT_PROPERTY:
            float valueF = mSelectDevice.getPropertyValueF(devProperty.getProperty());
            mGetTv.setText(Float.toString(valueF));
            break;
    }
} catch (Exception e) {
    e.printStackTrace();
}
```
Resource release
```
private void release() {
    try {
        // Device List Resource Release
        for (Device device : mDeviceList) {
            device.close();
        }
        mDeviceList.clear();
        
        // OBContext resource release
        if (null != mOBContext) {
            mOBContext.close();
            mOBContext = null;
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
## Recorder & Playback
Function description: : Connect the device to open the stream, record the current video stream to a file, and load the video file for playback.

First, you need to create a Context to get a list of device information and create a device
```java
private OBContext mOBContext;
```
Initialize the SDK and listen for device changes
```java
// 1.Initialize the SDK, and listen for device changes
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
    @Override
    public void onDeviceAttach(DeviceList deviceList) {
        try {
            StringBuilder builder = new StringBuilder();
            // 2.Check SDK version
            builder.append("SDK Version = " + OBContext.getVersionName() + "\n");
            
            // 3.Get the number of devices
            int deviceCount = deviceList.getDeviceCount();
            
            for (int i = 0; i < deviceCount; ++i) {
                // 4.Create device from device index
                Device device = deviceList.getDevice(i);
                
                // 5.Get device information
                DeviceInfo info = device.getInfo();
                
                // 6.Get device version information
                builder.append("Name: " + info.getName() + "\n");
                builder.append("Vid: " + LocalUtils.formatHex04(info.getVid()) + "\n");
                builder.append("Pid: " + LocalUtils.formatHex04(info.getPid()) + "\n");
                builder.append("Uid: " + info.getUid() + "\n");
                builder.append("SN: " + info.getSerialNumber() + "\n");
                String firmwareVersion = info.getFirmwareVersion();
                builder.append("FirmwareVersion: " + firmwareVersion + "\n");
                String hardwareVersion = info.getHardwareVersion();
                builder.append("HardwareVersion: " + hardwareVersion + "\n");
                
                // 7.Iterate over the sensors of the current device
                for (Sensor sensor : device.querySensors()) {
                    // 8.Query sensor type
                    builder.append("Sensor : type = " + sensor.getType() + "\n");
                }
                runOnUiThread(() -> txtInfo.setText(builder.toString()));
                
                // 9.Release device information
                info.close();
                
                // 10.Free up device resources
                device.close();
            }
            // 11.Release device list resources
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onDeviceDetach(DeviceList deviceList) {
        try {
            deviceList.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```
Create a device and create a Pipeline from the device
```java
// 2.Create a device and create a Pipeline from the device
mDevice = deviceList.getDevice(0);
mPipeline = new Pipeline(mDevice);
```
Create Pipeline Configuration
```java
// 4.Create Pipeline Configuration
mConfig = new Config();
```
Get the depth stream configuration and configure it to Config, where the configuration is in 640x480 Y16 format
```java
// 5.Get the depth stream configuration and configure it to Config, the configuration here is 640x480 Y16 format
StreamProfileList depthProfileList = mPipeline.getStreamProfileList(SensorType.DEPTH);
StreamProfile streamProfile = null;

if (null != depthProfileList) {
    streamProfile = depthProfileList.getVideoStreamProfile(640, 480, Format.Y16, 30);
    depthProfileList.close();
}

if (streamProfile == null) {
    return;
}

// 6.Enable depth stream configuration
mConfig.enableStream(streamProfile);
```
Setting mirroring through the Device attribute interface
```java
// 7.Set mirror
if (mDevice.isPropertySupported(DeviceProperty.OB_PROP_DEPTH_MIRROR_BOOL, PermissionType.OB_PERMISSION_WRITE)) {
    mDevice.setPropertyValueB(DeviceProperty.OB_PROP_DEPTH_MIRROR_BOOL, true);
}
```
Use Config configuration to open flow through Pipeline and release local resources
```java
// 8.open flow
mPipeline.start(mConfig);

// 9.free streamProfile
streamProfile.close();
```
Start the data stream preview thread
```java
private void start() {
	mIsStreamRunning = true;
	if (null == mStreamThread) {
		mStreamThread = new Thread(mStreamPrevRunnable);
		mStreamThread.start();
	}
}
```
Preview thread
```java
// preview thread
private Runnable mStreamPrevRunnable = () -> {
	while (mIsStreamRunning) {
		try {
			// After waiting for 100ms, if it can't get it, it will time out
			FrameSet frameSet = mPipeline.waitForFrameSet(100);

			if (null == frameSet) {
				continue;
			}

			// Get depth stream data
			if (!mIsPlaying) {
				DepthFrame frame = frameSet.getDepthFrame();

				if (frame != null) {
					// Get depth data and render
					byte[] frameData = new byte[frame.getDataSize()];
					frame.getData(frameData);
					synchronized (mSync) {
						mDepthGLView.update(frameData, frame.getWidth(), frame.getHeight(), StreamType.DEPTH, frame.getFormat());
					}

					// release depth data frame
					frame.close();
				}
			}

			// free dataset
			frameSet.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
};
```
Define the recording file save path
```java
private static final String BAG_FILE_PATH = "/sdcard/Orbbec/recorder.bag";
```
Stop recording
```java
private void startRecord() {
	try {
		if (!mIsRecording) {
			if (null != mPipeline) {
				// stop recording
				mPipeline.startRecord(BAG_FILE_PATH);
				mIsRecording = true;
				updateCtlPanel(mPlaybackCtlPanelLL, View.INVISIBLE);
			} else {
				Log.w(TAG, "mPipeline is null !");
			}
		}
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```
Stop recording
```java
private void stopRecord() {
	try {
		if (mIsRecording) {
			mIsRecording = false;
			if (null != mPipeline) {
				// stop recording
				mPipeline.stopRecord();
			}
			updateCtlPanel(mPlaybackCtlPanelLL, View.VISIBLE);
		}
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```
Stop the stream preview thread
```java
private void stop() {
	mIsStreamRunning = false;
	if (null != mStreamThread) {
		try {
			mStreamThread.join(300);
		} catch (InterruptedException e) {
		}
		mStreamThread = null;
	}
}
```
Define playback status callback
```java
private MediaStateCallback mMediaStateCallback = new MediaStateCallback() {
    @Override
    public void onState(MediaState state) {
        if (state == MediaState.OB_MEDIA_END) {
            stopPlayback();
        }
    }
};
```
Define the Runnable of the playback thread
```java
while (mIsPlaying) {
	try {
		// After waiting for 100ms, if it can't get it, it will time out
		FrameSet frameSet = mPlaybackPipe.waitForFrameSet(100);
		if (null == frameSet) {
			continue;
		}

		// Get depth stream data
		DepthFrame depthFrame = frameSet.getDepthFrame();
		if (null != depthFrame) {
			// Get depth data and render
			byte[] frameData = new byte[depthFrame.getDataSize()];
			depthFrame.getData(frameData);
			synchronized (mSync) {
				mDepthGLView.update(frameData, depthFrame.getWidth(), depthFrame.getHeight(), StreamType.DEPTH, depthFrame.getFormat());
			}

			// release depth data frame
			depthFrame.close();
		}

		// free dataset
		frameSet.close();
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```
Start playback
```java
private void startPlayback() {
    try {
        if (!FileUtils.isFileExists(BAG_FILE_PATH)) {
            Toast.makeText(RecordPlaybackActivity.this, "File not found!", Toast.LENGTH_LONG).show();
            return;
        }
        if (!mIsPlaying) {
            mIsPlaying = true;

            // Release Playback resources
            if (null != mPlayback) {
                mPlayback.close();
                mPlayback = null;
            }

            // Create a playback pipeline
            if (null != mPlaybackPipe) {
                mPlaybackPipe.close();
                mPlaybackPipe = null;
            }
            mPlaybackPipe = new Pipeline(BAG_FILE_PATH);

            // Get the replayer by replaying the Pipeline
            mPlayback = mPlaybackPipe.getPlayback();

            // Set playback status callback
            mPlayback.setMediaStateCallback(mMediaStateCallback);

            // start playback
            mPlaybackPipe.start(null);

            // Create a playback thread
            if (null == mPlaybackThread) {
                mPlaybackThread = new Thread(mPlaybackRunnable);
                mPlaybackThread.start();
            }

            // Update UI
            updateDeviceInfoView(true);
            updateCtlPanel(mRecordCtlPanelLL, View.INVISIBLE);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Stop playback
```java
private void stopPlayback() {
    try {
        if (mIsPlaying) {
            mIsPlaying = false;
            // stop playback thread
            if (null != mPlaybackThread) {
                try {
                    mPlaybackThread.join(300);
                } catch (InterruptedException e) {
                }
                mPlaybackThread = null;
            }

            // stop playback
            if (null != mPlaybackPipe) {
                mPlaybackPipe.stop();
            }
            
            // Update UI
            updateDeviceInfoView(false);
            updateCtlPanel(mRecordCtlPanelLL, View.VISIBLE);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Release resources
```java
try {
    // release the player
    if (null != mPlayback) {
        mPlayback.close();
        mPlayback = null;
    }

    // Release the playback pipeline
    if (null != mPlaybackPipe) {
        mPlaybackPipe.close();
        mPlaybackPipe = null;
    }

    //  Release config
    if (null != mConfig) {
        mConfig.close();
        mConfig = null;
    }

    // Release preview pipeline
    if (null != mPipeline) {
        mPipeline.close();
        mPipeline = null;
    }

    // Release Device
    if (null != mDevice) {
        mDevice.close();
        mDevice = null;
    }
    
    // Release SDK
    if (null != mOBContext) {
        mOBContext.close();
    }

    // Free render window resources
    if (mDepthGLView != null) {
        mDepthGLView.release();
    }
} catch (Exception e) {
    e.printStackTrace();
}
```



