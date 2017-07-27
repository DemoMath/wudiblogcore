---
title:  "Android中的多进程的模式"
date: 2017-07-26 14:47:01
categories: [Android,IPC]
tags: [Android]
---

![](http://ot0nm27pk.bkt.clouddn.com/android_ipc_01.jpg) 

#### 开启多进程模式
如果你想在一个应用中使用多个进程,通过清单文件给四大组件添加android:process属性,就可以很方便的开启多进程.
还有一种非常规的创建方式,通过JNI在native层去fork一个新的进程.这种我们暂时只是了解一下.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.wudi.demo">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".FristActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity android:name=".SecondActivity"
            android:process=":second">
        </activity>
        <activity android:name=".ThirdActivity"
            android:process="com.third.demo">
        </activity>
    </application>

</manifest>
```

当我们依次打开FristActivity, SecondActivity, ThirdActivity.此时应该打开了三个进程.

那么命令行中进行测试
`adb shell ps`可以把系统所有进程展示出来, 你可以加上过滤信息`| grep xxx xxx`替换你需要过滤出来信息,这里我们使用`adb shell ps | grep com.wudi`,结果为：

```shell
u0_a512   13555 533   1715032 47068 SyS_epoll_ 0000000000 S com.wudi.demo
u0_a512   13585 533   1715032 47208 SyS_epoll_ 0000000000 S com.wudi.demo:second
u0_a512   13625 533   1894268 61276 SyS_epoll_ 0000000000 S com.wudi.third
```
这里需要注意的是：

- 当以`:`开头的进程,属于当前应用的私有进程,其他应用的组件不可以和它跑在同一个进程
- 当不以`:`开头,那么进程属于全局进程,其他应用通过ShareUID方法可以和它跑在同一个进程


Android系统会为每一个应用分配唯一的UID. 相同UID的应用才能共享数据. 但是两个应用通过ShareUID跑在同一个进程是有要求的. 除了具有相同的ShareUID并且还要签名相同才可以. 这时如果不在同一进程他们之间可以共享data目录,组件信息等. 如果还在同一进程, 那么他们还能共享内存数据.


#### 进程模式的运行机制

两个进程间,每个单独的进程又会分配一个独立的虚拟机, 所以每个虚拟机在内存分配上有不同的地址空间.对于不同虚拟机访问同一个对象就会产生多份副本. 副本之间互相独立不干扰彼此.

我们需要注意的问题

- 静态成员和单例模式完全失效
- 线程同步机制完全失效
- SharedPreferences的可靠性下降
- Application会多次创建



1.如果在FristActivity中对静态变量进行修改, 在SecondActivity取出这个静态发现是FristActivity没修改之前的.
2.因为不是一块内存,所以不管是锁对象还是锁全局都无法保证线程同步,因为不是同一个对象.
3.因为Sp不支持两个进程同时读写,因为底层是通过读写XML文件实现的,并发可能会触发异常.
4.运行在多个进程中,那么就会创建多个虚拟机,每个虚拟机都有一个对应Application并需要启动加载这个文件.

一个应用的多进程:**它就相当于两个不同的应用采用了ShareUID的模式. 每个进程都会拥有独立的虚拟机, Application以及内存空间**
