---
title: "Java八股文 - Java异常"
date: 2023-01-23 10:06:31
categories: ["Java八股文"]
tags: ["异常"]
url: java-exception-error

################################目录################################
toc: true
autoCollapseToc: false
################################公式渲染################################
mathjax: false

################################基本不动################################
# lastmod: {{ .Date }}
draft: false
# keywords: []
# description: ""
author: "Yaxing"

comment: false
postMetaInFooter: false
hiddenFromHomePage: false
contentCopyright: true
---

> 整理的Java异常相关知识点和面试题，部分内容摘自网络，如有侵权请联系我～	<!--more-->

## Java异常架构

![Java异常架构](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes20200314173417278.png)

### 1. Throwable

Throwable 是 Java 语言中所有错误与异常的超类。

Throwable 包含两个子类：Error（错误）和 Exception（异常），它们通常用于指示发生了异常情况。

### 2. Error（错误）

Error 类及其子类。程序中无法处理的错误，表示运行应用程序中出现了严重的错误。

非受检异常，一般表示代码运行时 JVM 出现问题。比如 OutOfMemoryError：内存不足错误；StackOverflowError：栈溢出错误。此类错误发生时，JVM 将终止线程。

### 3. Exception（异常）

程序本身可以捕获并且可以处理的异常。Exception 这种异常又分为两类：运行时异常和编译时异常。

#### 运行时异常

Exception 中 RuntimeException 类及其子类，表示 JVM 在运行期间可能出现的异常。（非受检异常，编译时不会检查出异常，运行时会发生异常，需要对异常进行特殊处理，否则会发生逻辑错误）

#### 编译时异常

Exception 中除 RuntimeException 及其子类之外的异常。（受检异常，编译时会检查到该异常 ）

## Java异常关键字

• **try** – 用于监听。将要被监听的代（可能抛出异常的代码）放在 try 语句块之内，当 try 语句块内发生异常时，异常就被抛出。
• **catch** – 用于捕获异常。catch 用来捕获 try 语句块中发生的异常。
• **finally** – finally 语句块总是会被执行。它主要用于回收在 try 块里打开的资源(如数据库连接、网络连接和磁盘文件)。
• **throw** – 用于手动抛出 catch 到的异常。
• **throws** – 用在方法签名中，用于声明该方法可能抛出的异常，调用该方法的方法需要注意 catch 这个可能跑出的异常。

## Java异常处理

### 直接抛出异常

通常，应该捕获那些知道如何处理的异常，将不知道如何处理的异常继续传递下去。传递异常可以在方法签名处使用 **throws** 关键字声明可能会抛出的异常。调用该方法的方法需要注意 catch 这个可能抛出的异常。

```java
private static void readFile(String filePath) throws IOException {
    File file = new File(filePath);
    String result;
    BufferedReader reader = new BufferedReader(new FileReader(file));
    while((result = reader.readLine())!=null) {
        System.out.println(result);
    }
    reader.close();
}
```

### 封装异常再抛出

有时我们会从 catch 中抛出一个异常，目的是为了改变异常的类型。多用于在多系统集成时，当某个子系统故障，异常类型可能有多种，可以用统一的异常类型向外暴露，不需暴露太多内部异常细节。

```java
private static void readFile(String filePath) throws MyException {    
    try {
        // code
    } catch (IOException e) {
        MyException ex = new MyException("read file failed.");
        ex.initCause(e);
        throw ex;
    }
}
```

### 捕获异常

在一个 try-catch 语句块中可以捕获多个异常类型，并对不同类型的异常做出不同的处理

```java
private static void readFile(String filePath) {
    try {
        // code
    } catch (FileNotFoundException e) {
        // handle FileNotFoundException
    } catch (IOException e){
        // handle IOException
    }
}
```

同一个 catch 也可以捕获多种类型异常，用 | 隔开

```java
private static void readFile(String filePath) {
    try {
        // code
    } catch (FileNotFoundException | UnknownHostException e) {
        // handle FileNotFoundException or UnknownHostException
    } catch (IOException e){
        // handle IOException
    }
}
```

### 自定义异常

习惯上，定义一个异常类应包含两个构造函数，一个无参构造函数和一个带有详细描述信息的构造函数（Throwable 的 toString 方法会打印这些详细信息，调试时很有用）

```java
public class MyException extends Exception {
    public MyException(){ }
    public MyException(String msg){
        super(msg);
    }
    // ...
}
```

## Java异常常见面试题

### 1. Error 和 Exception 区别是什么？

Error 类型的错误通常为虚拟机相关错误，如系统崩溃，内存不足，堆栈溢出等，编译器不会对这类错误进行检测，Java 应用程序也不应对这类错误进行捕获，一旦这类错误发生，通常应用程序会被终止，仅靠应用程序本身无法恢复；

Exception 类的错误是可以在应用程序中进行捕获并处理的，通常遇到这种错误，应对其进行处理，使应用程序可以继续正常运行。

### 2. 运行时异常和一般异常(受检异常)区别是什么？

运行时异常包括 RuntimeException 类及其子类，表示 JVM 在运行期间可能出现的异常。 Java 编译器不会检查运行时异常。

受检异常是 Exception 中除 RuntimeException 及其子类之外的异常。 Java 编译器会检查受检异常。

**RuntimeException异常和受检异常之间的区别**：是否强制要求调用者必须处理此异常，如果强制要求调用者必须进行处理，那么就使用受检异常，否则就选择非受检异常(RuntimeException)。一般来讲，如果没有特殊的要求，我们建议使用 RuntimeException 异常。

### 3. JVM 是如何处理异常的？

在一个方法中如果发生异常，这个方法会创建一个异常对象，并转交给 JVM，该异常对象包含异常名称，异常描述以及异常发生时应用程序的状态。创建异常对象并转交给 JVM 的过程称为抛出异常。可能有一系列的方法调用，最终才进入抛出异常的方法，这一系列方法调用的有序列表叫做调用栈。

JVM 会顺着调用栈去查找看是否有可以处理异常的代码，如果有，则调用异常处理代码。当 JVM 发现可以处理异常的代码时，会把发生的异常传递给它。如果 JVM 没有找到可以处理该异常的代码块，JVM 就会将该异常转交给默认的异常处理器（默认处理器为 JVM 的一部分），默认异常处理器打印出异常信息并终止应用程序。

### 4. throw 和 throws 的区别是什么？

Java 中的异常处理除了包括捕获异常和处理异常之外，还包括声明异常和拋出异常，可以通过 throws 关键字在方法上声明该方法要拋出的异常，或者在方法内部通过 throw 拋出异常对象。

**throws 关键字和 throw 关键字在使用上的几点区别如下**：

- throw 关键字用在方法内部，只能用于抛出一种异常，用来抛出方法或代码块中的异常，受查异常和非受查异常都可以被抛出。
- throws 关键字用在方法声明上，可以抛出多个异常，用来标识该方法可能抛出的异常列表。一个方法用 throws 标识了可能抛出的异常列表，调用该方法的方法中必须包含可处理异常的代码，否则也要在方法签名中用 throws 关键字声明相应的异常。

### 5. try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？

**答：会执行，在 return 前执行。**

**代码示例1：**

```java
public static int getInt() {
    int a = 10;
    try {
        System.out.println(a / 0);
        a = 20;
    } catch (ArithmeticException e) {
        a = 30;
        return a;	// 不会直接return，先执行finally，finally执行过后回来执行return a
    } finally {
        a = 40;
    }
	return a;
}
```

执行结果：30

**代码示例2：**

```java
public static int getInt() {
    int a = 10;
    try {
        System.out.println(a / 0);
        a = 20;
    } catch (ArithmeticException e) {
        a = 30;
        return a;	// 不会直接return，先执行finally
    } finally {
        a = 40;
        return a;	// finally中有return就直接return了
    }
}
```

执行结果：40

### 6. 类 ExampleA 继承 Exception，类 ExampleB 继承ExampleA。

```java
public class ExceptionA extends Exception {
    public ExceptionA(String message) {
        super(message);
    }
}
public class ExceptionB extends ExceptionA {
    public ExceptionB(String message) {
        super(message);
    }
}

try {
  throw new ExceptionB("a");
} catch (ExceptionA ea) {
  System.out.println("ExceptionA");
} catch (Exception e) {
  System.out.println("Exception");
}
```

输出：ExampleA。（根据里氏代换原则[能使用父类型的地方一定能使用子类型]，抓取 ExceptionA 类型异常的 catch 块能够抓住 try 块中抛出的 ExceptionB 类型的异常）

```java
try {
 	throw new ExceptionB("b");
} catch (Exception e) {
  System.out.println("Exception: " + e);
}
```

输出：Exception: com.yaxing.demo.exceptiondemo.ExceptionB: b，仍会使用子类型。

面试题 - 说出下面代码的运行结果。（此题的出处是《Java 编程思想》一书）

```java
class Annoyance extends Exception {
}
class Sneeze extends Annoyance {
}
class Human {
  public static void main(String[] args) throws Exception {
    try {
      try {
        throw new Sneeze();
      } catch ( Annoyance a ) {
        System.out.println("Caught Annoyance");
        throw a;
      }
    } catch ( Sneeze s ) {
      System.out.println("Caught Sneeze");
      return ;
    } finally {
      System.out.println("Hello World!");
    }
  }
}
```

结果

```java
Caught Annoyance
Caught Sneeze
Hello World!
```

