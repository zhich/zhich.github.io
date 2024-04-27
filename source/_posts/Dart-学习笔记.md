---
title: Dart 学习笔记
date: 2020-12-30 15:56:00
categories: "Dart"
tags:
     - Dart
---

<meta name="referrer" content="no-referrer">





> 本笔记整理自大地老师的 Dart 笔记，同时加入了自己写的一些测试例子。


### 环境搭建

要在我们本地开发 Dart 程序的话首先需要安装 Dart SDK

官方文档：https://dart.dev/get-dart
```dart
windows(推荐):

  http://www.gekorm.com/dart-windows/

mac：

  如果 mac 电脑没有安装 brew 这个工具首先第一步需要安装它： https://brew.sh/

  brew tap dart-lang/dart
    
  brew install dart
```

### 开发工具
```dart
Dart 的开发工具有很多： IntelliJ IDEA  、 WebStorm、 Atom、Vscode 等

这里我们主要给大家讲解的是如果在 vscode 中配置 Dart。
    
1、找到 vscode 插件安装 Dart

2、找到 vscode 插件安装 code runner, Code Runner 可以运行我们的文件
```

### 变量
Dart 中定义变量可以使用具体的类型（int、double、String、bool 等）申明 ，也可以通过 var 关键字来申明变量。

### 常量
> final 和 const 修饰符都可以表示常量。

- const 值不变，一开始就得赋值。
- final 可以开始不赋值，只能赋一次。而 final 不仅有 const 的编译时常量的特性，最重要的它是运行时常量，并且 final 是惰性初始化，即在运行时第一次使用前才初始化。
- 注意：永远不改变的量，请使用 final 或 const 修饰它，而不是使用 var 或其他变量类型。
```dart
main() {
  const PI = 3.14159;
  // PI = 3.14; // 报错
  print(PI);

  // const x = new DateTime.now(); // 报错

  final a = 1;
  // a = 2; // 报错
  print(a);

  final time = new DateTime.now();
  print(time);
}
```

### 常用数据类型
Dart 中支持以下常用数据类型：
```dart
Numbers（数值）
    int
    double
Strings（字符串）
    String
Booleans(布尔)
    bool
List（数组）
    在 Dart 中，数组是列表对象，所以大多数人只是称它们为列表
Maps（字典）
    通常来说，Map 是一个键值对相关的对象。键和值可以是任何类型的对象。每个键只出现一次，而一个值则可以出现多次
```

基本使用：
```dart
main() {
  /**
   * 1、字符串类型（String）
   */
  var str1 = "Hello";
  String str2 = "World";
  print("$str1 $str2"); // 字符串拼接
  print(str1 + " " + str2); // 字符串拼接

  // 三个单引号或三个双引号，让字符串原样输出
  var str3 = '''Hello
                World''';
  print(str3);

  /**
   * 2、数值类型（int、double）
   */
  int x = 1; // 必须为整型
  x = 20;
  print(x);

  double d = 99.9; // 整型或浮点型
  d = 100;
  print(d);

  /**
   * 3、布尔类型（bool）
   */
  bool flag = true;
  if (flag) {
    print('真');
  } else {
    print('假');
  }

  /**
   * 4、List（数组/集合）
   */
  // 第一种定义方式
  var list1 = ['a', 'b', 'c'];
  print(list1);

  // 第二种定义方式
  var list2 = new List();
  list2.add('a');
  list2.add('b');
  list2.add('c');
  print(list2);

  // 定义 List 时指定类型
  var list3 = new List<String>();
  // list3.add(1); // 错误
  list3.add("Hello");
  print(list3);

  /**
   * 5、Maps（字典）
   */
  // 第一种定义方式
  var person1 = {
    "name": "张三",
    "age": 20,
    "work": ["程序员", "送外卖"]
  };
  print(person1);
  print(person1["name"]);
  print(person1["work"]);

  // 第二种定义方式
  var person2 = new Map();
  person2["name"] = "李四";
  person2["age"] = 21;
  person2["work"] = ["程序员", "送外卖"];
  print(person2);
}
```
使用 `is` 关键词来判断类型
```dart
main() {
  var x = 123;
  if (x is String) {
    print('是 String 类型');
  } else if (x is int) {
    print('是 int 类型'); // 执行这里
  } else {
    print('是其他类型');
  }
}
```

### 运算符
Dart 运算符
```dart
算术运算符

    +    -    *    /     ~/ (取整)     %（取余）
    
关系运算符

    ==    ！=   >    <    >=    <=

逻辑运算符

    !  &&   ||

赋值运算符

    基础赋值运算符   =   ??=
    复合赋值运算符   +=  -=  *=   /=   %=  ~/=
```

基本使用：
```dart
main() {
  /**
   * 1、算术运算符
   */
  var x = 10;
  var y = 3;
  print(x / y); // 3.3333333333333335
  print(x ~/ y); // 3
  print(x % y); // 1

  /**
   * 2、关系运算符
   */
  var a = 123;
  var b = 123;
  var c = "123";
  var d = 123.0;
  if (a == b) {
    print('a 等于 b'); // 执行
  }
  if (a == c) {
    print('a 等于 c'); // 不执行
  }
  if (a == d) {
    print('a 等于 d'); // 执行
  }

  /**
   * 3、逻辑运算符
   */
  bool flag = false;
  print(!flag); // true（取反）

  /**
   * 4、基础赋值运算符（=   ??=）
   */
  int aa = 10;
  int bb;
  aa ??= 100;
  bb ??= 200; // 因为 b 为空，所以重新赋值
  print(aa); // 10
  print(bb); // 200

  /**
   * 5、复合赋值运算符   +=  -=  *=   /=   %=  ~/=
   */
  int xx = 10;
  xx += 10;
  print(xx); // 20
}
```

### 条件表达式
```dart
main() {
  /**
   * 1、if else / switch case
   */
  bool flag = true;
  if (flag) {
    print("true");
  } else {
    print("false");
  }

  String sex = "女";
  switch (sex) {
    case "男":
      print("男");
      break;
    case "女":
      print("女");
      break;
    default:
      print("不男不女");
      break;
  }

  /**
   * 2、三目运算符
   */
  String s = flag ? "我是 true" : "我是 false";
  print(s);

  /**
   * 3、?? 运算符
   */
  var a = 20;
  var b = a ?? 30;
  print(b); // 20
}
```

### 类型转换
```dart
main() {
  /**
   * 1、Number 与 String 类型之间的转换
   */
  int num = 20;
  double d = 30.0;
  String str = "50";
  print(num.toString()); // 20
  print(d.toString()); // 30.0
  print(int.parse(str)); // 50
  print(double.parse(str)); // 50.0
  print(int.tryParse("100a")); // null
  // print(int.parse("100a")); // 报错
  try {
    var myNum = double.parse("");
    print(myNum);
  } catch (err) {
    print(err); // FormatException: Invalid double
  }

  /**
   * 2、其他类型转换成 Booleans 类型
   */
  var myStr = '';
  if (myStr.isEmpty) {
    print('myStr 为空'); // 执行
  } else {
    print('myStr 不为空');
  }

  var myStr2;
  if (myStr2 == null) {
    print('myStr2 为空'); // 执行
  } else {
    print('myStr2 不为空');
  }

  var x = 0 / 0;
  print(x); // NaN
  if (x.isNaN) {
    print("x is NaN"); // 执行
  }
}
```

### 循环语句
```dart
main() {
  /**
   * 1、++  --   表示自增 自减 1
   * 在赋值运算里面，如果 ++ -- 写在前面，先运算后赋值，如果 ++ -- 写在后面，先赋值后运算
   */
  var a = 10;
  var b = a--;
  print(a); // 9
  print(b); // 10

  var aa = 100;
  var bb = --aa;
  print(aa); // 99
  print(bb); // 99

  /**
   * 2、for 语句
   */
  var list = ['新闻0', '新闻1', '新闻2', '新闻3', '新闻4'];
  for (var i = 0; i < list.length; i++) {
    if (i % 2 == 0) {
      print(list[i]);
    }
  }

  /**
   * 3、while / do while
   */
  var i = 1;
  var sum = 0;
  while (i <= 100) {
    sum += i;
    i++;
  }
  print(sum); // 5050

  i = 1;
  sum = 0;
  do {
    sum += i;
    i++;
  } while (i <= 100);
  print(sum); // 5050

  /**
   * 4、break / continue
   */
  sum = 0;
  for (var i = 0; i < 5; i++) {
    if (i > 3) {
      break; // 结束当前循环体
    }
    sum += i;
  }
  print(sum); // 1 + 2 + 3 = 6

  sum = 0;
  for (var i = 0; i < 5; i++) {
    if (i % 2 == 0) {
      continue; // 本次循环跳过下面代码
    }
    sum += i;
  }
  print(sum); // 1 + 3 = 4
}
```

### 集合类型

#### List
```dart
main() {
  /*
    List 里面常用的属性和方法：

    常用属性：
        length          长度
        reversed        翻转
        isEmpty         是否为空
        isNotEmpty      是否不为空
    常用方法：  
        add         增加
        addAll      拼接数组
        indexOf     查找  传入具体值
        remove      删除  传入具体值
        removeAt    删除  传入索引值
        fillRange   修改   
        insert(index,value);            指定位置插入    
        insertAll(index,list)           指定位置插入 List
        toList()    其他类型转换成 List
        join()      List 转换成字符串
        split()     字符串转化成 List
        forEach   
        map
        where
        any
        every
  */
  List myList = ['AA', 'BB', 'CC'];
  myList.add('DD');
  var newList = myList.reversed.toList();
  print(newList); // [DD, CC, BB, AA]

  // myList.fillRange(1, 3, 'aaa');
  // print(myList); // [AA, aaa, aaa, DD]

  var str = myList.join('-');
  print(str); // AA-BB-CC-DD
}
```

#### Set
```dart
main() {
  /**
   * Set
   * 1、用它最主要的功能就是去除数组重复内容；
   * 2、Set 是没有顺序且不能重复的集合，所以不能通过索引去获取值；
   */
  var set = new Set();
  set.add('AA');
  set.add('BB');
  set.add('AA');
  print(set); // {AA, BB}
  print(set.toList()); // [AA, BB]

  var myList = ['AA', 'BB', 'AA', 'CC', 'AA', 'BB'];
  var mySet = new Set();
  mySet.addAll(myList);
  print(mySet); // {AA, BB, CC}
  print(mySet.toList()); // [AA, BB, CC]
}
```

#### Map
```dart
main() {
  /*
    映射（Maps）是无序的键值对：

    常用属性：
        keys            获取所有的 key 值
        values          获取所有的 value 值
        isEmpty         是否为空
        isNotEmpty      是否不为空
    常用方法:
        remove(key)     删除指定 key 的数据
        addAll({...})   合并映射  给映射内增加属性
        containsValue   查看映射内的值  返回 true/false
        forEach   
        map
        where
        any
        every
  */
  Map zhangSan = {"name": "张三", "age": 20};
  var liSi = new Map();
  liSi["name"] = "李四";
  liSi["age"] = 21;

  print(zhangSan.keys.toList()); // [name, age]
  print(zhangSan.values.toList()); // [张三, 20]

  zhangSan.addAll({
    "sex": "男",
    "work": ['敲代码', '送外卖']
  });
  print(zhangSan); // {name: 张三, age: 20, sex: 男, work: [敲代码, 送外卖]}

  zhangSan.remove("sex");
  print(zhangSan); // {name: 张三, age: 20, work: [敲代码, 送外卖]}
}
```

#### 集合的 forEach、map、where、any、every 方法使用
```dart
main() {
  /**
   * List
   */
  var list = ['AA', 'BB', 'CC'];
  for (var i = 0; i < list.length; i++) {
    print(list[i]);
  }

  for (var item in list) {
    print(item);
  }

  list.forEach((element) {
    print(element);
  });

  var myList = [1, 3, 4];
  var newList = myList.map((e) {
    return e * 2;
  });
  print(newList.toList()); // [2, 6, 8]

  var newList2 = myList.where((element) {
    return element > 1;
  });
  print(newList2.toList()); // [3, 4]

  var f = myList.any((element) {
    // 只要集合里面有满足条件的就返回 true
    return element > 3;
  });
  print(f); // true

  var g = myList.every((element) {
    // 每一个都满足条件返回 true, 否则返回 false
    return element > 1;
  });
  print(g); // false

  /**
   * Set
   */
  var set = new Set();
  set.addAll(['1', '2', '3']);
  set.forEach((element) => print(element));

  /**
   * Map
   */
  var zhangSan = {"name": "张三", "age": 20};
  zhangSan.forEach((key, value) {
    print("$key---$value");
  });
}
```

### 函数

#### 函数定义、作用域
```dart
main() {
  /*
  内置方法/函数：

      print();

  自定义方法：
      自定义方法的基本格式：

      返回类型  方法名称（参数1，参数2,...）{
        方法体
        return 返回值;
      }
  */
  printInfo("Hello World");
  int sum = getSum(1, 2);
  print(sum);

  // 演示方法的作用域
  void exec() {
    void sayHello() {
      print("Hello");
    }

    sayHello();
  }

  exec(); // 调用方法
  // sayHello(); // 错误写法
}

void printInfo(String info) {
  print(info);
}

int getSum(int a, int b) {
  return a + b;
}
```

#### 函数传参、默认参数、可选参数、命名参数
```dart
main() {
  /**
   * 1、基本传参
   */
  var sum = getSum(10);
  print(sum); // 55

  /**
   * 2、可选参数
   */
  printUserInfo("张三", 20);
  printUserInfo("李四");

  /**
   * 3、默认参数
   */
  printUserInfo2("张三", "女", 20);
  printUserInfo2("李四");

  /**
   * 4、命名参数
   */
  printUserInfo3("王五", sex: "未知", age: 22);

  /**
   * 5、把方法当参数的方法
   */
  fun2(fun1);
}

int getSum(int n) {
  var sum = 0;
  for (var i = 1; i <= n; i++) {
    sum += i;
  }
  return sum;
}

/**
 * 带可选参数
 */
void printUserInfo(String name, [int age]) {
  if (age == null) {
    print("姓名：$name---年龄保密");
  } else {
    print("姓名：$name---年龄：$age");
  }
}

/**
 * 带默认参数
 */
void printUserInfo2(String name, [String sex = '男', int age]) {
  if (age == null) {
    print("姓名：$name---性别：$sex---年龄保密");
  } else {
    print("姓名：$name---性别：$sex---年龄：$age");
  }
}

/**
 * 带命名参数
 */
void printUserInfo3(String name, {String sex = '男', int age}) {
  if (age == null) {
    print("姓名：$name---性别：$sex---年龄保密");
  } else {
    print("姓名：$name---性别：$sex---年龄：$age");
  }
}

fun1() {
  print("fun1");
}

fun2(fn) {
  fn();
}
```

#### 箭头函数、匿名函数、自执行方法、方法递归
```dart
main() {
  /**
   * 1、箭头函数
   * 只能写一行，不能多行。
   */
  List myList = [1, 3, 4];
  myList.forEach((element) => print(element));

  var newList = myList.map((e) => e > 2 ? e * 2 : e);
  print(newList.toList()); // [1, 6, 8]

  /**
   * 2、匿名函数
   */
  var myNum = () {
    return 10;
  };
  print(myNum()); // 10

  /**
   * 3、自执行方法
   */
  ((int n) {
    print("我是自执行方法");
    print(n * 10); // 180
  })(18);

  /**
   * 4、方法递归
   */
  var sum = fn(100);
  print(sum); // 5050
}

int fn(int n) {
  if (n == 1) {
    return 1;
  }
  return n + fn(n - 1);
}
```

#### 闭包
```dart
main() {
  /**
   * 全局变量特点：全局变量常驻内存、全局变量污染全局；
   * 局部变量的特点：不常驻内存会被垃圾机制回收、不会污染全局；
   * 
   * 闭包：
   * 1、可以常驻内存，不污染全局；
   * 2、函数嵌套函数，内部函数会调用外部函数的变量或参数，变量或参数不会被系统回收(不会释放内存)；
   * 3、写法：函数嵌套函数，并 return 里面的函数，这样就形成了闭包；
   */
  var f = fn();
  f(); // 2
  f(); // 3
  f(); // 4
}

fn() {
  var x = 1;
  return () {
    x++;
    print(x);
  };
}
```

### 类与对象

#### 面向对象的介绍

面向对象编程(OOP)的三个基本特征是：封装、继承、多态

- **封装**：封装是对象和类概念的主要特性。封装，把客观事物封装成抽象的类，并且把自己的部分属性和方法提供给其他对象调用, 而一部分属性和方法则隐藏。
- **继承**：面向对象编程 (OOP) 语言的一个主要功能就是“继承”。继承是指这样一种能力：它可以使用现有类的功能，并在无需重新编写原来的类的情况下对这些功能进行扩展。
- **多态**：允许将子类类型的指针赋值给父类类型的指针, 同一个函数调用会有不同的执行效果 。

Dart 所有的东西都是对象，所有的对象都继承自 Object 类。

Dart 是一门使用类和**单继承**的面向对象语言，所有的对象都是类的实例，并且所有的类都是 Object 的子类

一个类通常由属性和方法组成。

> Dart 和其他面向对象语言不一样，Dart 中没有 public、private、protected 这些访问修饰符，但是我们可以使用 `_` 把一个属性或者方法定义成私有。

#### 类与对象的基本使用
```dart
class Person {
  String _name; // 加了下划线表示私有属性
  int age;

  // 默认构造函数的简写
  Person(this._name, this.age);

  Person.now() {
    print('我是命名构造函数');
  }

  Person.setInfo(String name, int age) {
    this._name = name;
    this.age = age;
  }

  void printInfo() {
    print("${this._name}---${this.age}");
  }

  void _sayHello() {
    print("_sayHello 是一个私有方法");
  }

  execRun() {
    this._sayHello();
  }
}
```

```dart
class Rect {
  int width;
  int height;

  Rect(this.width, this.height);

  getArea() {
    // 通过方法获取面积
    return this.width * this.height;
  }

  get area {
    // 通过属性获取面积
    return this.width * this.height;
  }

  set areaHeight(height) {
    this.height = height;
  }
}
```

```dart
import 'Person.dart';
import 'Rect.dart';

main() {
  Person p = new Person("张三", 20);
  p.printInfo();
  p.execRun();

  Person p2 = Person.now();

  Person p3 = Person.setInfo("李四", 21);
  p3.printInfo();

  var r = Rect(10, 5);
  print(r.getArea()); // 50
  print(r.area); // 50

  r.areaHeight = 10;
  print(r.area); // 100
}
```

#### 类中的初始化列表
```dart
class Rect {
  int width;
  int height;

  // 在构造函数体运行之前初始化实例变量
  Rect()
      : width = 3,
        height = 2 {
    print("${this.width}---${this.height}");
  }

  get area {
    return this.width * this.height;
  }
}
```

```dart
import 'Rect.dart';

main() {
  var r = Rect();
  print(r.area); // 6
}
```

#### 类的静态变量与静态函数

Dart 中的静态成员:

1、使用 static 关键字来实现类级别的变量和函数；

2、静态方法不能访问非静态成员，非静态方法可以访问静态成员；

```dart
class Person {
  static String name = '张三';
  static void showName() {
    print(name);
  }
}
```

```dart
import 'Person.dart';

main() {
  print(Person.name);
  Person.showName();
}
```

#### 对象操作符

Dart 中的对象操作符：

- `?` 条件运算符
- `as` 类型转换
- `is` 类型判断
- `..` 级联操作（连缀）

```dart
class Person {
  String name;
  num age;
  Person(this.name, this.age);

  void printInfo() {
    print("${this.name}---${this.age}");
  }
}
```

```dart
import 'Person.dart';

main() {
  Person p = Person("张三", 20);
  p?.printInfo(); // 张三---20

  Person p2;
  p2?.printInfo(); // 无打印结果

  if (p is Person) {
    p.name = '李四';
  }
  p.printInfo(); // 李四---20

  print(p is Object); // true

  var p3;
  p3 = '';
  p3 = new Person("王五", 22);
  p3.printInfo(); // 王五---22
  (p3 as Person).printInfo(); // 王五---22

  Person p4 = new Person("AA", 10);
  p4.printInfo(); // AA---10

  p4
    ..name = "BB"
    ..age = 11
    ..printInfo(); // BB---11
}
```

#### 类的继承

> 面向对象的三大特性：封装 、继承、多态

Dart 中的类的继承：  
- 子类使用 extends 关键词来继承父类；
- 子类会继承父类里面可见的属性和方法，但是不会继承构造函数；
- 子类能复写父类的方法；

```dart
class Person {
  String name;
  num age;
  Person(this.name, this.age);
  Person.xxx(this.name, this.age);

  void printInfo() {
    print("${this.name}---${this.age}");
  }
}
```

```dart
import 'Person.dart';

class Teacher extends Person {
  String sex;
  Teacher(String name, num age, String sex) : super(name, age) {
    this.sex = sex;
  }
  // Teacher(String name, num age, String sex) : super.xxx(name, age) {
  //   this.sex = sex;
  // }

  @override
  void printInfo() {
    // super.printInfo(); //自类调用父类的方法
    print("${this.name}---${this.age}---${this.sex}");
  }

  run() {
    print("${this.name}***${this.age}***${this.sex}");
  }
}
```

```dart
import 'Teacher.dart';

main() {
  Teacher t = new Teacher('张三', 20, '男');
  t.printInfo(); // 张三---20---男
  t.run(); // 张三***20***男
}
```

#### 抽象类

Dart 抽象类主要用于定义标准，子类可以继承抽象类，也可以实现抽象类接口。
- 抽象类通过 abstract 关键字来定义；
- 抽象方法不能用 abstract 声明，Dart 中没有方法体的方法我们称为抽象方法；
- 如果子类继承抽象类必须得实现里面的抽象方法；
- 如果把抽象类当做接口实现的话，必须得实现抽象类里面定义的所有属性和方法；
- 抽象类不能被实例化，只有继承它的子类可以；

**extends 抽象类和 implements 的区别：**
- 如果要复用抽象类里面的方法，并且要用抽象方法约束自类的话，我们就用 extends 继承抽象类；
- 如果只是把抽象类当做标准的话，我们就用 implements 实现抽象类；

```dart
abstract class Animal {
  eat(); // 抽象方法
  run(); // 抽象方法

  printInfo() {
    print('我是一个抽象类里面的普通方法');
  }
}
```

```dart
import 'Animal.dart';

class Dog extends Animal {
  @override
  eat() {
    print("Dog eat ...");
  }

  @override
  run() {
    print("Dog run ...");
  }
}
```

```dart
import 'Animal.dart';

class Cat extends Animal {
  @override
  eat() {
    print("Cat eat ...");
  }

  @override
  run() {
    print("Cat run ...");
  }
}
```

```dart
import 'Cat.dart';
import 'Dog.dart';

main() {
  Dog d = new Dog();
  d.eat(); // Dog eat ...
  d.printInfo(); // 我是一个抽象类里面的普通方法

  Cat c = new Cat();
  c.eat(); // Cat eat ...
  c.printInfo(); // 我是一个抽象类里面的普通方法
}
```

#### 多态

- 允许将子类类型的指针赋值给父类类型的指针，同一个函数调用会有不同的执行效果；
- 子类的实例赋值给父类的引用；
- 多态就是父类定义一个方法不去实现，让继承他的子类去实现，每个子类有不同的表现；

```dart
import 'Animal.dart';
import 'Cat.dart';
import 'Dog.dart';

main() {
  Animal d = new Dog();
  d.eat(); // Dog eat ...

  Animal c = new Cat();
  c.eat(); // Cat eat ...
}
```

### 接口

和 Java 一样，Dart 也有接口，但是和 Java 还是有区别的：
- Dart 的接口没有 interface 关键字定义接口，而是普通类或抽象类都可以作为接口被实现；
- 同样使用 implements 关键字进行实现；
- Dart 的接口有点奇怪，如果实现的类是普通类，需要将普通类中抽象的属性和方法全部覆写一遍；
- 因为抽象类可以定义抽象方法，普通类不可以，所以一般如果要实现像 Java 接口那样的方式，一般会使用抽象类；
- 建议使用抽象类定义接口；

> 一个类可以实现多个接口。

```dart
abstract class DB {
  // 当做接口   接口：就是约定 、规范
  String uri;
  insert(String data);
  delete(String id);
}
```

```dart
import 'DB.dart';

class MySQL implements DB {
  @override
  String uri;

  @override
  insert(String data) {
    print("MySQL insert");
  }

  @override
  delete(String id) {
    print("MySQL delete");
  }
}
```

```dart
import 'DB.dart';

class MsSQL implements DB {
  @override
  String uri;

  @override
  insert(String data) {
    print("MsSQL insert");
  }

  @override
  delete(String id) {
    print("MsSQL delete");
  }
}
```

```dart
import 'DB.dart';
import 'MySQL.dart';

main() {
  DB db = new MySQL();
  db.insert('123456');
}
```

### mixins

mixins 的中文意思是混入，就是在类中混入其他功能。

在 Dart 中可以使用 mixins 实现类似**多继承**的功能。

因为 mixins 使用的条件，随着 Dart 版本一直在变，这里讲的是 Dart 2.x 中使 用 mixins 的条件：
- 作为 mixins 的类只能继承自 Object, 不能继承其他类；
- 作为 mixins 的类不能有构造函数；
- 一个类可以 mixins 多个 mixins 类；
- mixins 绝不是继承，也不是接口，而是一种全新的特性；

mixins 的实例类型就是其超类的子类型。mixins 使用 `with` 关键字实现其功能。

```dart
class A {
  String info = "this is A";
  printA() {
    print("A");
  }
}
```

```dart
class B {
  printB() {
    print("B");
  }
}
```

```dart
import 'A.dart';
import 'B.dart';
import 'Person.dart';

class C extends Person with A, B {
  C(String name, num age) : super(name, age);
}
```

```dart
import 'A.dart';
import 'B.dart';
import 'C.dart';

main() {
  var c = new C("张三", 20);
  c.printA(); // A
  c.printB(); // B
  print(c.info); // this is A
  c.printInfo(); // 张三---20

  print(c is C); // true
  print(c is A); // true
  print(c is B); // true
}
```

### 泛型

通俗理解：泛型就是解决类、接口、方法的复用性、以及对不特定数据类型的支持（类型校验）。

#### 泛型方法
```dart
main() {
  print(getData(1)); // A
  print(getData2(2)); // 2
}

getData<T>(T value) {
  return "A";
}

T getData2<T>(T value) {
  // return "A"; // 错误，只能返回 T 类型
  return value;
}
```

#### 泛型类
```dart
main() {
  PrintClass p = new PrintClass<int>();
  p.add(1);
  p.add(2);
  p.add(3);
  // p.add("A"); // 报错
  p.printInfo();
}

class PrintClass<T> {
  List list = new List<T>();
  void add(T value) {
    this.list.add(value);
  }

  void printInfo() {
    for (var i = 0; i < this.list.length; i++) {
      print(this.list[i]);
    }
  }
}
```

### 泛型接口

需求：实现数据缓存的功能。有文件缓存和内存缓存，内存缓存和文件缓存按照接口约束实现。

1. 定义一个泛型接口，约束实现它的子类必须有 getByKey(key) 和 setByKey(key,value)；
2. 要求 setByKey 的时候的 value 的类型和实例化子类的时候指定的类型一致；

```dart
main() {
  MemoryCache m = MemoryCache<Map>();
  m.setByKey("index", {"name": "张三", "age": 20}); // 我是内存缓存，把 key = index  value = {name: 张三, age: 20} 写入到了内存中
}

abstract class Cache<T> {
  getByKey(String key);
  void setByKey(String key, T value);
}

class MemoryCache<T> implements Cache<T> {
  @override
  getByKey(String key) {
    return null;
  }

  @override
  void setByKey(String key, T value) {
    print("我是内存缓存，把 key = ${key}  value = ${value} 写入到了内存中");
  }
}

class FileCache<T> implements Cache<T> {
  @override
  getByKey(String key) {
    return null;
  }

  @override
  void setByKey(String key, T value) {
    print("我是文件缓存，把 key = ${key}  value = ${value} 写入到了文件中");
  }
}
```

### 库

在 Dart 中，库的使用是通过 `import` 关键字引入的。library 指令可以创建一个库，每个 Dart 文件都是一个库，即使没有使用 library 指令来指定。

```dart
Dart 中的库主要有三种：

    1、我们自定义的库     
          import 'lib/xxx.dart';
    2、系统内置库       
          import 'dart:math';    
          import 'dart:io'; 
          import 'dart:convert';
    3、pub 包管理系统中的库  
        https://pub.dev/packages
        https://pub.flutter-io.cn/packages
        https://pub.dartlang.org/flutter/

        1、需要在自己项目根目录新建一个 pubspec.yaml
        2、在 pubspec.yaml 文件配置名称、描述、依赖等信息
        3、然后运行 pub get 获取包下载到本地  
        4、项目中引入库 import 'package:http/http.dart' as http; 看文档使用
```

#### 导入自己本地库

```dart
import 'Person.dart';

main() {
  var p = new Person("张三", 20);
  p.printInfo(); // 张三---20
}
```

#### 导入系统内置库

```dart
import 'dart:math';

main() {
  print(max(10, 20)); // 20
  print(min(-1, -2)); // -2
}
```

#### 导入系统内置库实现请求数据

```dart
import 'dart:convert';
import 'dart:io';

main() async {
  var result = await getDataFromZhihuAPI();
  print(result);
}

// api 接口： http://news-at.zhihu.com/api/3/stories/latest
getDataFromZhihuAPI() async {
  // 1、创建 HttpClient 对象
  var httpClient = new HttpClient();
  // 2、创建 Uri 对象
  var uri = new Uri.http('news-at.zhihu.com', '/api/3/stories/latest');
  // 3、发起请求，等待请求
  var request = await httpClient.getUrl(uri);
  // 4、关闭请求，等待响应
  var response = await request.close();
  // 5、解码响应的内容
  return await response.transform(utf8.decoder).join();
}
```

#### async 和 await

这两个关键字的使用只需要记住两点：①、只有 async 方法才能使用 await 关键字调用方法；②、如果调用别的 async 方法必须使用 await 关键字；

async 是让方法变成异步，await 是等待异步方法执行完成。

```dart
main() async {
  var result = await testAsync();
  print(result); // testAsync
}

testAsync() async {
  return "testAsync";
}
```

#### 导入 pub 包管理系统中的库
```dart
pub 包管理系统：

1、从下面网址找到要用的库
        https://pub.dev/packages
        https://pub.flutter-io.cn/packages
        https://pub.dartlang.org/flutter/

2、创建一个 pubspec.yaml 文件，内容如下

    name: xxx
    description: A new flutter module project.
    dependencies:  
      date_format: ^1.0.9

3、配置 dependencies

4、运行 pub get 获取远程库

5、看文档引入库使用
```

```dart
import 'package:date_format/date_format.dart';

main() {
  print(formatDate(DateTime(1989, 02, 21), [yyyy, '-', mm, '-', dd])); // 1989-02-21
}
```

#### 库重命名与冲突解决
```dart
当引入两个库中有相同名称标识符的时候，如果是 Java, 通常我们通过写上完整的包名路径来指定使用的具体标识符，甚至不用 import 都可以，
但是 Dart 里面是必须 import 的。当冲突的时候，可以使用 as 关键字来指定库的前缀。如下例子所示：

import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;


Element element1 = new Element();           // Uses Element from lib1.
lib2.Element element2 = new lib2.Element(); // Uses Element from lib2.
```

#### 部分导入
```dart
如果只需要导入库的一部分，有两种模式：

模式一：只导入需要的部分，使用 show 关键字，如下例子所示：

import 'package:lib1/lib1.dart' show foo;

模式二：隐藏不需要的部分，使用 hide 关键字，如下例子所示：

import 'package:lib2/lib2.dart' hide foo;    
```

MyMath.dart
```dart
getOne() {
  return "one";
}

getTwo() {
  return "two";
}
```

```dart
import 'MyMath.dart' show getOne;

main() {
  print(getOne());
  // print(getTwo()); // 报错
}
```

#### 延迟加载
```dart
延迟加载也称为懒加载，可以在需要的时候再进行加载。
懒加载的最大好处是可以减少 APP 的启动时间。

懒加载使用 deferred as 关键字来指定，如下例子所示：

import 'package:deferred/hello.dart' deferred as hello;

当需要使用的时候，需要使用 loadLibrary() 方法来加载：

greet() async {
    await hello.loadLibrary();
    hello.printGreeting();
}
```

### 2.13 之后的一些新特性

#### Null safety

```dart
/**
  Null safety 翻译成中文的意思是空安全。

  null safety 可以帮助开发者避免一些日常开发中很难被发现的错误，并且额外的好处是可以改善性能。

  Flutter2.2.0（2021 年 5 月 19 日发布）之后的版本都要求使用 null safety。

  1、? 可空类型
  2、! 类型断言
*/

String? getData(apiUrl) {
  if (apiUrl != null) {
    return "this is server data";
  }
  return null;
}

void printLength(String? str) {
  try {
    print(str!.length);
  } catch (e) {
    print("str is null");
  }
}

void main() {
  // 1、? 可空类型
  String? str = "abc"; // String?  表示 str 是一个可空类型
  str = null; // 允许设置为 null
  print(str);

  print(getData("http://www.baidu.com"));
  print(getData(null));

  // 2、! 类型断言
  String? str2 = "this is str";
  // str2 = null;
  // 如果 str2 不等于 null 会打印 str2 的长度，如果等于 null 会抛出异常
  print(str2!.length);

  printLength("str");
  printLength(null);
}
```

#### late 关键字

```dart
/**
 * late 关键字主要用于延迟初始化。
 */
class Person {
  late String name;
  late int age;
  void setInfo(String name, int age) {
    this.name = name;
    this.age = age;
  }

  String getInfo() {
    return "${this.name}---${this.age}";
  }
}

void main(args) {
  Person p = new Person();
  p.setInfo("张三", 20);
  print(p.getInfo());
}
```

#### required 关键字

```dart
/**
 * required 翻译成中文的意思是需要、依赖
 * 
 * 最开始 @required 是注解，现在它已经作为内置修饰符，主要用于允许根据需要标记任何命名参数（函数或类），
 * 使得它们不为空。因为可选参数中必须有个 required 参数或者该参数有个默认值。
 */
String printInfo(String username, {int age = 10, String sex = "男"}) {
  return "姓名:$username---性别:$sex--年龄:$age";
}

String printInfo2(String username, {required int age, required String sex}) {
  return "姓名:$username---性别:$sex--年龄:$age";
}

void main(args) {
  print(printInfo('张三'));

  print(printInfo('张三', age: 20, sex: "女"));

  // age 和 sex 必须传入
  print(printInfo2('张三', age: 22, sex: "女"));
}
```

```dart
/**
 * name 可以传入也可以不传入，age 必须传入
 */
class Person {
  String? name; // 可空属性
  int age;
  Person({this.name, required this.age}); // age 必须传入

  String getInfo() {
    return "${this.name}---${this.age}";
  }
}

void main(args) {
  Person p = new Person(name: "张三", age: 20);
  print(p.getInfo()); // 张三---20

  Person p1 = new Person(age: 20);
  print(p1.getInfo()); // null---20
}
```

### 性能优化

#### 回顾 Dart 常量

```dart
/**
Dart 常量: final 和 const 修饰符。
  1、const 声明的常量是在编译时确定的，永远不会改变；
  2、final 声明的常量允许声明后再赋值，赋值后不可改变，final 声明的变量是在运行时确定的；
  3、final 不仅有 const 的编译时常量的特性，最重要的它是运行时常量，并且 final 是惰性初始化，即在运行时第一次使用前才初始化。
*/

void main() {
  // const 常量
  // const PI = 3.14;
  // PI = 3.14159; // const 定义的常量没法改变
  // print(PI);

  // final 常量
  // final PI = 3.14;
  // print(PI);

  final a;
  a = 13;
  // a = 14;
  print(a);

  final d = new DateTime.now();
}
```

#### const、identical 函数

```dart
/**
 dart:core 库中 identical 函数的用法介绍如下：

用法:
bool identical(
   Object? a,    
   Object? b   
)
检查两个引用是否指向同一个对象。
 */

void main() {
  // var o1 = new Object();
  // var o2 = new Object();
  // print(identical(o1, o2)); // false，不共享存储空间
  // print(identical(o1, o1)); // true，共享存储空间

  // var o1 = Object();
  // var o2 = Object();
  // print(identical(o1, o2)); // false
  // print(identical(o1, o1)); // true

  // 表示实例化常量构造函数
  // o1 和 o2 共享了存储空间
  // var o1 = const Object();
  // var o2 = const Object();
  // print(identical(o1, o2)); // true，共享存储空间
  // print(identical(o1, o1)); // true，共享存储空间

  // print(identical([2], [2])); // false

  // var a = [2];
  // var b = [2];
  // print(identical(a, b)); // false，不共享存储空间

  const c = [2];
  const d = [3];
  print(identical(c, d)); // false，不共享存储空间

  // 发现：const 关键词在多个地方创建相同的对象的时候，内存中只保留了一个对象。
  // 共享存储空间条件：1、常量   2、值相等。
}
```

#### 普通构造函数

```dart
class Container {
  int width;
  int height;
  Container({required this.width, required this.height});
}

void main() {
  var c1 = new Container(width: 100, height: 100);
  var c2 = new Container(width: 100, height: 100);
  print(identical(c1, c2)); // false，c1 和 c2 在内存中存储了 2 份
}
```

#### 常量构造函数

```dart
/*
常量构造函数总结如下几点：
  1、常量构造函数需以 const 关键字修饰；
  2、const 构造函数必须用于成员变量都是 final 的类；
  3、如果实例化时不加 const 修饰符，即使调用的是常量构造函数，实例化的对象也不是常量实例；
  4、实例化常量构造函数的时候，多个地方创建这个对象，如果传入的值相同，只会保留一个对象；
  5、Flutter 中 const 修饰不仅仅是节省组件构建时的内存开销，Flutter 在需要重新构建组件的时候，由于这个组件是不应该改变的，重新构建没有任何意义，因此 Flutter 不会重新构建 const 组件。 
*/

// 常量构造函数
class Container {
  final int width;
  final int height;
  const Container({required this.width, required this.height});
}

void main() {
  var c1 = Container(width: 100, height: 100);
  var c2 = Container(width: 100, height: 100);
  print(identical(c1, c2)); // false

  var c3 = const Container(width: 100, height: 100);
  var c4 = const Container(width: 100, height: 100);
  print(identical(c3, c4)); // true

  var c5 = const Container(width: 100, height: 110);
  var c6 = const Container(width: 120, height: 100);
  print(identical(c5, c6)); // false
}
// 实例化常量构造函数的时候，多个地方创建这个对象，如果传入的值相同，只会保留一个对象。
```

