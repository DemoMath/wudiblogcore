---
layout: post
title:  "Android 关于 so 所需要了解的"
date:   2017-04-14
categories: [Android,NDK开发]
tag: [Android]
---
- 什么是so文件：
           
        so是Shared Object的缩写，即共享的对象，机器可以直接运行的二进制码，
        从操作系统到专用软件，都离不开so。
        so文件主要用于Unix和Linux系统中。
        

- .a和.so对比
	- .a:archive

			存档的含义，是unix系统中对于静态库的文件后缀，在软件打包时和主程序表态链接在一起，表现形式是在链接成同一个文件。
			go lang即广泛采用这一形式，对于软件分发只有一个文件。对于打包好的软件来讲，这是专属库，所有都在出厂前打包在一起了，好处是不受外界影响，坏处是任何改动要全部分发。
			对于安装应用的系统来讲，当然是共享的越多越好，既省内存又省硬盘。

	- .so:shared object

			共享库，用过Windows的同学应该都或多或少碰到过找不到DLL或DLL错误之类的问题，其中最为著名的问题就是DLL Hell（某个著名的库，软件a使用1.0，新装的软件b使用1.0.1，导致软件a运行异常），DLL即Dynamic Link Library的缩写，和shared object表示同样的事物，只是名字不同而已。
			运行时按需加载，不论是系统提供的共享库还是自带的共享库，最大化利用软件分治的原理，修Bug也是更新所在so文件，不需全部更新。


- Android中的so

		so是与平台相关的二进制机器码，与ABI（Application Binary Interface）相对应，
		一个ABI表示相应的CPU的指令集与内存页管理，也对应于不同的C运行环境，所以so是有不同的系统版本的。
		
		随着Android系统的快速发展，搭载Android的硬件平台也早已多样化了（对比WinTel联盟，直到2012年才新发展了Windows RT来适配ARM平台，2015年的Win10才进入 Raspberry Pi 2这类基于ARM的新型设备中），
		现在已经运行在7个ABI：armeabi，armeabi-v7a (armeabi-v7a-hard)，arm64-v8a，x86，x86_64，mips 和 mips64。
		
- Android为什么选择使用so

		Android 是基于 Linux Kernl，同样也继承了所有so的相关设计。
		除了系统原因，还有一下几点：
			1. so机制能够让开发者最大化利用己有的C 和 C++代码
			2.so比java执行速度快
			3.内存分配不受Dalivik/ART的单个应用限制，减少OOM
			
- 如何使用
	- Android Studio中。将得到的ABI放到jniLibs/API
	
			├── AndroidManifest.xml
			└── jniLibs
			    ├── armeabi
			    │   └── libsnappydb-native.so
			    ├── armeabi-v7a
			    │   └── libsnappydb-native.so
			    ├── mips
			    │   └── libsnappydb-native.so
			    └── x86
				└── libsnappydb-native.so


	- 或者使用jniLibs.srcDir属性指定：
			
			android {
			    sourceSets {
				main {
				    jni.srcDirs = [] //disable automatic ndk-build call
				    jniLibs.srcDir 'main/libs'
				}
			    }
			}

	- eclipse中直接放到libs/ABI目录。
	
	- 在aar文件中，so处于jni/ABI目录中，对于库开发者和应用开发者都不必关注，全部自动处理。
	
	- 在生成的APK中，所有so文件对应于lib/ABI中。
	
	- 当APK安装到Android系统中时，so文件位置:
	
			Android<5.0，/data/data/PACKAGE_NAME/lib
			Android>=5.0，/data/app/PACKAGE_NAME/lib/CPU_ARCH/和/data/data/PACKAGE_NAME/lib

- 可能出现的问题

    UnsatisfiedLinkError

    dlopen：falied
