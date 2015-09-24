---
author: Alishex
comments: true
date: 2014-07-07 13:03:37+00:00
layout: post
slug: android-fundementals
title: Android Fundementals
wordpress_id: 56
categories:
- Other
tags:
- android
- android fundementals
- java
---

Android applications are made out of components. There are four Android components.


* Activity
* Service
* Broadcast Reciever
* Content Provider


Applications are typically created from multiple components which Android instantiates and runs them as needed where each of these components serves a different purpose in the Android ecosystem.

**Activity**

* Primary class for user interactions
* Serves a single focused purpose


**Service**

* Run in background
* Used for long running operations
* To support operations with processes


**BroadcastReciever**

* Component that listens for and responds to an event
* Servers as a Subscriber in Publish/Subscribe pattern. Similar to Observer pattern


**ContentProvider**

* Used to store&share data across applications
* Uses database-style interface
* Handles interprocess communication


Remember that your Android applications are much more then just source codes. There are pleanty of non-source code entities such as drawables, animations, layouts, values such as string and styles and finally the manifest.

One more thing to remember is generated code, such as famous **R.java**.R.java is a class that is automaticly created by Android to address the non-sourcecode entities. Whenever you create a layout, a drawable etc, Android generates a static value in R.java to later address that in source code.

**Building An Application**
Building an application in Android is a multiphased proccess. First your code will be compiled with javac into ".class" files so bytecode. These bytecodes will be converted by "dx" to ".dex" files which are Dalvik Executables. After this your .Dalvik executables will be archived into an Android Package(".apk"). So basicly it is as such :


**Java(.java) -> Bytecode(.class) -> Dalvik Executables(.dex) -> Android Package(.apk)**
