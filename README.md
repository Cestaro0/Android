# Android
My notes about android, programming, reverse enginnering.

The document is talk about the following tópics:
 - Android architeture
 - Reverse Engineering whit IDA PRO

## Android architeture
First is necessary understand that android platform is based in linux kernel.
See the image below.
<img src="https://github.com/Cestaro0/Android/assets/99103680/6a73d8cd-4008-4e16-9dba-d4e1175cf083"/>
This image covers many layers, and we'll discuss each one.
 ### Linux Kernel
  As mentioned before, Android is based on the Linux kernel, which provides various features such as security levels, permissions, and user management.
 ### HAL
  HAL is the standard interface for implementing hardware on Android. It is independent of lower-level drivers.
 ### Android runtime
 This part is responsible for executing Android apps. Each program runs in its own virtual machine in .DEX (Dalvik Executable Format). App compilation can occur in two ways: AOT (Ahead-Of-Time) and JIT (Just-In-Time).
 ### Native C/C++ libraries
  This part, which I find more interesting (as I love reverse engineering), involves many components in Android created in native code (C/C++). We can also create our own .so files in C++ for Android applications.
 ### Java API
  Similar to native C/C++ libraries, the Java API utilizes Java packages for GUI or app functionalities with exclusive characteristics for Android.
 ### System apps
  Android comes with a basic set of apps for email, messaging, calendar, web browsing, contacts, and more.



## Reverse Engineering
 This topic is more detailed because I love reverse engineering and it's my area of expertise. We'll cover the following:
  - ABD
  - ROOT
  - APK
  - JNI
  - Analisys whit IDA
   - Static
   - Dinamic
    - Debug  
   

### ABD
ADB (Android Debug Bridge) is a command-line interface that allows direct communication and control of Android devices connected to a host computer. This versatile tool enables developers to access advanced Android OS features and perform various essential tasks such as installing/uninstalling apps, transferring files, diagnosing problems, simulating touch events, etc. It works via USB or TCP. The most used commands are:
```
adb root
adb shell
adb push
adb pull
adb reboot
adb reboot bootloader
```

### ROOT
 Since Android is based on the Linux kernel, the command line is extensively used. With user permissions, "root" is the level with all permissions (similar to sudo). To root a device, it's necessary to grant access to "img.boot" (a modified image for root access) specific to our device and use the "fastboot" tool. After connecting the device to the PC, use adb reboot bootloader to boot the device into recovery mode, then use fastboot boot "boot.img". After that, our device is rooted.

 ### APK
 APK is the extension for Android applications. To access an APK file on a connected device, use adb push "path to the app"; then, on the PC, the APK will be accessible as "base.apk". The structure of the .apk file is as follows:
```
  .apk
    |- extension for lib C/C++
    |            |- libname.so
    |- .dex classes
    |- .xml
```

  ### JNI
   Now, let's talk about JNI (Java Native Interface). To put it simply, it's a way that Oracle created for Java to communicate with C++. There's also JNA, maintained by a GitHub community (but we won't discuss it here). Below is an example image:
<img src="https://github.com/Cestaro0/Android/assets/99103680/f9ac3384-ea7c-4331-9fcb-a09f59a3aeb4">
Here, you can see that JNI acts as an intermediary between the JVM and the C++ .so where it runs the Java code, which in turn calls C++ functions. The .so is created with a mix of Java and C++ in a JNI header <a href="https://github.com/Cestaro0/How-To-Use-JNI">see more</a>

### Analisys whit IDA
### static
When unpacking the .apk file, we can find the .so JNI folders and analyze them. Reverse engineering .so files is relatively simple, but a plugin is required:
  - jni_all.h
in the arm64-v8a (x64 architeture) we don't have the problem of rebuilding structures.
so you will often come across this:
 ```c++
void foo(__int64 a1, __int64 a2, __int8* a3)
{
 wchar_t* a4;
 a4 = (wchar_t*)(**a1 + 123)(a1, a3, 0)
}
 ```
The analysis they is so complicated, but not be afraid, just click on the variable and press "y"
to display is:
```c++
__int64 a1
```
Modify it to:
```c++
JNIEnv* a1
```
And it works like magic:
```c++
a4 = (*a1)->NewStringUTF(env, a3, 0)
```
after, press "n" and rename the variable for conventional name

```c++
env
```
now see how it turned out after fixes
```
void foo(JNIEnv* env, jclass clazz, __int8* str)
{
 wchar_t* a4;
 a4 = (*env)->NewStringUTF(env, str, 0)
}
```
In x32 it is more complicated as it does not accept this modification, so you must use this header, on ida access
File > Load File > Parse C header file
and choose the jni_all.h donwload in this <a href="https://gist.github.com/jcalabres/bf8d530b3f18c30ca6f66388357b1d91">link</a>
before, use between x64.
note: the header will not work if there is any protection in the .so

### debug / dinamic
Debugging Android applications in IDA Pro involves converting the application's executable code, present in the APK file, into a readable and structured format. With the resulting .so file, which contains the code in C/C++ format, the analyst can open the lib in IDA Pro and explore its structure, identify functions and understand the execution flow. The ability to set breakpoints and pause application execution at specific times makes it possible to drill down into the device's memory and registers while running.

Preparation at IDA
Load the lib you are going to analyze into IDA and select the desired breakpoints, after that we will move on to the part where we use ADB.

Preparation with ABD
In your IDA Pro folder, you will find the "dbgsrv" folder, inside it we will find "android_server" (for armeabi-v7a x32 cell phones) and "android_server64" (for arm64-v8a x64 cell phones), according to the architecture of your device, move one of the files to the folder where ADB is. After that, we open the terminal inside the ADB folder and execute the following commands, with the cell phone connected to the PC via the USB cable with USB debugging activated on the device:
```
C:\platform-tools\> adb devices

C:\platform-tools\> adb push <android_server> /data/local/tmp

C:\platform-tools\> adb shell
```
Now on the device terminal
```
android:/ $ su

android:/ # chmod 755 /data/local/tmp/<android_server>

android:/ # ./data/local/tmp/<android_server>

After that, minimize this terminal and open another one from adb, let's redirect the ports so we can debug with IDA:

C:\platform-tools\> adb forward tcp:23946 tcp:23946
```
Starting debugging
Within IDA Pro, in the "Debugger>Select debugger" tab, select "Remote ARM Linux/Android debugger" and click OK.

Now in the "Debugger>Process options..." tab, in the "Hostname" you will enter 127.0.0.1 and in the "Port" you will enter the same number generated in the previous cmd, after that click OK.

Okay, basically all the configurations have been made and everything is ready to start debugging. With the application you want to debug open on your Android device, go to IDA Pro in the "Debugger>Attach to process..." tab to access the processes running on your device, find the process of the application you want to debug, From then on, it will freeze the application with it paused, press F9 and then continue with the application, when it reaches its breakpoint it will pause and you can debug using IDA Pro (Remember to keep "Use source-level debugging" activated to breakpoint function).
