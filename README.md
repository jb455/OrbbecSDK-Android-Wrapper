
# Quick start
## download source
```
git clone https://github.com/orbbec/OrbbecSDK-Android-Wrapper.git
```

## import project
1. Open Android studio
2. Menu: File --> open, and select project directory
3. Click Ok button
4. wait gradle sync complete

![](doc/readme-images/Open-module-Android-wrapper.png)

## run example

### build example
![](doc/readme-images/run-example.png)

### Main UI
![](doc/readme-images/Example-HelloOrbbec.png)

Click 'DepthViewer' to show depth sensor stream.

### DepthViewer
![](doc/readme-images/Example-DepthViewer.png)

# Build Tools
## Android studio
Android studio **Giraffe | 2022.3.1 Patch 1**
download link [Android studio](https://developer.android.com/studio)

## NDK
**version:** 21.4.7075529

## CMake
**version:** 3.18.1

## gradle
gradle/wrapper/gradle-wrapper.properties
```txt
distributionUrl=https\://services.gradle.org/distributions/gradle-8.0-bin.zip
```

## gradle pulgins
build.gradle
```groovy
plugins {
id 'com.android.application' version '8.1.0' apply false
    id 'org.jetbrains.kotlin.android' version '1.8.10' apply false
    id 'com.android.library' version '8.1.0' apply false
}
```

# library and example
## sensorsdk
Orbbec basic sdk implementation with c & cpp, android wrapper import sensorsdk as so.

## so files
**path:** obsensor_jni/libs

## include headers
**path:** obsensor_jni/src/main/cpp/sensorsdk

_Note_: To short include path in native code, module of 'obsensor_jni' import as path 'obsensor_jni/src/main/cpp/sensorsdk/include/libobsensor'

## obsensor_jni
Android wrapper implementation with jni which is forward or transfer data between java and native sensorsdk. 

## Support android version
```groovy
minSdk 24
//noinspection ExpiredTargetSdkVersion
targetSdk 27
```
**targetSdkVersion** 27 to fixed bug 'Android 10 Devices Do NOT Support USB Camera Connection' which fixed on android 11.
\[reference 01] [Android 10 sdk28 usb camera device class bug.](https://forums.oneplus.com/threads/android-10-sdk28-usb-camera-device-class-bug.1258389/)

\[reference 02] [Android 10 Devices Do NOT Support USB Camera Connection.](https://www.camerafi.com/notice-android-10-devices-do-not-support-usb-camera-connection/)

## example
Example of sensorsdk android wrapper

## Support android version
```groovy
minSdk 24
//noinspection ExpiredTargetSdkVersion
targetSdk 27
```
**targetSdkVersion** 27 to fixed bug 'Android 10 Devices Do NOT Support USB Camera Connection' which fixed on android 11.

# Support orbbec device
OrbbecSDK：v1.8.1
Publish: 2023-11-16
Support device list (firmware version):
|Class|Product|Firmware|
|-|-|-|
|UVC Device|Astra+ & Astra+s|V1.0.20|
||Femto|V1.6.9|
||Femto-W|V1.1.8|
||Femto-Live|V1.1.1|
||Astra2|V2.8.20|
||Gemini2|V1.4.60|
||Gemini2L|V1.4.32|
||Gemini2XL|Obox: V2.0.1 VL:1.4.56|
|OpenNI|Gemini||
||Dabai DW||
||Dabai DCW||
||Dabai DC1||
||Astra Mini||
||AstraMini S||
||Astra Mini Pro||
||Dabai||
||Dabai Pro||
||Deeya||
||Astra Plus||
||Dabai D1||
||A1 Pro||
||Gemini E||
||Gemini E Lite||

# Quick Start
Create OBContext global member to manager attach devices
```java
// Application hold only one OBContext instance.
private OBContext mOBContext;
private Object mCurrentDeviceLock = new Object();
private Device mCurrentDevice;
private DeviceInfo mCurrentDeviceInfo;
```

Initialize OBContext with DeviceChangedCallback
```java
mOBContext = new OBContext(getApplicationContext(), new DeviceChangedCallback() {
   @Override
   public void onDeviceAttach(DeviceList deviceList) {
         synchronized (mCurrentDeviceLock) {
            if (null == mCurrentDevice) {
               // DeviceList#getDevice(index) can only call once inside onDeviceAttach()
               mCurrentDevice = deviceList.getDevice(0);
               mCurrentDeviceInfo = mCurrentDevice.getInfo();
               Log.d("Orbbec", "Device connection. name: " + mCurrentDeviceInfo.getName() + ", uid: " + mCurrentDeviceInfo.getUid());
            }
         }
         try {
            deviceList.close();
         } catch (Exception e) {
            e.printStackTrace();
         }
   }

   @Override
   public void onDeviceDetach(DeviceList deviceList) {
         try {
            int deviceCount = deviceList.getDeviceCount();
            for (int i = 0; i < deviceCount; i++) {
               String uid = deviceList.getUid();
               if (null != mCurrentDevice) {
                  synchronized (mCurrentDeviceLock) {
                      if (null != mCurrentDeviceInfo && mCurrentDeviceInfo.getUid().equals(uid)) {
                           // handle device disconnection
                           // do something

                           Log.d("Orbbec", "Device disconnection. name: " + mCurrentDeviceInfo.getName() + ", uid: " + mCurrentDeviceInfo.getUid());
                           mCurrentDevice.close();
                           mCurrentDevice = null;

                           mCurrentDeviceInfo.close();
                           mCurrentDeviceInfo null;
                      }
                  } // synchronized
               }
            } // for
         } catch (Exception e) {
            e.printStackTrace();
         }

         try {
            deviceList.close();
         } catch (Exception e) {
            e.printStackTrace();
         }
   }
});
```

Define Pipeline and Device
```java
private Pipeline mPipeline;
```

Start Depth stream
```java
try {
   mPipeline = new Pipeline(mCurrentDevice);
   StreamProfileList depthProfileList = mPipeline.getStreamProfileList(SensorType.DEPTH);
   StreamProfile streamProfile = depthProfileList.getStreamProfile(0);
   if (null != depthProfileList) {
      depthProfileList.close();
      return;
   }

   Config config = new Config();
   mPipeline.start(config, new FrameSetCallback() {
      public void onFrameSet(FrameSet frameSet) {
         DepthFrame depthFrame = frameSet.getDepthFrame()
         if (null != depthFrame) {
            Log.d("Orbbec", "onFrameSet depthFrame index: " + depthFrame.getFrameIndex() + ", timeStamp: " + depthFrame.getTimeStamp());

            // do Render

            depthFrame.close();
         }
      }
   });
   config.close();
} catch (OBException e) {
   e.printStackTrace();
}
```

Stop stream
```java
try {
   mPipeline.stop();
   mPipeline.close();
   mPipeline = null;
} catch (OBException e) {
   e.printStackTrace();
}
```



# QA
## LintModelSeverity has been compiled by a more recent version of the Java 
```
An exception occurred applying plugin request [id: 'com.android.application']
> Failed to apply plugin 'com.android.internal.application'.
   > Could not create an instance of type com.android.build.gradle.internal.dsl.ApplicationExtensionImpl$AgpDecorated.
      > Could not create an instance of type com.android.build.gradle.internal.dsl.LintImpl$AgpDecorated.
         > Could not generate a decorated class for type LintImpl$AgpDecorated.
            > com/android/tools/lint/model/LintModelSeverity has been compiled by a more recent version of the Java Runtime (class file version 61.0), this version of the Java Runtime only recognizes class file versions up to 55.0

* Try:
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.
```

**Reason**: AGP(android gradle plugin) is v8.1.0, need jdk 17, Please update android studio to new version, and check gradle jdk version.
Android studio --> File --> Settings --> Build,Execution,Deployment --> Build Tools --> Gradle, check Gradle Projects -> Gradle JDK 

**reference**: 
https://developer.android.com/studio/releases#android_gradle_plugin_and_android_studio_compatibility

https://developer.android.com/build/releases/gradle-plugin