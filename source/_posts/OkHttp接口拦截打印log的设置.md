---
layout: post
title:  "OkHttp接口拦截打印log的设置"
date:   2016-06-30 
categories: [Android,网络]
tags: [Android]
---

通常调试网络接口时都会将网络请求和响应相关数据通过日志的形式打印出来。OkHttp也提供了一个网络拦截器okhttp-logging-interceptor，通过它能拦截okhttp网络请求和响应所有相关信息（请求行、请求头、请求体、响应行、响应行、响应头、响应体）。

<hr>

- 使用okhttp网络日志拦截器：

		compile 'com.squareup.okhttp3:logging-interceptor:3.5.0'
	
- 定义拦截器中的网络日志工具

		public class HttpLogger implements HttpLoggingInterceptor.Logger {
			@Override
			public void log(String message) {
			    Log.d("HttpLogInfo", message);
			}
		 }
	 
- 初始化OkHttpClient，并添加网络日志拦截器

        ```java
		/**
		* 初始化okhttpclient.
		*
		* @return okhttpClient
		*/
		private OkHttpClient okhttpclient() {
		    if (mOkHttpClient == null) {
			HttpLoggingInterceptor logInterceptor = new HttpLoggingInterceptor(new HttpLogger());
			logInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
			mOkHttpClient = new OkHttpClient.Builder()
			    .connectTimeout(15, TimeUnit.SECONDS)
			    .addNetworkInterceptor(logInterceptor)
			    .build();
		    }
		    return mOkHttpClient;
		}
		```
	
- 打印出来的日志
	- 拦截的网络请求日志信息1

	![](http://ot0nm27pk.bkt.clouddn.com/logger_01.png)

	- 拦截的网络请求日志信息2

	![](http://ot0nm27pk.bkt.clouddn.com/logger_02.png)

	- 拦截的网络请求日志信息3

	![](http://ot0nm27pk.bkt.clouddn.com/logger_03.png)
	
	
<hr>

- 注意:
在给OkhttpClient添加网络请求拦截器的时候应该调用方法addNetworkInterceptor，而不是addInterceptor。因为有时候可能会通过cookieJar在header里面去添加一些持久化的cookie或者session信息。这样就在请求头里面就不会打印出这些信息。

看一下OkHttpClient调用拦截器的源码：


	Response getResponseWithInterceptorChain() throws IOException {
	    // Build a full stack of interceptors.
	    List<Interceptor> interceptors = new ArrayList<>();
	    interceptors.addAll(client.interceptors());
	    interceptors.add(retryAndFollowUpInterceptor);
	    interceptors.add(new BridgeInterceptor(client.cookieJar()));
	    interceptors.add(new CacheInterceptor(client.internalCache()));
	    interceptors.add(new ConnectInterceptor(client));
	    if (!forWebSocket) {
	      interceptors.addAll(client.networkInterceptors());
	    }
	    interceptors.add(new CallServerInterceptor(forWebSocket));
	    Interceptor.Chain chain = new RealInterceptorChain(
		interceptors, null, null, null, 0, originalRequest);
	    return chain.proceed(originalRequest);
	  }
在okhttp执行网络请求时，会先构造拦截链，此时是将所有的拦截器都放入一个ArrayList中，看源码就知道添加拦截器的顺序是：

	client.interceptors()，
	BridgeInterceptor，
	CacheInterceptor，
	ConnectInterceptor，
	networkInterceptors，
	CallServerInterceptor。
	在通过拦截链执行拦截逻辑是按先后顺序递归调用的。如果是我们调用addInterceptor方法来添加HttpLoggingInterceptor拦截器，那么网络日志拦截器就会被添加到client.networkInterceptors()里面，根据添加到ArrayList中的顺序，执行拦截时会先执行HttpLoggingInterceptor，并打印出日志。然后才会执行CookieJar包装的拦截器BridgeInterceptor。这就导致我们添加header中的cookie等信息不会打印出来。
	
<hr>

现在我们打印出了完整的日志，但是格式看起来很不舒服，下面我们来对打印出来的数据格式化。

我采用的是开源日志库looger来打印

加入依赖：

 	compile 'com.orhanobut:logger:1.15'
	
使用looger库的时候建议先封装一层，作为一个工具类。

	public class LogUtil {
	    /**
	     * 初始化log工具，在app入口处调用
	     *
	     * @param isLogEnable 是否打印log
	     */
	    public static void init(boolean isLogEnable) {
		Logger.init("LogHttpInfo")
		        .hideThreadInfo()
		        .logLevel(isLogEnable ? LogLevel.FULL : LogLevel.NONE)
		        .methodOffset(2);
	    }

	    public static void d(String message) {
		Logger.d(message);
	    }

	    public static void i(String message) {
		Logger.i(message);
	    }

	    public static void w(String message, Throwable e) {
		String info = e != null ? e.toString() : "null";
		Logger.w(message + "：" + info);
	    }

	    public static void e(String message, Throwable e) {
		Logger.e(e, message);
	    }

	    public static void json(String json) {
		Logger.json(json);
	    }
	}
	
我们还需要在Application调用初始化方法

	// 初始化Looger工具
	LogUtil.init(BuildConfig.LOG_DEBUG);
	
这时候我们继续写HttpLogger类

	private class HttpLogger implements HttpLoggingInterceptor.Logger {
	    private StringBuilder mMessage = new StringBuilder();

	    @Override
	    public void log(String message) {
		// 请求或者响应开始
		if (message.startsWith("--> POST")) {
		    mMessage.setLength(0);
		}
		// 以{}或者[]形式的说明是响应结果的json数据，需要进行格式化
		if ((message.startsWith("{") && message.endsWith("}"))
		    || (message.startsWith("[") && message.endsWith("]"))) {
		    message = formatJson(decodeUnicode(message));
		}
		mMessage.append(message.concat("\n"));
		// 响应结束，打印整条日志
		if (message.startsWith("<-- END HTTP")) {
		    LogUtil.d(mMessage.toString());
		}
	    }
	    /**
		 * 格式化json字符串
		 *
		 * @param jsonStr 需要格式化的json串
		 * @return 格式化后的json串
		 */
		public static String formatJson(String jsonStr) {
		    if (null == jsonStr || "".equals(jsonStr)) return "";
		    StringBuilder sb = new StringBuilder();
		    char last = '\0';
		    char current = '\0';
		    int indent = 0;
		    for (int i = 0; i < jsonStr.length(); i++) {
			last = current;
			current = jsonStr.charAt(i);
			//遇到{ [换行，且下一行缩进
			switch (current) {
			    case '{':
			    case '[':
				sb.append(current);
				sb.append('\n');
				indent++;
				addIndentBlank(sb, indent);
				break;
			    //遇到} ]换行，当前行缩进
			    case '}':
			    case ']':
				sb.append('\n');
				indent--;
				addIndentBlank(sb, indent);
				sb.append(current);
				break;
			    //遇到,换行
			    case ',':
				sb.append(current);
				if (last != '\\') {
				    sb.append('\n');
				    addIndentBlank(sb, indent);
				}
				break;
			    default:
				sb.append(current);
			}
		    }
		return sb.toString();
		}

		/**
		 * 添加space
		 *
		 * @param sb
		 * @param indent
		 */
		private static void addIndentBlank(StringBuilder sb, int indent) {
		    for (int i = 0; i < indent; i++) {
			sb.append('\t');
		    }
		}
		/**
		 * http 请求数据返回 json 中中文字符为 unicode 编码转汉字转码
		 *
		 * @param theString
		 * @return 转化后的结果.
		 */
		public static String decodeUnicode(String theString) {
		    char aChar;
		    int len = theString.length();
		    StringBuffer outBuffer = new StringBuffer(len);
		    for (int x = 0; x < len; ) {
			aChar = theString.charAt(x++);
			if (aChar == '\\') {
			    aChar = theString.charAt(x++);
			    if (aChar == 'u') {
				int value = 0;
				for (int i = 0; i < 4; i++) {
				    aChar = theString.charAt(x++);
				    switch (aChar) {
				        case '0':
				        case '1':
				        case '2':
				        case '3':
				        case '4':
				        case '5':
				        case '6':
				        case '7':
				        case '8':
				        case '9':
				            value = (value << 4) + aChar - '0';
				            break;
				        case 'a':
				        case 'b':
				        case 'c':
				        case 'd':
				        case 'e':
				        case 'f':
				            value = (value << 4) + 10 + aChar - 'a';
				            break;
				        case 'A':
				        case 'B':
				        case 'C':
				        case 'D':
				        case 'E':
				        case 'F':
				            value = (value << 4) + 10 + aChar - 'A';
				            break;
				        default:
				            throw new IllegalArgumentException(
				                    "Malformed   \\uxxxx   encoding.");
				    }

				}
				outBuffer.append((char) value);
			    } else {
				if (aChar == 't')
				    aChar = '\t';
				else if (aChar == 'r')
				    aChar = '\r';
				else if (aChar == 'n')
				    aChar = '\n';
				else if (aChar == 'f')
				    aChar = '\f';
				outBuffer.append(aChar);
			    }
			} else
			    outBuffer.append(aChar);
		    }
		    return outBuffer.toString();
		}		
	}
	
<hr>

最终效果

	D/LogHttpInfo: ╔════════════════════════════════════════════════════════════════════════════════════════
	D/LogHttpInfo: ║ RealInterceptorChain.proceed  (RealInterceptorChain.java:92)
	D/LogHttpInfo: ║    HttpLoggingInterceptor.intercept  (HttpLoggingInterceptor.java:266)
	D/LogHttpInfo: ╟────────────────────────────────────────────────────────────────────────────────────────
	D/LogHttpInfo: ║ --> POST http://op.juhe.cn/onebox/movie/video http/1.1
	D/LogHttpInfo: ║ Content-Type: application/x-www-form-urlencoded
	D/LogHttpInfo: ║ Content-Length: 95
	D/LogHttpInfo: ║ Host: op.juhe.cn
	D/LogHttpInfo: ║ Connection: Keep-Alive
	D/LogHttpInfo: ║ Accept-Encoding: gzip
	D/LogHttpInfo: ║ User-Agent: okhttp/3.5.0
	D/LogHttpInfo: ║ 
	D/LogHttpInfo: ║ key=a3d3a43fcc149b6ed8268b8fa41d27b7&dtype=json&q=%E9%81%97%E8%90%BD%E7%9A%84%E4%B8%96%E7%95%8C
	D/LogHttpInfo: ║ --> END POST (95-byte body)
	D/LogHttpInfo: ║ <-- 200 OK http://op.juhe.cn/onebox/movie/video (760ms)
	D/LogHttpInfo: ║ Server: nginx
	D/LogHttpInfo: ║ Date: Mon, 16 Jan 2017 09:36:35 GMT
	D/LogHttpInfo: ║ Content-Type: application/json;charset=utf-8
	D/LogHttpInfo: ║ Transfer-Encoding: chunked
	D/LogHttpInfo: ║ Connection: keep-alive
	D/LogHttpInfo: ║ X-Powered-By: PHP/5.6.23
	D/LogHttpInfo: ║ 
	D/LogHttpInfo: ║ {
	D/LogHttpInfo: ║     "reason":"查询成功",
	D/LogHttpInfo: ║     "result":{
							...
	D/LogHttpInfo: ║     },
	D/LogHttpInfo: ║     "error_code":0
	D/LogHttpInfo: ║ }
	D/LogHttpInfo: ║ <-- END HTTP (2994-byte body)
	D/LogHttpInfo: ╚══════════════════════════════════════════════════════════════════

<hr>
以上就是所有的设置了，下面来说一下怎么更改打印的log不同等级的颜色

流程：

	1. File->Settings 或Ctrl + Alt +S
	2. 找到 Editor -> Colors &Fonts -> Android Logcat 或在上面的搜索框中输入Logcat
	3. 点中Verbose , Info, Debug等选项，然后在后面将Use Inberited attributes 去掉勾选
	4. 再将 Foreground 前的复选框选上，就可以双击后面的框框去选择颜色了
	Apply–>OK

对应颜色值：

	VERBOSE	BBBBBB
	DEBUG	0070BB
	INFO	48BB31
	WARN	BBBB23
	ERROR	FF0006
	ASSERT	8F0005
	
效果：

![](http://ot0nm27pk.bkt.clouddn.com/logger_04.png)
	
