---
title:  "Android6.0新的权限机制"
date: 2016-05-02 14:47:01
categories: [Android,适配]
tags: [Android]
---
接触到了android6.0权限的几个坑，这里记录一下
参考自：[http://blog.csdn.net/lmj623565791/article/details/50709663](http://blog.csdn.net/lmj623565791/article/details/50709663) 

对于6.0以下的权限及在安装的时候，根据权限声明产生一个权限列表，用户只有在同意之后才能完成app的安装，造成了我们想要使用某个app，就要默默忍受其一些不必要的权限（比如是个app都要访问通讯录、短信等）。

而在6.0以后，我们可以直接安装，当app需要我们授予不恰当的权限的时候，我们可以予以拒绝。当然你也可以在设置界面对每个app的权限进行查看，以及对单个权限进行授权或者解除授权。



新的权限机制更好的保护了用户的隐私，Google将权限分为两类：

- 一类是Normal Permissions，这类权限一般不涉及用户隐私，是不需要用户进行授权的，比如手机震动、访问网络等；
- 另一类是Dangerous Permission，一般是涉及到用户隐私的，需要用户进行授权，比如读取sdcard、访问通讯录等。


Normal Permissions如下：

	ACCESS_LOCATION_EXTRA_COMMANDS
	ACCESS_NETWORK_STATE
	ACCESS_NOTIFICATION_POLICY
	ACCESS_WIFI_STATE
	BLUETOOTH
	BLUETOOTH_ADMIN
	BROADCAST_STICKY
	CHANGE_NETWORK_STATE
	CHANGE_WIFI_MULTICAST_STATE
	CHANGE_WIFI_STATE
	DISABLE_KEYGUARD
	EXPAND_STATUS_BAR
	GET_PACKAGE_SIZE
	INSTALL_SHORTCUT
	INTERNET
	KILL_BACKGROUND_PROCESSES
	MODIFY_AUDIO_SETTINGS
	NFC
	READ_SYNC_SETTINGS
	READ_SYNC_STATS
	RECEIVE_BOOT_COMPLETED
	REORDER_TASKS
	REQUEST_INSTALL_PACKAGES
	SET_ALARM
	SET_TIME_ZONE
	SET_WALLPAPER
	SET_WALLPAPER_HINTS
	TRANSMIT_IR
	UNINSTALL_SHORTCUT
	USE_FINGERPRINT
	VIBRATE
	WAKE_LOCK
	WRITE_SYNC_SETTINGS
	
Dangerous Permissions:

	group:android.permission-group.CONTACTS
	permission:android.permission.WRITE_CONTACTS
	permission:android.permission.GET_ACCOUNTS
	permission:android.permission.READ_CONTACTS

	group:android.permission-group.PHONE
	permission:android.permission.READ_CALL_LOG
	permission:android.permission.READ_PHONE_STATE
	permission:android.permission.CALL_PHONE
	permission:android.permission.WRITE_CALL_LOG
	permission:android.permission.USE_SIP
	permission:android.permission.PROCESS_OUTGOING_CALLS
	permission:com.android.voicemail.permission.ADD_VOICEMAIL

	group:android.permission-group.CALENDAR
	permission:android.permission.READ_CALENDAR
	permission:android.permission.WRITE_CALENDAR

	group:android.permission-group.CAMERA
	permission:android.permission.CAMERA

	group:android.permission-group.SENSORS
	permission:android.permission.BODY_SENSORS

	group:android.permission-group.LOCATION
	permission:android.permission.ACCESS_FINE_LOCATION
	permission:android.permission.ACCESS_COARSE_LOCATION

	group:android.permission-group.STORAGE
	permission:android.permission.READ_EXTERNAL_STORAGE
	permission:android.permission.WRITE_EXTERNAL_STORAGE

	group:android.permission-group.MICROPHONE
	permission:android.permission.RECORD_AUDIO

	group:android.permission-group.SMS
	permission:android.permission.READ_SMS
	permission:android.permission.RECEIVE_WAP_PUSH
	permission:android.permission.RECEIVE_MMS
	permission:android.permission.RECEIVE_SMS
	permission:android.permission.SEND_SMS
	permission:android.permission.READ_CELL_BROADCASTS
	
可以通过adb shell pm list permissions -d -g进行查看。

其中上面的dangerous permissions，危险权限都是一组一组的。如果app运行在android 6.x的机器上，对于授权机制是这样的。如果你申请某个危险的权限，假设你的app早已被用户授权了同一组的某个危险权限，那么系统会立即授权，而不需要用户去点击授权。比如你的app对READ_CONTACTS已经授权了，当你的app申请WRITE_CONTACTS时，系统会直接授权通过。此外，对于申请时弹出的dialog上面的文本说明也是对整个权限组的说明，而不是单个权限（注意这个dialog是不能进行定制的）。

当然也不要对权限组过多的依赖，对每个危险权限都要进行正常流程的申请，因为在后期的版本中这个权限组可能会产生变化。