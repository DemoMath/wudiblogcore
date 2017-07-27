---
layout: post
title:  "实现WebView中网页与App的Activity跳转"
date:   2016-08-13 
categories: [Android,WebView]
tags: [Android]
---
介绍三种实现方式:

	1. 通过JS来实现
	2. 通过scheme来实现(不支持webview的重定向)
	3. 通过webview的重定向（setWebViewClient（WebView view,String url）{})方法中对url判断，隐式意图打开Activity


1.通过JS

只需要三个步骤:

	* WebView开启JavaScript脚本执行。
	* WebView设置供JavaScript调用的交互接口。
	* 客户端和网页端编写调用对方的代码。
写了两遍都没保存上，恼火...什么时候想起来什么时候补吧

2.通过scheme（Deep Linking深度链接，可以自行查资料https://www.linkedme.cc/）

在html中，设置链接    

	<a href="myscheme://host：8080/path/?action=com.example.wudi.webviewdemo.baginfo&id=123"> Take a QR code </a>
	<!--bagin
	     下面介绍各个部分:
	          "myscheme":scheme
	          "host":host主机
	          "8080":post端口号
	          "path":path路径
	          "action"=com.example.wudi.webviewdemo.baginfo&id=123:params参数
	  -->             
	    
然后我们在清单文件中需要给要打开的activity设置intent-filter的scheme(scheme:计划；组合；体制；诡计)，其中scheme必须和链接中scheme保持一致,host可写可不写，自行忽略

	<activity android:name=".DemoActivity">
	   <!— URI Scheme方式 -—>
	    <intent-filter>
	        <data android:scheme="myscheme" />
	        <data android:host="host"/>
	        <action android:name="android.intent.action.VIEW" />
	        <category android:name="android.intent.category.DEFAULT" />
	        <category android:name="android.intent.category.BROWSABLE" />
	        <!--<data android:host=""
	            android:mimeType=""
	            android:path=""
	            android:pathPattern=""
	            android:pathPrefix=""
	            android:port=""
	            android:scheme=""
	            android:ssp=""
	            android:sspPattern=""
	            android:sspPrefix=""/>-->
	    </intent-filter>
	</activity>

这样我们在要打开的activity中获取参数:

		Intent intent = getIntent();
		Log.e("TAG", "scheme:" + intent.getScheme());
		Uri uri = intent.getData();
		if (uri != null) {
		    Log.e("TAG", "id:" + uri.getQueryParamter("id"));
		    Log.e("TAG", "action:" + uri.getQueryParamter("name"));
		}


需要注意的是：这种方式不支持webview的重定向操作，如果你对webview设置了重定向，那么就会返回:ERR_UNKNOWN_URL_SCHEME。

3.通过隐式意图

如果我们不想写js，又想要对webview进行重定向（大多数android开发都需要，因为要适配),那么就可以通过隐式意图
基本上和第二种方式相同：
   在html中:
         
	<a href="android://?action=com.example.wudi.webviewdemo.baginfo&id=123"> Take a QR code </a>
     
   与第二种方式不同的是：当中的链接是自己自定义的，如果你够聪明，就能拼写出优秀的uri，就像上边我拼的，因为项目中，需要和ios同步开发（ios用的LMBIOS），所以我定义了android来作为我的区分。
   下边我们需要在清单文件中给activity设置action和category

	<activity android:name=".DemoActivity">
	  <!-- <!— URI Scheme方式 -—>
	    <intent-filter>
	        <action android:name="com.example.wudi.webviewdemo.baginfo"/>
	        <category android:name="android.intent.category.DEFAULT"/>
	    </intent-filter>
	</activity>

   可以发现我设置的action和上边链接中action参数一致，下边就需要在webview重定向时，进行判断了

	webview.setWebViewClient(new WebViewClient() {
	    @Override
	    public boolean shouldOverrideUrlLoading(WebView view, String url) {
	        // TODO Auto-generated method stub
	        //返回值是true的时候控制去WebView打开，为false调用系统浏览器或第三方浏览器
	        if (url.startsWith("android")) {
	            Uri uri = Uri.parse(url);
	            String host = uri.getHost();
	            String action = uri.getQueryParameter("action");
	            String id = uri.getQueryParameter("id");
	            Intent intent = new Intent();
	            intent.setAction(action);
	            intent.putExtra("id",id);
	            startActivity(intent);
	        } else {
	            view.loadUrl(url);
	        }
	        return true;
	    }
	});

   可以，看到，我的链接拼接方式可以直接把url转成uri，并获取参数，打开activity
