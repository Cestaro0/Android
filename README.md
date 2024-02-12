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


































References:
<a href="https://developer.android.com/guide/platform">Android architeture</a>
