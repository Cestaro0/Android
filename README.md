# Android
My notes about android, programming, reverse enginnering.

The document is talk about the following tópics:
 - Android architeture
 - Reverse Engineering whit IDA PRO

## Android architeture
First is necessary understand that android platform is based in linux kernel.
See the image below.
<img src="https://github.com/Cestaro0/Android/assets/99103680/6a73d8cd-4008-4e16-9dba-d4e1175cf083"/>
This image cover a lot layers, and we'll talk about all.
 ### Linux Kernel
  As said before, the android is based in linux kernel, for more optios that he offers, as security levels, permissions, management of the users.
 ### HAL
  Hal is the standard interface of implementation the hardware for the android it's independent for down levels drivers.
 ### Android runtime
  This part is relative for execution the android apps, wich works as follow: each program will have the own virtual machine in .DEX (dalvik executable format).
 This compilation of app may happen for two ways: AOT(compilation advance) and JIT(just in time).
 ### Native C/C++ libraries
  For my, this part is more interesting (i love reverse engineering). More components in android they created in native code (C/C++). And we can create our own .so in C++ for android application (let's talk about this later).
 ### Java API
  Like this native C/C++ libraries, Java API is a bit more different, but consist in usage java packages for GUI or functionalities in own app, with characteristics exclusives to android.
 ### System apps
  Android comes with a basic set of apps for email, messaging, calendar, web browsing, contacts, and more.



## Reverse Engineering
 This topic is more detailed because I love RE and is my area of ​​activity. We'll talking for this:
  - ABD
  - ROOT
  - APK
  - JNI
  - Analisys whit IDA
   - Static
   - Dinamic
    - Debug  
   

### ABD
It is a command-line interface that allows direct communication and control of Android devices connected to a host computer. This versatile tool allows developers to access advanced Android operating system features and perform a variety of essential tasks such as installing and uninstalling apps, transferring files, diagnosing problems, simulating touch events, and more. It works via USB or TCP. most used commands: adb root, adb shell, adb push, adb pull, adb reboot, adb reboot bootloader.

### ROOT
 The android is based on linux kernel, so command line is more used. And exists the user permissions, the root is the level with all permission (sudo).
 To get root in this device is necessary granted the "img.boot" (modified image for root access) relative our dispositive and use "fastoot" tool. Firstly connect the device in this PC, and use adb reboot bootloader (for the device in mode recovery) after use fastboot boot "boot.img", ready, our device is rooted!

 ### APK
 APK is the extension of the app, like this .jar or .zip, then in our device connected in PC use adb push "path to the app" with in this PC will have "base.apk". The .apk struct is: 
  .apk
    |- extension for lib C/C++
    |            |- libname.so
    |- .dex classes
    |- .xml


  ### JNI
   Now, let's talk about JNI (Java Native Interface). To put it simply, it's a way that Oracle created for Java to communicate with C++. There's also JNA, maintained by a GitHub community (but we won't discuss it here). Below is an example image:
<img src="https://github.com/Cestaro0/Android/assets/99103680/f9ac3384-ea7c-4331-9fcb-a09f59a3aeb4">
Here, you can see that JNI acts as an intermediary between the JVM and the C++ .so where it runs the Java code, which in turn calls C++ functions. The .so is created with a mix of Java and C++ in a JNI header <a href="https://github.com/Cestaro0/How-To-Use-JNI">see more</a>

### Analisys whit IDA
### static
 When descompact the .apk file, in the folders we can find the .so jni, and analysis it!
 The reverse engineering in .so is so simple, but is necessary this plugin:
  - jni_all.h
in the arm64-v8a (x64 architeture) we don't have the problem of rebuilding structures.
so you will often come across this:
 ```c++
void foo(__int64 a1, __int64 a2, __int8* a3)
{
 wchar_t* a4;
 a4 = (wchar_t*)(**a1 + 123)(a1, a3)
}
 ```
The analysis they is so complicated, but not be afraid, just click on the variable and press "y"
the display is:
```c++
__int64 a1
```
just modify to
```c++
JNIEnv* a1
```
it's the magic 

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

C:\platform-tools\> adb devices

C:\platform-tools\> adb push <android_server> /data/local/tmp

C:\platform-tools\> adb shell

Now on the device terminal

android:/ $ su

android:/ # chmod 755 /data/local/tmp/<android_server>

android:/ # ./data/local/tmp/<android_server>

After that, minimize this terminal and open another one from adb, let's redirect the ports so we can debug with IDA:

C:\platform-tools\> adb forward tcp:23946 tcp:23946

Starting debugging
Within IDA Pro, in the "Debugger>Select debugger" tab, select "Remote ARM Linux/Android debugger" and click OK.

Now in the "Debugger>Process options..." tab, in the "Hostname" you will enter 127.0.0.1 and in the "Port" you will enter the same number generated in the previous cmd, after that click OK.

Okay, basically all the configurations have been made and everything is ready to start debugging. With the application you want to debug open on your Android device, go to IDA Pro in the "Debugger>Attach to process..." tab to access the processes running on your device, find the process of the application you want to debug, From then on, it will freeze the application with it paused, press F9 and then continue with the application, when it reaches its breakpoint it will pause and you can debug using IDA Pro (Remember to keep "Use source-level debugging" activated to breakpoint function).













References:
<a href="https://developer.android.com/guide/platform">Android architeture</a>
