---
layout: post
title: "Effective Java Note-2"
date: 2013-05-02 00:18
comments: true
categories: [java, reading note] 
---
##创建与销毁对象##

###遇到多个构造器参数时考虑用构建器###
当构造器有多个可选的参数的时候，程序员往往使用重叠构造器模式(telescoping constructor), 即：  
	public class NutritionFacts
	{
		private final int servingSize; //required
		private fianl int serings; required
		private final int calories; //optional
		private final int fat;      //optional
		private final int sodium;   //optional

		public NutritionFacts(int servingSize, int servings)
		{
			this(servingSize, servings, 0);
		}

		public NutritionFacts(int servingSize, int servings, int calories)
		{
			this(servingSize, servings, calories, 0);
		}

		public NutritionFacts(int servingSize, int servings, int calories, int fat)
		{
			this(servingSize, servings, calories, fat, 0);
		}

		public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium)
		{
			this.servingSize = servingSize;
			this.servings = servings;
			this.calories = calories;
			this.fat = fat;
			this.sodium = sodium;
		}
	}  
这种写法不易与程序员的理解与记忆，很容易将参数的顺序弄错，而编译器也不会发现这种错误，从而导致难以发现的bug。还有一种模式是Java Bean模式，即： 	
	public class NutritionFacts
	{
		private final int servingSize; //required
		private fianl int serings; required
		private final int calories; //optional
		private final int fat;      //optional
		private final int sodium;   //optional

		public NutritionFacts(){}

		//setter
		public void setServiceSize(int val){ servingSize = val; }
        public void setServings(int val){ servings = val; }
		public void setCalories(int val){ calories = val; }
		public void setFat(int val){ fat = val; }
		public void setSodium(int val){ sodium = val; }		
	}  
这种模式虽然易于理解和使用，但是有一个缺点，就是在构造过程中Java Bean可能处于不一致的状态，而且也不能将其设计成不可变(imutable)的类。后面介绍的构建器(builder)模式既能够有第一种模式的安全性，又有第二种模式的可读性。它的例子如下： 
	public class NUtritionFacts
	{
		private final int servingSize; //required
		private fianl int serings; required
		private final int calories; //optional
		private final int fat;      //optional
		private final int sodium;   //optional

		public static class Builder
		{
			private final int servingsize; //required
			private fianl int serings; //required
			private final int calories = 0; //optional
			private final int fat = 0;      //optional
			private final int sodium = 0;   //optional

			public Builder(int servingSize, int servings)
			{
				this.servingSize = servingSize;
				this.servings = servings;
			}

			public Builder calories(int val){ calories = val; return this;}
			public Builder fat(int val){ fat = val; return this;}
			public Builder sodium(int val){ sodium = val; return this;}	
	
			public NutritionFacts build()
			{
				return new NutritionFacts(this);
			}
		}

		public NutritionFacts(Builder builder)
		{
			servingSize = builder.servingSize;
			servings = builder.servings;
			calories = builder.calories;
			fat = builder.fat;
			sodium = builder.sodium;
		}
	} 
以上的例子有三点需要注意。  
1. Builder是一个内嵌类，而且是静态的。静态的内嵌类和静态的成员变量不一样。静态的内嵌类表示该类和外围类的实例没有关联。如果内嵌类不是静态的，那么必须先有外围类的实例，才能有内嵌类的实例，内嵌类的实例中会有对外围类实例的引用。  
2. 细心的读者可能发现，在NutritionFacts(Builder builder)这个构造器方法中，能够使用builder这个实例的私有域，这是因为Builder是NutritionFacts的内嵌类，Builder的私有域对于NutritionFacts内的方法是可见的。 
3. Builder中的setter方法返回的是Builder对象，在设值后都会return this,这是为了能够链式调用，如：  
	NutritionFacts cala = new NutritionFacts.Builder(240 , 0).calories(100).sodium(35).build();


###用私有构造器或者枚举强化Singleton属性###
Singleton指一个类只有一个实例。实现单例有3种方式。  
第一种方法的公有静态成员是个final域： 
	public class Elvis
	{
		public static final Elvis INSTANCE = new Elvis();
		private Elvis() {...}  
		...  
	} 
第二种方法中公有的成有是个静态工厂方法： 
	public class Elvis  
    {
		private static final Elivs INSTANCE = new Elvis();
		private Elsvis(){...}
		public static Elvis getInstance()
        {
			return INSTANCE;
        }
    }
需要注意的是，以上两种方法，如果要将其变成是可序列化(serializable)的，为了保证其Singleton的属性，需要将所有实例域声明为transient，并且提供一个readResolve方法。
第三种方法是将其定义为枚举类， 枚举类本身就是Singlenton，并且无偿提供了序列化机制。


###通过私有构造器强化不可实例化的能力###
对于一些工具类，如java.lang.Math, java.util.Coolections, 把基本类型的值或者特定接口的对象上的静态方法组织起来，实例对他们没有意义，确保它们不会被实例化的方法是将构造器定义为私有的(private)， 如：  
    public class UtilityClass 
    {
	    private UtilityClass();
        ...
    }  
这种做法的副作用是该类是不能被继承的。

