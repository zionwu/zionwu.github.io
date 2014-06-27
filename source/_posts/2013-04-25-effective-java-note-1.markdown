---
layout: post
title: "effective java note-1"
date: 2013-04-25 23:41
comments: true
categories:  [java, reading note]
---
##前言##
对于Effective java，有人初看时可能会觉得这本书讲的都是java的一些技巧。书中确实介绍了很多更好的使用java特性的技巧。但是，本书的精华在于怎么用写API的思想来编写程序。在微薄上看到陈皓老师的一条微薄，印象深刻，大致意思是目前很多程序员仍然带着写应用的思想在写程序，而不是从写平台API提供者的角度来写。这一系列的读书笔记将突出书中的API编写思想。注：本文的读者需要对面向对象编程(Object Oriented Programming)有一定的了解。

##创建与销毁对象##

###使用静态工厂方法代替构造器###
静态工厂方法的例子如下：
	public static Boolean valueOf(boolean b)
    {
		return b ? Boolean.TRUE : Boolean.FALSE;
    }

####静态工厂方法(static factory method)的优缺点####
为什么要使用静态工厂方法(static factory method)呢？  
1. *静态工厂方法有名字。*  
写程序时，变量名，方法名都是很重要的元素。*程序的本质在于抽象*，一个好的名字就是好的抽象。构造器(constructor)的名字都必须和类名相同，当类拥有多个构造器的时候，使用者往往很难理解不同构造器的含义。而静态方法可以通过使用不同的名字代表该方法的抽象，因此使用者不会混淆。  
2. 不必每次被调用都创建心的对象。  
即通过使用静态工厂方法可以实现单例模式(singleton)，从而避免创建不必要的重复对象。上面的例子中，Boolean的valueOf方法每次返回的都是一个枚举(enum)类型，而*枚举类型就是单例的*   
3. 可以返回原类型的任何字类型对象。  
例子如下： 
    public static Type getInstance()
	{
		return new SubType();
	}  
例子中的SubType是Type的子类。使用构造器的时候，用户只能得到该类的实例，而静态方法能够返回其子类的类型。这就是设计模式中所强调的*针对接口编程*的原则，这将增加代码的灵活性，如果日后需要对Type有新的实现，那么只需要修改静态方法中的一句话就够了，即return new AnotherSubType();。书中在此点时介绍了服务者提供者框架(service provider framework), 这是一个十分重要的设计模式，此节后面会详细介绍。  
4. 创建参数化实例的时候代码能够更加简洁。 
书中的例子如下： 
	Map<String, List<String>> m = new HashMap<Stirng, List<String>>();
当初始化参数化的容器时，需要重复敲写两次参数。而使用静态方法的话： 
	public static <K, V> HashMap<K, V> newInstance()
	{
		return new HashMap<K, V>();
	}
    Map<String, List<String>> m = HashMap.newInstance()  
因为编译器的类型推导(type inference)，上面的静态范型方法能够推导出 K=String, V=List\<Stirng\>的事实。  
静态方法的缺点有：   
1. 如果类不含有非私有的构造器，就不能被继承。  
2. 不能和其他静态方法区分开。  

####服务者提供者框架(service provider framework)####
服务提供者框架有三个基本的组件：*服务接口(service API)*，*提供者注册API(provider registration API)*， *服务访问API(service access API)*。还有一个可选的组件是*服务提供者接口(service provider interface)*。首先举个通俗的例子来帮助大家理解这个框架的三个基本组件。大学里某门课程，往往会提供数个教学班，在不同时间由不同老师来上。虽然老师不一样，但是课程的教学大纲必须是一致的，而且期末考试也是一样的。课程的大纲，期末试卷，都是由课程组规定的，是该门课提供给学生的统一的服务，也就是服务接口。无论你上A老师的还是B老师的计算机组成，都是学习多周期CPU，内存模型等知识，而不可能是计算理论的知识。但是不同的老师，可以采取不同的教学模式，A老师可能比较注重理论，B老师则可能注重实践，即他们对服务接口有不一样的实现。A老师和B老师要开课，需要将他们的课程的地点和时间等信息填表向课程组申请开课。这个申请的过程即提供者注册API，即服务者将服务获取的渠道注册在某个地方。学生选课的过程则是服务访问API，即获取服务的方式。笔者接下来将详细介绍每一个组件。  
1. 服务接口   
服务接口定义了系统所能够提供的服务，一般是有平台工程师编写，所谓平台就是上面例子中的课程组。书中例子如下： 
	public interface Service
    {
        ...// Service-specific methods, 即这个接口定义的方法，也就是Service所能提供的服务
	}  
对于一个开放平台，这个接口给该平台的应用工程师使用，应用工程师则是例子中的老师，是服务的真正提供者。  
2. 提供者注册API与服务访问API   
这两个组件一般是在一个工厂类中实现的，例子如下：  
	public class Services
	{
		private Services(){}

		private static final Map<String, Service> services = new ConcurrentHashMap<String, Service>();
	    public static final String DEFAULT_SERVICE_NAME = "def";

		//提供者注册API(provider registration API)
		public static void registerDefaultService(Service s)
		{
			registerService(DEFAULT_SERVICE_NAME, s);
		}

		public static void registerService(String name, Service s)
		{
			services.put(name, p);
		}

		//服务访问API(service access API)
		public static Service newInstance()
        {
			return services.newInstance(DEFAULT_SERVICE_NAME);
		}

		public static Service newInstance(String name)
		{
			Service s = services.get(name);
			if (s == null)
				throws IllegalArgumentException( "No service registered with name" + name );
			return s;
		}
	}  
该类是模仿书中的例子编写的，书中的例子包含了可选组件服务提供者接口，这里为了简单而只用了三个基本组件。首先注意该类的构造器是私有的，即该类*不可实例化*。对于提供者注册API和服务访问API，都重载了一个方法，即一个默认的服务。比如当学生没有主动选课的时候系统将自动为他选择一个默认的老师。而注册和获取实际都是对一个Map进行操作，注意该Map是一个ConcurrentHashMap，这是为了*线程安全*，防止多个线程同时调用类中方法时出现数据不一致或者冲突的情况。  
3. 服务提供者接口  
服务提供者接口实际上是将服务提供和服务注册分离开。例子如下：  
	public interface Provider
	{
		Service newService();
	}
然后将第二点的例子中的Service类替换成Provider, 如下：  
	public class Services
	{
		private Services(){}

		private static final Map<String, Provider> providers = new ConcurrentHashMap<String, Provider>();
	    public static final String DEFAULT_SERVICE_NAME = "def";

		//提供者注册API(provider registration API)
		public static void registerDefaultProvider(Provider s)
		{
			registerService(DEFAULT_SERVICE_NAME, s);
		}

		public static void registerProvider(String name, Provider s)
		{
			providers.put(name, p);
		}

		//服务访问API(service access API)
		public static Service newInstance()
        {
			return providers.newInstance(DEFAULT_SERVICE_NAME);
		}

		public static Service newInstance(String name)
		{
			Service s = providers.get(name);
			if (s == null)
				throws IllegalArgumentException( "No provider registered with name" + name );
			return s.newService();
		}
	}
这样注册的就是Provider而不是Service了，那么同一个提供者，日后可以修改自己的服务，并在自己实现的Provider子类中的newService()方法中提供自己新的服务。这样就大大增加了灵活性。  


总而言之，服务提供者框架是一个十分重要而且常用的框架，使用它来创建API，可以大大的*增加API的灵活性，达到高内聚性和低耦合度*。


