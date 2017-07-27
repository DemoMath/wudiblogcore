---
layout: post
title:  "Android热修复技术之Java代理模式"
date:   2017-03-28
categories: [Android,设计模式]
tags: [Android,Java]
---
#### 概念
**代理模式**是一种比较常用的设计模式，一般来说，当我们不想或者不能直接访问一个对象A，必须通过一个中介对象B来访问，这种方式就叫做代理模式。	

#### 优点
- 隐藏委托类的实现，调用者需要和代理类交互
- 解耦，在不改变委托类代码情况下可以做一些额外处理。

#### 区分
- **静态代理**

	代理类的字节码文件，代理类和委托类的关系 在运行之前就已经确定了。
	
	代理接口定义：
	
		public interface Subject {
		    void m1();
		    void m2();
		}
	
	代理委托者，实现了代理接口：
	
		public class RealSubject implements Subject{
		    @Override
		    public void m1() {
			SystemClock.sleep(1000);
			Log.d("DEBUG","##### call method 1");
		    }
		    @Override
		    public void m2() {
			SystemClock.sleep(2000);
			Log.d("DEBUG","##### call method 2");
		    }
		}
		
	代理类：
	
		public class ProxySubject implements Subject {
		    //引用真正的实现类
		    private RealSubject subject;
		    @Override
		    public void m1() {
			log();
			if (null == subject) {
			    subject = new RealSubject();
			}
			subject.m1();
		    }
		    @Override
		    public void m2() {
			if (null == subject) {
			    subject = new RealSubject();
			}
			subject.m2();
			log();
		    }
		    private void log() {
			Log.d("DEBUG", "### log it");
		    }
		}
		
	代理类通过引用，去调用真实对象的方法，在代理类方法中可以加入一些其他操作，比如日志操作等。
	
- **动态代理**

	当我们遇到要代理的方法比较多。比方说，我们在接口中增加一个方法，除了委托类要实现这个方法，代理类也要实现这个方法，感觉上就非常难受。
	那我们就需要一个代理类完成所有的代理功能，或者动态的生成代理类，就需要动态代理。
	
	- 实现步骤
	
		1. 创建一个实现InvocationHandler的类，实现invoke方法
		2. 通过Proxy的newProxyInstance创建一个代理
		3. 创建代理类的接口
		4. 通过代理调用方法
		
		
	- 根据步骤我们分别创建Subject和RealSubject
		Subject：
	
			package ProxyMode;
			/*
			 * 抽象接口，对应类图中的Subject
			 * 
			 */
			public interface Subject {
			    public void SujectShow();
			}
			
		RealSubject：

			package ProxyMode;
			public class RealSubject implements Subject{
			    @Override
			    public void SujectShow() {
				// TODO Auto-generated method stub
				System.out.println("我才是真正的操作，我是黑衣人！By---"+getClass());
			    }
			}
			
	- 建立InvocationHandler用来响应代理的任何调用

			package ProxyMode;

			import java.lang.reflect.InvocationHandler;
			import java.lang.reflect.Method;

			public class ProxyHandler implements InvocationHandler {

			    private Object proxied;   

			      public ProxyHandler( Object proxied )   
			      {   
					this.proxied = proxied;   
			      }   


			    @Override
			    public Object invoke(Object proxy, Method method, Object[] args)
				    throws Throwable {

				  System.out.println("准备工作之前：");

				//转调具体目标对象的方法
				  Object object=   method.invoke( proxied, args);

				  System.out.println("工作已经做完了！");
				  return object;
			    }

			}
			
	- 测试
	
			package ProxyMode;


			import java.lang.reflect.Proxy;

			public class DynamicProxy  {

			    public static void main( String args[] )   
			      {   
				RealSubject real = new RealSubject();   
				Subject proxySubject = (Subject)Proxy.newProxyInstance(Subject.class.getClassLoader(), 
				 new Class[]{Subject.class}, 
				 new ProxyHandler(real));

				proxySubject.SujectShow();;

			      }   
			}
			
	- 测试结果
	
			准备工作之前：
			我才是真正的操作，我是黑衣人！By---class ProxyMode.RealSubject
			工作已经做完了！
			





