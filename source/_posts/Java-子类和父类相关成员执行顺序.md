---
title: Java 子类和父类相关成员执行顺序
date: 2018-6-2 14:20:00
categories: "Java"
tags:
     - Java
---

<meta name="referrer" content="no-referrer">






> 先说结论，再通过运行程序去验证。


## 结论

- new 一个子类的初始化顺序

  父类静态成员变量、静态代码块 --> 子类静态成员变量、静态代码块 --> 父类成员变量、构造代码块 --> 父类无参构造方法 --> 子类成员变量、   构造代码块 --> 子类无参构造方法

- 多次创建对象，静态成员变量和静态代码块都只执行一次

- 除非在子类的构造方法中显示调用父类的构造方法，否则它们默认会先去访问父类的无参构造方法，即在构造方法中的第一行默认调用 super();

## 程序运行

**打印类（用于打印结果）**

```Java
class Test {

	public Test(String str) {
		System.out.println(str);
	}
}
```

**父类**

```Java
class Parent {

	private static Test t10 = new Test("父类静态成员变量 0");
	private Test t11 = new Test("父类成员变量  0");

	{
		Test t14 = new Test("父类构造代码块 0");
	}

	static {
		Test t12 = new Test("父类静态代码块 0");
	}

	private Test t16 = new Test("父类成员变量 1");

	{
		Test t15 = new Test("父类构造代码块 1");
	}

	private static Test t13 = new Test("父类静态成员变量 1");

	public Parent() {
		System.out.println("父类无参构造方法");
	}

	public Parent(int i) {
		System.out.println("父类有参构造方法 0");
	}

	public void f() {
		System.out.println("父类成员方法");
	}

	public static void g() {
		System.out.println("父类静态成员方法");
	}
}
```

**子类**

```Java
class Son extends Parent {

	private static Test t0 = new Test("子类静态成员变量 0");
	private Test t1 = new Test("子类成员变量 0");

	{
		Test t4 = new Test("子类构造代码块 0");
	}

	static {
		Test t2 = new Test("子类静态代码块 0");
	}

	private Test t6 = new Test("子类成员变量 1");

	{
		Test t5 = new Test("子类构造代码块 1");
	}

	private static Test t3 = new Test("子类静态成员变量 1");

	public Son() {
		// super(); 默认调用
		System.out.println("子类无参构造方法");
	}

	public Son(int i) {
		// super(); 默认调用
		System.out.println("子类有参构造方法 0");
	}

	public Son(int i, int j) {
		// super(); 不会调用了，因为显式调用了 super(1)
		super(1);
		System.out.println("子类有参构造方法 1");
	}

	public void ff() {
		System.out.println("子类成员方法");
	}

	public static void gg() {
		System.out.println("子类静态成员方法");
	}
}
```

**调用类**

```Java
public class Solution {

	public static void main(String[] args) {
		new Son();
		System.out.println("------------------------------------------------");
		new Son();
		System.out.println("------------------------------------------------");
		new Son(1);
		System.out.println("------------------------------------------------");
		new Son(1, 2);
	}
}
```


执行了调用类后，打印结果如下：

```Java
父类静态成员变量 0
父类静态代码块 0
父类静态成员变量 1
子类静态成员变量 0
子类静态代码块 0
子类静态成员变量 1
父类成员变量  0
父类构造代码块 0
父类成员变量 1
父类构造代码块 1
父类无参构造方法
子类成员变量 0
子类构造代码块 0
子类成员变量 1
子类构造代码块 1
子类无参构造方法
------------------------------------------------
父类成员变量  0
父类构造代码块 0
父类成员变量 1
父类构造代码块 1
父类无参构造方法
子类成员变量 0
子类构造代码块 0
子类成员变量 1
子类构造代码块 1
子类无参构造方法
------------------------------------------------
父类成员变量  0
父类构造代码块 0
父类成员变量 1
父类构造代码块 1
父类无参构造方法
子类成员变量 0
子类构造代码块 0
子类成员变量 1
子类构造代码块 1
子类有参构造方法 0
------------------------------------------------
父类成员变量  0
父类构造代码块 0
父类成员变量 1
父类构造代码块 1
父类有参构造方法 0
子类成员变量 0
子类构造代码块 0
子类成员变量 1
子类构造代码块 1
子类有参构造方法 1
```

## 扩展说明

[引自此处](https://blog.csdn.net/u010687392/article/details/42388585)

- 子类中所有的构造方法默认都会先去调用父类中的无参构造方法。因为子类继承了父类的数据，可能还会使用到父类的数据，因此，子类初始化前需先完成父类数据的初始化。父类的初始化是通过调用方法区中的构造方法进行的，不会创建父类对象，对象需要通过 new 关键字创建。

- new 关键字有两个作用。①、分配内存，创建对象；②、调用构造方法，完成对象的初始化工作。完成这两步后，才算创建了一个完整的 Java 对象。
所以 new 子类的时候，调用父类的构造方法不是创建了一个父类对象，而是只对它的数据进行初始化，那么父类这些数据存储在哪里呢？通俗地说，子类对象内存区域中会划一部分区域给父类的数据做存储，即子类对象内存中封装了父类的初始化数据，创建子类对象时，父类的数据就是子类的对象的一部分，不存在独立的父类的对象，所有的东西在一起才是一个完整的子类的对象。