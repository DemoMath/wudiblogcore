---
layout: post
title:  "Android中屏幕适配的方式"
date:   2016-05-27 
categories: [Android,适配]
tags: [Android]
---
android中屏幕适配的方式：

1. 图片适配 

	根据不同手机屏幕的分辨率，加载不同文件夹下的图片
	跟手机屏幕的像素密度有关系
	像素密度又是什么呢？就是假如说，手机的屏幕是5英寸的，那么分辨率就是1280*720- 的，利用勾股定理，计算出斜边的值，再除以5，
	计算出来的就是像素密度。根据这个像素密度，再确定加载哪个文件夹下的图片。	

2. dimens.xml适配

	这个主要是适配控件的宽高，就是在dimens.xml文件中设定空间的宽高，再在xml布局中引用dimens.xml问价中的值，假如想想要对固定手机屏幕分辨率进行设定，可以在res目录下，创建 values-1280 * 720 的目录，把dimens.xml文件复制过来更改里面的属性值，就可以了，这样的话，其他手机引用的就是普通的值，假如是 1280 * 720 的手机，就直接引用values-1280 * 720问价中的属性值了，但是要注意，1280 * 720的顺序不能写反了，大数一定写在前边
3. layout布局适配

	就是在res下创建layout-1280*720的目录，再这里写布局文件，这样就大大增加了应用的体积，给用户的体验不好，不到万不得已的时候不用。
4. java代码适配，这个用的比较多，他可以设定空间的控件的宽高，也可以设定之间的距离 

	怎么样做到适配呢？首先要获取手机屏幕的宽高，再通过设定比例值，选择一个适配的手机，计算出宽高的比例值，（控件的宽/手机屏幕的宽），然后再在下需要设定值的地方，拿着求出来的手机的宽*这个我们算出来的比例值，就是适配的值了。再有就是pd–>px px–>pd就是在需要适配的地方，用dp–>px的方式去设定值，因为在手机上展示都是px单位，
5. 权重适配 
	
	权重适配就是用到了weight这个属性，他是设定显示比例的，一般不能达到我们的需求，所以在使用权重适配的时候，一般会结合其他的

适配方法一起使用
