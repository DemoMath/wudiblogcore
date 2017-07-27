---
layout: post
title:  "JavaSpcript常用互动方法"
date:   2017-01-16 
categories: [JS,基础]
tags: [JS]
---
#### 1.输出内容

document.write() 可用于直接向HEML输出流写内容。简单的说就是直接在网页中输出内容

 1.1 第一种：输出内容用“”括起，直接输出“”号内的内容

	<script type="text/javascript">
	  document.write("I love JavaScript！"); //内容用""括起来，""里的内容直接输出。
	</script>

 1.2 第二种：通过变量，输出内容

	<script type="text/javascript">
	  var mystr="hello world!";
	  document.write(mystr);  //直接写变量名，输出变量存储的内容。
	</script>
 
 1.3 第三种：输出多项内容，内容之间用+连接

	<script type="text/javascript">
	  var mystr="hello";
	  document.write(mystr+"I love JavaScript"); //多项内容之间用+号连接
	</script>

 1.4 第四中：输出HTML标签，并起作用，标签使用“”括起来

	<script type="text/javascript">
	  var mystr="hello";
	document.write(mystr+"<br>");//输出hello后，输出一个换行符
	  document.write("JavaScript");
	</script>

 注意：
	
	空格使用&nbsp;
#### 2.alert消息对话框

我们在访问网站的时候，有时候会突然弹出一个小窗口，上面写着一段提示文字。

语法：

	alert(字符串或变量);  

demo：

	<!DOCTYPE HTML>
	<html>
	<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title>alert</title>
	  <script type="text/javascript">
	  function rec(){
	  var mychar="I love JavaScript";
	  alert(mychar);
	  }
	  </script>
	</head>
	<body>
	  <input name="button" type="button" onClick="rec()" value="点击我，弹出对话框" />
	</body>
	</html>
#### 3.confirm消息对话框

confirm消息对话框通常用于允许用户做选择的动作，如：“你对吗”等，弹出对话框（包括一个确定按钮和一个取消按钮）

语法：

	confirm(str);

参数说明：

	str：在消息对话框中要显示的文本
	返回值: Boolean值

返回值：

	当用户点击"确定"按钮时，返回true
	当用户点击"取消"按钮时，返回false

demo：

	<script type="text/javascript">
	    var mymessage=confirm("你喜欢JavaScript吗?");
	    if(mymessage==true)
	    {   document.write("很好,加油!");   }
	    else
	    {  document.write("JS功能强大，要学习噢!");   }
	</script>

注意：

	消息对话框是排斥其他操作的，即用户在点击对话框按钮前，不能进行任何其它的操作

#### 4.prompt消息对话框

prompt弹出消息对话框，通常用于询问一些需要与用户交互的信息，弹出消息对话框（包含一个确定按钮，取消按钮与一个文本输入框）

语法：

	prompt(str1, str2);

参数说明：

	str1: 要显示在消息对话框中的文本，不可修改
	str2：文本框中的内容，可以修改

返回值：

	1. 点击确定按钮，文本框中的内容将作为函数返回值
	2. 点击取消按钮，将返回null

demo：

	var myname=prompt("请输入你的姓名:");
	if(myname!=null)
	  {   alert("你好"+myname); }
	else
	  {  alert("你好 my friend.");  }

#### 5.打开新窗口window.open
open()方法可以查找一个已经存在或者新建的浏览器窗口

语法：

	window.open([URL], [窗口名称], [参数字符串])

参数说明：

	URL：可选参数，在窗口中要显示网页的网址或路径。如果省略这个参数，或者它的值是空字符串，那么窗口就不显示任何文档。
	窗口名称：可选参数，被打开窗口的名称。
	    1.该名称由字母、数字和下划线字符组成。
	    2."_top"、"_blank"、"_self"具有特殊意义的名称。
	      _blank：在新窗口显示目标网页
	      _self：在当前窗口显示目标网页
	      _top：框架网页中在上部窗口中显示目标网页
	    3.相同 name 的窗口只能创建一个，要想创建多个窗口则 name 不能相同。
	    4.name 不能包含有空格。
参数字符串：
	
	可选参数，设置窗口参数，各参数用逗号隔开。

#### 6.关闭窗口window.close
close()关闭窗口

用法：
	
	window.close();   //关闭本窗口
	或
	<窗口对象>.close();   //关闭指定的窗口

demo：

	<script type="text/javascript">
	   var mywin=window.open('http://www.imooc.com'); //将新打的窗口对象，存储在变量mywin中
	   mywin.close();
	</script>







