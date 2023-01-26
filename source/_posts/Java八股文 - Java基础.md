---
title: Java八股文 - Java基础
copyright: true
mathjax: false
categories:
  - Java八股文
date: 2023-01-23 10:05:51
tags:
toc: true
urlname: java-basic-knowledge
---

> 整理的Java基础相关知识点和面试题，部分内容摘自网络，如有侵权请联系我～	<!--more-->

## Java概述

### JVM、JRE和JDK的关系

JVM：Java Virtual Machine，Java 虚拟机，Java 程序需要运行在虚拟机上，不同的平台有自己的虚拟机，因此 Java 语言可以实现跨平台。

JRE：Java Runtime Environment，Java 运行环境，包括 Java 虚拟机和 Java 程序所需的核心类库等。

JDK：Java Development Kit，Java 开发工具包，包括了 JRE 和开发工具。

### 什么是跨平台性？原理是什么

所谓跨平台性，是指 Java 语言编写的程序，一次编译，到处运行。

实现原理：Java 程序是通过 Java 虚拟机在系统平台上运行的，只要该系统可以安装相应的 Java 虚拟机，该系统就可以运行 Java 程序。

### Java语言有哪些特点

面向对象（封装，继承，多态）

平台无关性（Java 虚拟机实现平台无关性）

### 什么是字节码？采用字节码的最大好处是什么

**字节码**：Java 源代码经过虚拟机编译器编译后产生的文件（即扩展为.class 的文件），它不面向任何特定的处理器，只面向虚拟机。（javac HelloWorld.Java → HelloWorld.class → Java HelloWorld 执行程序）

**编译与解释并存**

Java 源代码(.Java 文件)---->编译器---->jvm 可执行的 Java 字节码(.class 文件)---->jvm---->jvm 中解释器----->机器可执行的二进制机器码---->程序运行

### 什么是Java程序的主类？

一个程序中可以有多个类，但只能有一个类是主类。在 Java 应用程序中，这个主类是指包含 main()方法的类。
主类是 Java 程序执行的入口。（main 方法除了是个主方法以外，和普通的静态方法没有区别，都可以重载、调用、继承）

### Java和C++的区别

相同点：都是**面向对象**的语言，都支持**封装、继承和多态**

不同点：

- **Java不提供指针**来直接访问内存，程序内存更加安全，有自动内存管理；C++用指针管理内存

- Java 的**类是单继承**的，C++支持多重继承；虽然 Java 的类不可以多继承，但是接口可以多继承。

	为什么 Java 中类不支持多继承？——多重继承的钻石问题

	![多重继承的钻石问题](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes20190622142003813.png)
	类 B 和类 C 继承自类 A，且都重写了类 A 中的同一个方法，而类 D 同时继承了类 B 和类 C（多继承)，那么此时类 D 会继承 B、C 的方法，那对于 B、C 重写的 A 中的方法，类 D 会继承哪一个呢？

- Java 有**自动内存管理**机制，不需要程序员手动释放无用内存

## 基础语法

### 数据类型

**定义**：Java 语言是强类型语言，对于每一种数据都定义了明确的具体的数据类型，在内存中分配了不同大小的内存空间。

**分类**

- 基本数据类型
	- 数值型
		- 整数类型(byte,short,int,long)
		- 浮点类型(float,double)
	- 字符型(char)
	- 布尔型(boolean)
- 引用数据类型
	- 类(class)
	- 接口(interface)
	- 数组([])

**Java基本数据类型图**

![Java基本数据类型图](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL0pvdXJXb24vaW1hZ2UvbWFzdGVyL0phdmElRTUlOUYlQkElRTclQTElODAlRTglQUYlQUQlRTYlQjMlOTUvSmF2YSVFNSU5RiVCQSVFNiU5QyVBQyVFNiU5NSVCMCVFNiU4RCVBRSVFNyVCMSVCQiVFNSU5RSU4Qi5wbmc.png)

### 类型转换

自动类型转换

> 两种数据类型彼此兼容
>
> 低级类型数据转换成高级类型数据

- 数值型数据的转换：byte→short→int→long→float→double
- 字符型转换为整型：char→int。

强制类型转换

> 当两种数据类型不兼容，或高级类型数据转换成低级类型数据时，自动转换将无法进行（编译错误），这时就需要进行强制类型转换

### 访问修饰符

![访问修饰符](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL0pvdXJXb24vaW1hZ2UvbWFzdGVyL0phdmElRTUlOUYlQkElRTclQTElODAlRTglQUYlQUQlRTYlQjMlOTUvSmF2YSVFOCVBRSVCRiVFOSU5NyVBRSVFNCVCRiVBRSVFOSVBNSVCMCVFNyVBQyVBNi5wbmc.png)

### 关键字

#### final 有什么用？

用于修饰类、方法和变量；

- 被 final 修饰的类不可以被继承

- 被 final 修饰的方法不可以被重写

- 被 final 修饰的变量不可以被改变，被 final 修饰**不可变的是变量的引用，而不是引用指向的内容**，引用指向的内容是可以改变

	```java
	final int[] nums = new int[]{1, 2, 3};
	nums[0] = 11;
	System.out.println(Arrays.toString(nums));	// [11, 2, 3]
	```

#### final、finally、finalize区别

- final 是 Java 修饰符，可以修饰类、方法、变量，修饰类表示该类不能被继承、修饰方法表示该方法不能被重写、修饰变量表示该变量不可以被改变。
- finally 一般作用在 try-catch 代码块中，在处理异常的时候，通常我们将一定要执行的代码方法 finally 代码块
	中，表示不管是否出现异常，该代码块都会执行，一般用来存放一些关闭资源的代码。
- finalize 是 Object 类的一个方法，该方法一般由垃圾回收器来调用，在执行垃圾回收前，由垃圾回收器调用对象的 finalize()方法，并且在该方法中对象可能可以复活。

#### this关键字的用法

指向对象本身的一个指针。

#### super关键字的用法

指向自己父类对象（最近的父类）的一个指针。

#### static存在的主要意义

static**静态变量**：和类一起加载，**以致于即使没有创建对象，也能使用属性和调用方法。**

static**静态代码块**：在类初次被加载的时候，会按照 static 块的顺序来执行每个 static 块，并且只会执行一次。
加载 → 链接（验证、准备、解析） → 初始化

static 类型的变量在准备阶段进行默认初始化，但是有 static final 修饰的会在准备阶段直接赋值， 在初始化阶段进行显式初始化。

#### static注意事项

1、静态只能访问静态。 2、非静态既可以访问非静态的，也可以访问静态的。

### 初始化顺序

静态属性 → 静态方法块 → 普通属性 → 普通方法块 → 构造函数

### 流程控制语句

#### break 、continue、return 的区别及作用

break 结束当前的循环体

continue 结束正在执行的循环，进入下一个循环条件

return 结束当前的方法，直接返回

#### 跳出当前的多重嵌套循环

在 Java 中，要想跳出多重循环，可以在外面的循环语句前定义一个标号，然后在里层循环体的代码中使用带有标号的 break 语句，即可跳出外层循环。例如：

```java
public static void main(String[] args) {
    ok:
    for (int i = 0; i < 10; i++) {
        for (int j = 0; j < 10; j++) {
            System.out.println("i=" + i + ",j=" + j);
            if (j == 5) {
                break ok;
            }
        }
    }
}
```

## 面向对象

### 面向对象三大特性

#### 面向对象的特征有哪些方面

**抽象**：抽象是将一类对象的共同特征总结出来构造类的过程，包括数据抽象和行为抽象两方面。抽象只关注对象有哪些属性和行为，并不关注这些行为的细节是什么。

**封装**

封装把一个对象的属性私有化，同时提供一些可以被外界访问的属性的方法，如果属性不想被外界访问，我们大可不必提供方法给外界访问。

**继承**

继承是**使用已存在的类作为基础建立新类**的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能。通过使用继承我们能够非常方便地复用以前的代码。

关于继承如下 3 点请记住：

1. 子类拥有父类非 private 的属性和方法。
2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法，override 重写。

**多态**

所谓多态就是指一个引用变量到底会指向哪个类的实例对象，该引用变量调用的方法到底是哪个类中实现的方法，必须在程序运行期间才能确定，编译期无法确定。

父类的引用指向子类实例对象、接口指向实现类实例对象。提高了程序的拓展性。

在 Java 中有两种形式可以实现多态：继承（多个子类对同一方法的重写）和接口（实现接口并覆盖接口中同一方法）。

方法重载（overload）实现编译时的多态性（也称为前绑定），方法重写（override）实现运行时的多态性（也称为后绑定）。

Java 实现多态有三个必要条件：继承、重写、向上转型。

> 继承：在多态中必须存在有继承关系的子类和父类。
>
> 重写：子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法。
>
> 向上转型：在多态中需要将子类的引用赋给父类对象，只有这样该引用才能够具备技能调用父类的方法和子类的方法。

只有满足了上述三个条件，我们才能够在同一个继承结构中使用统一的逻辑实现代码处理不同的对象，从而达到执行不同的行为。

#### 抽象类和接口的对比

抽象类是对类的抽象，用来**捕捉通用特性**。接口是**行为的抽象**，抽象方法的集合。

**相同点**

- 接口和抽象类都不能实例化
- 都位于继承的顶端，用于被其他类实现或继承
- 都包含抽象方法，其子类都必须重写这些抽象方法

**不同点**

| 参数       | 抽象类                             | 接口                                             |
| ---------- | ---------------------------------- | ------------------------------------------------ |
| 声明       | 抽象类使用abstract关键字声明       | 接口使用interface关键字声明                      |
| 实现       | 子类使用extends关键字来继承抽象类  | 子类使用implements关键字来实现接口               |
| 构造器     | 抽象类可以有构造器                 | 接口不能有构造器                                 |
| 访问修饰符 | 抽象类中的方法可以是任意访问修饰符 | 接口方法默认修饰符是public，也可以不加访问修饰符 |
| 多继承     | 一个类最多只能继承一个抽象类       | 一个类可以实现多个接口                           |

#### 普通类和抽象类有哪些区别？

- 普通类不能包含抽象方法，抽象类可以包含抽象方法。
- 抽象类不能直接实例化，普通类可以直接实例化。

#### 成员变量和局部变量的区别

|              | 成员变量               | 局部变量                                     |
| ------------ | ---------------------- | -------------------------------------------- |
| **作用域**   | 整个类可使用           | 方法体内使用                                 |
| **存储位置** | 属于对象，存在堆内存中 | 属于方法，存在栈帧中                         |
| **生命周期** | 和对象同生命周期       | 和方法同生命周期                             |
| **初始值**   | 不指定的话有默认初始值 | 必须手动初始化否则<br />不能使用（编译错误） |

**使用原则**

在使用变量时需要遵循的原则为：就近原则
首先在局部范围找，有就使用；接着在成员位置找。

```java
public class Main {
    int a = 2;
    public static void main(String[] args) {
        int a = 1;
        System.out.println(a);	// 1
    }
}
```

### 内部类

#### 什么是内部类？

在 Java 中，可以将一个类的定义放在另外一个类的定义内部，这就是**内部类**。

#### 内部类的分类有哪些

内部类可以分为四种：**静态内部类、成员内部类、局部内部类、匿名内部类**。

##### 静态内部类

定义在类内部的静态类，就是静态内部类。

静态内部类可以访问外部类所有的静态变量，而不可访问外部类的非静态变量

##### 成员内部类

定义在类内部，成员位置上的非静态类，就是成员内部类。

成员内部类可以访问外部类所有的变量和方法，包括静态和非静态，私有和公有。

##### 局部内部类

定义在方法中的内部类，就是局部内部类。

定义在实例方法中的局部类可以访问外部类的所有变量和方法，定义在静态方法中的局部类只能访问外部类的静态变量和方法

##### 匿名内部类

匿名内部类就是没有名字的内部类，日常开发中使用的比较多。匿名内部类必须继承或实现一个已有的接口 ，匿名内部类不能定义任何静态成员和静态方法

#### 局部内部类和匿名内部类访问局部变量的时候，为什么变量必须要加上final？

局部内部类和匿名内部类访问局部变量的时候，为什么变量必须要加上 final 呢？它内部原理是什么呢？

先看这段代码：

```java
public class Outer {
    void outMethod(){
        final int a =10;
        class Inner {
            void innerMethod(){
                System.out.println(a);
            }
        }
    }
}
```

以上例子，为什么要加 final 呢？是因为**生命周期不一致**

局部变量存储在栈中，当方法执行结束后，如果不用 final 修饰那么该局部变量就被销毁。而局部内部类对局部变量的引用依然存在，如果局部内部类要调用局部变量时，就会出错。
加了 final，可以确保局部内部类使用的变量与外层的局部变量区分开，解决了这个问题。

### 重写与重载

#### 构造器（constructor）是否可被重写（override）

构造器不能被继承，因此不能被重写，但可以被重载。

#### 重载（Overload）和重写（Override）的区别。重载的方法能否根据返回类型进行区分？

方法的重载和重写都是实现多态的方式，区别在于重载实现的是**编译时的多态性**，而重写实现的是**运行时的多态性**。

重载：发生在**同一个类**中，**方法名相同 参数列表不同**（参数类型不同、个数不同、顺序不同），与方法返回值和访问修饰符无关

重写：发生在**父子类**中，方法名、参数列表必须相同，返回值类型小于等于父类，抛出的异常小于等于父类，**访问修饰符大于等于父类**（里氏代换原则）

### 对象相等判断

#### == 和 equals 的区别是什么

**==** : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象。
(基本数据类型 == 比较的是值，引用数据类型 == 比较的是内存地址)

**equals()** : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

情况 1：类没有重写 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。

情况 2：类重写了 equals() 方法。一般，我们都重写 equals() 方法来两个对象的内容相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。

#### hashCode 与 equals

**hashCode()介绍**

hashCode() 的作用是获取哈希码（本地方法），也称为散列码；它实际上是返回一个 int 整数。hashCode() 定义在 JDK 的 Object 类中，这就意味着 Java 中的任何类都有 hashCode()方法。

**为什么要有 hashCode**

**我们以“hashSet 如何检查重复”为例子来说明为什么要有 hashCode**：

当你把对象加入 hashSet 时，hashSet 会先计算对象的 hashcode 值来判断对象加入的位置，如果没有相符的 hashcode，hashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 equals()方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，表明已经有了相同的对象了，hashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。

**hashCode()与equals()的相关规定**

- 如果两个对象的 hashCode 值相等，那这两个对象不一定相等（哈希碰撞）。
- 如果两个对象的 hashCode 值相等并且 equals()方法也返回 true，我们才认为这两个对象相等。
- 如果两个对象的 hashCode 值不相等，我们就可以直接认为这两个对象不相等。

#### 只重写 hashCode 可以吗？

如果只重写 hashCode，不重写 equals，那么如果两个对象判断 hashCode 相同了，以为这俩对象相同，那么只会插入一次到 set 里面，而实际上这两个不一样一个是 A 一个是 B，都需要插入。

> 两个对象的 hashCode 相同，但是内容可能不同，在 set 中明明应该是两个不同的对象，这时候只通过 hashCode 判断他俩相同会导致只插入一个元素。

#### 只重写 equals可以吗？

如果在重写 equals 时，不重写 hashCode，就会引起比如说将两个相等的自定义对象存储在 Set 集合时，会将这两个对象都进行插入。（无法去重）

> 如果定义一个set里面存放Person对象，Person只重写了equals方法即可以判断对象内容是否相同。
>
> 如果只重写了 equals 方法，那么默认情况下，Set 进行去重操作时，会先判断两个对象的 hashCode 是否相同，此时因为没有重写 hashCode 方法，所以会直接执行 Object 中的 hashCode 方法，而 Object 中的 hashCode 方法对比的是两个不同引用地址的对象，所以结果是 false，那么 equals 方法就不用执行了，直接返回的结果就是 false：两个对象不是相等的，于是就在 Set 集合中插入了两个相同的对象。
>
> 但是，如果在重写 equals 方法时，也重写了 hashCode 方法，那么在执行判断时会去执行重写的 hashCode 方法，此时对比的是两个对象的所有属性的 hashCode 是否相同，于是调用 hashCode 返回的结果就是 true，再去调用 equals 方法，发现两个对象确实是相等的，于是就返回 true 了，因此 Set 集合就不会存储两个一模一样的数据了，于是整个程序的执行就正常了。

### 值传递

#### 为什么 Java 中只有值传递

Java 采用按值调用，方法得到的是所有参数值的一个拷贝，也就是说，方法不能修改传递给它的任何参数变量的内容。

example 1

```java
public static void main(String[] args) {
    int num1 = 10;
    int num2 = 20;

    swap(num1, num2);

    System.out.println("num1 = " + num1);
    System.out.println("num2 = " + num2);
}

public static void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;

    System.out.println("a = " + a);
    System.out.println("b = " + b);
}
```

**结果**：

```java
a = 20
b = 10
num1 = 10
num2 = 20
```

**解析**：

![swap(int a, int b)](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNDAyMzk3Mi00ZWQyZjQ0NTlmN2MyYzkwLmpwZw.png)

在 swap 方法中，a、b 的值进行交换，并不会影响到 num1、num2。因为 a、b 中的值，只是从 num1、num2 的复制过来的。也就是说，a、b 相当于 num1、num2 的副本，副本的内容无论怎么修改，都不会影响到原件本身。

**通过上面例子，我们已经知道了一个方法不能修改一个基本数据类型的参数，而对象引用作为参数就不一样，请看 example2.**

example 2

```java
    public static void main(String[] args) {
        int[] arr = { 1, 2, 3, 4, 5 };
        System.out.println(arr[0]);
        change(arr);
        System.out.println(arr[0]);
    }

    public static void change(int[] array) {
        // 将数组的第一个元素变为0
        array[0] = 0;
    }
```

**结果**：

```undefined
1
0
```

**解析**：

![change(int[] array)](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNDAyMzk3Mi1kYWI5Mzg5ZWRmMjIxNmIzLmpwZw.png)

array 被初始化 arr 的拷贝也就是一个对象的引用，也就是说 array 和 arr 指向的时同一个数组对象。 因此，外部对引用对象的改变会反映到所对应的对象上。

**通过 example2 我们已经看到，实现一个改变对象参数状态的方法并不是一件难事。理由很简单，方法得到的是对象引用的拷贝，对象引用及其他的拷贝同时引用同一个对象。**

example 3

```java
public class Test {

    public static void main(String[] args) {
        Student s1 = new Student("小张");
        Student s2 = new Student("小李");
        Test.swap(s1, s2);
        System.out.println("s1:" + s1.getName());
        System.out.println("s2:" + s2.getName());
    }

    public static void swap(Student x, Student y) {
        Student temp = x;
        x = y;
        y = temp;
        System.out.println("x:" + x.getName());
        System.out.println("y:" + y.getName());
    }
}
```

**结果**：

```undefined
x:小李
y:小张
s1:小张
s2:小李
```

**解析**：

交换之前：

![before swap(Student x, Student y)](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNDAyMzk3Mi03MmIwYzFmYjlmM2IwNzc2LmpwZw.png)

交换之后：

![after swap(Student x, Student y)](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNDAyMzk3Mi0zYWUzNzY4NGNiMzg2Njc4LmpwZw.png)

通过上面两张图可以很清晰的看出： **方法并没有改变存储在变量 s1 和 s2 中的对象引用。swap方法的参数x和y被初始化为两个对象引用的拷贝，这个方法交换的是这两个拷贝**

#### 值传递和引用传递有什么区别

值传递：指的是在方法调用时，传递的参数是按值的拷贝传递，传递的是值的拷贝，也就是说传递后就互不相关了。

引用传递：指的是在方法调用时，传递的参数是按引用进行传递，其实传递的引用的地址，也就是变量所对应的内存空间的地址。传递的是值的引用，也就是说传递前和传递后都指向同一个引用（也就是同一个内存空间）。

## IO流

> 参考：https://blog.csdn.net/weixin_44579258/article/details/90758359

阻塞和非阻塞是指进程访问的数据如果尚未准备就绪，进程是否需要等待
同步和异步是指访问数据的机制，同步一般主动请求等待 IO 操作完毕的方式。当数据就绪后，再读写的时候必须阻塞，异步则主动请求数据后便可以继续处理其他任务，随后等待 IO 完毕通知，这可以使进程在数据读写时也不阻塞

准备数据：网卡 → 内核
数据就绪：数据拷贝到了内核
拷贝数据：将内核数据拷贝到用户空间

### Blocking I/O

**BIO 属于同步阻塞 IO 模型** 。

同步阻塞 IO 模型中，应用程序**发起 read 调用**后，会一直阻塞，直到内核准备数据、数据就绪，数据从内核拷贝到用户空间。

![BIO](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes20190603201109632.png)

### NIO (Non-blocking I/O)

同步非阻塞 IO 模型中，用户进程会**一直发起 read 调用**（如果内核返回 error 则说明数据未准备就绪），直到数据准备就绪，用户发起 read 调用时，数据从内核空间拷贝到用户空间（用户进程是阻塞的）。

> 相比于同步阻塞 IO 模型，同步非阻塞 IO 模型确实有了很大改进。通过轮询操作，避免了一直阻塞。

![NIO](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes20190603201218527.png)

### IO多路复用（NIO）

select/epoll 的好处就在于单个 process 就可以同时处理多个网络连接的 IO。

![NIO](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes2019060320125821.png)

IO 复用和同步阻塞本质一样，在 NIO 中，每次用户进程进行 read 轮询数据是否准备就绪，而在 IO 多路复用中，利用了新的 select 系统调用，由内核来负责本来是请求进程该做的轮询操作。

当用户线程调用 select，那么整个进程会被阻塞，而同时，kernel 内核会“监视”所有 select 负责的 socket，当任何一个 socket 中的数据准备好了，select 就会返回可读条件让用户进程发起 read 调用，用户进程发起调用 read 操作，将数据从内核空间拷贝到用户空间。


### Asynchronous I/O

异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

用户发起 read 调用，此时用户进程不会阻塞。内核将数据准备就绪并将其从内核拷贝到用户空间后，内核给用户进程发送一个信号给之前调用 read 的进程，数据已经拷贝到用户空间了。

![AIO](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes2019060320135956.png)

## 反射

### 什么是反射机制？

JAVA 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为 Java 语言的反射机制。

### 反射机制优缺点

- **优点：** 运行期类型的判断，动态加载类，提高代码灵活度。
- **缺点：** 性能瓶颈：反射相当于一系列解释操作，通知 JVM 要做的事情，性能比直接的 Java 代码要慢很多。

### 反射机制的应用场景有哪些？

①我们在使用 JDBC 连接数据库时使用 Class.forName()通过反射加载数据库的驱动程序；

②Spring 通过 XML 配置模式装载 Bean 的过程

1. 加载配置文件，解析成 BeanDefinition 放在 Map 里，map 中存放`<BeanName，Class对象>`的映射
2. 调用 getBean 的时候，从 BeanDefinition 所属的 Map 里，拿出 Class 对象使用反射进行实例化，同时，如果有依赖关系，将递归调用 getBean 方法 —— 完成依赖注入

### Java获取反射的三种方法

1.通过 new 对象实现反射机制 2.通过路径实现反射机制 3.通过类名实现反射机制

## 包装类

#### 自动装箱与拆箱

**装箱**：将基本类型用它们对应的引用类型包装起来；`Integer a = 1;`

**拆箱**：将包装类型转换为基本数据类型；`int b = a;`

#### int 和 Integer 有什么区别

引入包装类是为了能够将这些**基本数据类型当成对象操作**

Java 为每个原始类型提供了包装类型：

原始类型: boolean，char，byte，short，int，long，float，double

包装类型：Boolean，Character，Byte，Short，Integer，Long，Float，Double

#### Integer a = 127 与 Integer b = 127相等吗

> 包装类和基本数据类型比较，会将包装类拆箱成基本数据类型然后比较，
>
> 只有当包装类是自动装箱，并且数值范围在 [-128, 127] 时，才会使用 cache 中的对象。

对于对象引用类型：==比较的是对象的内存地址。
对于基本数据类型：==比较的是值。

```java
public static void main(String[] args) {
    Integer a = new Integer(3);
  	Integer b = 3;
    int c = 3;
    Integer d = 3;
    System.out.println(a == b); // false，两个对象比较
    System.out.println(a == c); // true，a自动拆箱
    System.out.println(b == c); // true，b自动拆箱
    System.out.println(b == d); // true，包装类自动装箱且数值范围在cache范围内

    Integer a = new Integer(128);
    Integer b = 128;
    int c = 128;
    Integer d = 128;
    System.out.println(a == b); // false，两个对象比较
    System.out.println(a == c); // true，a自动拆箱
    System.out.println(b == c); // true，b自动拆箱
    System.out.println(b == d); // false，包装类自动装箱但是数值范围不在cache范围内
}
```

