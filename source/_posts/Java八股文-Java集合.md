---
title: Java八股文- Java集合
copyright: true
mathjax: false
categories:
  - Java八股文
date: 2023-01-23 10:06:52
tags:
toc: true
urlname: java-collection
---

## 集合容器概述

### 什么是集合

用于存储数据的容器。

### 集合和数组的区别

- 数组是固定长度的；集合可变长度的。
- 数组可以存储基本数据类型，也可以存储引用数据类型；集合只能存储引用数据类型。

### 常用的集合类有哪些？

Collection接口和Map接口是所有集合框架的父接口。

Collection接口：子接口包括Set接口和List接口

1. Set接口的实现类主要有：HashSet、TreeSet、LinkedHashSet等；
2. List接口的实现类主要有：ArrayList、LinkedList、Stack以及Vector等。

Map接口：实现类主要有HashMap、TreeMap、HashTable、ConcurrentHashMap以及Properties等。

### List、Set、Map三者的区别？

![Collection架构](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL290aGVyLzE0MDgxODMvMjAxOTExLzE0MDgxODMtMjAxOTExMTkxODQxNDk1NTktMTU3MTU5NTY2OC5qcGc.png)

List和Set两大接口是Collection的子接口

- List：有序容器，元素可以重复，可以插入多个null元素，元素都有索引；
	常用实现类：ArrayList、LinkedList 和 Vector。
	
- Set：无序容器，不可以存储重复元素，只允许存入一个null元素，必须保证元素唯一性；

	常用实现类：HashSet、LinkedHashSet 以及 TreeSet。

* Map是一个键值对集合，存储键、值和之间的映射。 key无序，唯一；value 不要求有序，允许重复；

	常用实现类：HashMap、TreeMap、HashTable、LinkedHashMap、ConcurrentHashMap。

### 集合框架底层数据结构

#### Collection

1. List

- Arraylist： Object数组
- Vector： Object数组
- LinkedList： 双向链表

2. Set

- HashSet（无序，唯一）：基于 HashMap 实现的，底层采用 HashMap 来保存元素
- LinkedHashSet： LinkedHashSet 继承于 HashSet，并且其内部是通过 LinkedHashMap 来实现的。
- TreeSet（有序，唯一）： 红黑树(自平衡的排序二叉树)

#### Map

- HashMap： JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8），且数组长度大于 64，将链表转化为红黑树，以减少搜索时间，否则扩容数组。
- LinkedHashMap：LinkedHashMap 继承自 HashMap，所以它的底层由数组和链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
- HashTable： 数组+链表组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的
- TreeMap： 红黑树（自平衡的排序二叉树）

### 哪些集合类是线程安全的？

vector 和 hashtable，通过在方法上加 synchronized 关键字实现线程安全

### Java集合的快速失败机制 “fail-fast”？

fail-fast是Java集合的一种**错误检测机制**，当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。

例如：假设存在两个线程（线程1、线程2），线程1通过Iterator在遍历集合A中的元素，在某个时候线程2修改了集合A的结构（add、remove、clear等结构上的修改，而不是简单的修改集合元素的内容），那么这个时候程序就会抛出 **ConcurrentModificationException** 异常，从而产生fail-fast事件。

原因：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。集合在被遍历期间如果内容发生变化，就会改变modCount的值，从而modCount不等于expectedmodCount值抛出异常，也就是产生了fail-fast事件。

解决办法：

1. 在遍历过程中，所有涉及到改变modCount值的地方全部加上synchronized；
2. 使用Java.util.concurrent下的CopyOnWriteArrayList来替换ArrayList；
3. 使用Java.util.concurrent下的ConcurrentHashMap来替换HashMap。

### 怎么确保一个集合不能被修改？

可以使用 Collections. unmodifiableCollection(Collection c) 方法来创建一个只读集合，这样改变集合的任何操作都会抛出 Java. lang. UnsupportedOperationException 异常。

```Java
List<String> list = new ArrayList<>();
list. add("x");
Collection<String> clist = Collections. unmodifiableCollection(list);
clist.add("y"); // 运行时此行报错
System.out.println(list. size());
```

## Collection接口

### List接口

#### 迭代器 Iterator 是什么？

Collection接口实现了Iterable接口，通过实现Iterable接口中的iterator()方法返回Iterator接口的实例，通过iterator对集合的元素进行迭代操作。

#### Iterator 怎么使用？有什么特点？

```Java
List<String> list = new ArrayList<>();
Iterator<String> it = list.iterator();
while(it.hasNext()){
    String obj = it.next();
    System.out.println(obj);
}
```

Iterator 的特点是只能单向遍历，但是更加安全，因为它可以确保，在当前遍历的集合元素被更改的时候，就会抛出 ConcurrentModificationException 异常。

#### 如何边遍历边移除 Collection 中的元素？

边遍历边修改 Collection 的唯一正确方式是使用 Iterator.remove() 方法，如下：

```java
Iterator<Integer> it = list.iterator();
while(it.hasNext()){
   *// do something*
   it.remove();
}
```

一种最常见的**错误**代码如下：

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
list.add("5");
int cnt = 0;
for (String item : list) {
  System.out.println(++cnt);
  list.remove(item);
}
System.out.println(Arrays.toString(list.toArray()));
```

运行以上错误代码会报 **ConcurrentModificationException 异常**。
这是因为当使用 foreach(for(Integer i : list)) 语句时，会自动生成一个iterator 来遍历该 list，但同时该 list 正在被 Iterator.remove() 修改。集合发生结构性变化，根据fail-fast机制，会抛异常。

#### 遍历一个 List 有哪些不同的方式？

1. for 循环，维护一个计数器；
2. 迭代器，Iterator。Collection接口实现了Iterable接口，通过实现Iterable接口中的iterator()方法返回Iterator接口的实例，通过iterator对集合的元素进行迭代操作；
3. 增强for循环。增强for循环foreach 内部也是采用了 Iterator 的方式实现，使用时不需要显式声明 Iterator 或计数器。
	优点是代码简洁，不易出错；缺点是只能做简单的遍历，不能在遍历过程中操作数据集合，例如删除、替换。

#### 说一下 ArrayList 的优缺点

**ArrayList 比较适合顺序添加、随机访问的场景。**

ArrayList的优点如下：

- ArrayList 底层以数组实现，是一种随机访问模式。ArrayList 实现了 RandomAccess 接口，标记 List 实现是否支持 Random Access，导致它可以随机访问的根本原因是它底层是数组结构，数组可以通过下标获取相应的值，而 RandomAccess 只是一个标记。因此查找的时候非常快。可以用 for 循环通过下标获取元素。
- ArrayList 在顺序添加一个元素的时候非常方便。

ArrayList 的缺点如下：

- 向**中间某个位置删除**元素的时候，需要做元素复制操作。如果要复制的元素很多，那么就会比较耗费性能；
- 向**中间某个位置插入**元素的时候，也需要做元素复制操作，缺点同上。

#### 如何实现数组和 List 之间的转换？

- 数组转 List：使用 Arrays. asList(array) 进行转换。
- List 转数组：使用 List 自带的 toArray() 方法。

```Java
// list to array
List<String> list = new ArrayList<String>();
list.add("123");
list.add("456");
list.toArray(new String[0]);

// array to list
String[] array = new String[]{"123", "456"};
Arrays.asList(array);
```

#### ArrayList 和 LinkedList 的区别是什么？

- 底层实现：ArrayList 是动态数组的数据结构实现，而 LinkedList 是双向链表的数据结构实现；
- 访问和增删元素效率：ArrayList 随机访问和顺序插入/删除元素效率更高；
- 内存占用：LinkedList 比 ArrayList 更占内存，因为 LinkedList 的节点除了存储数据，还存储了两个引用，一个指向前一个元素，一个指向后一个元素；
- 线程安全：ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全。

综合来说，在需要**频繁读取**集合中的元素时，更推荐使用 ArrayList，而在**插入和删除操作较多**时，更推荐使用 LinkedList。

#### ArrayList 和 Vector 的区别是什么？

主要区别是线程安全问题：
Vector 使用了 synchronized 来实现线程同步（性能差），是线程安全的，而 ArrayList 是非线程安全的。

#### 多线程场景下如何使用 ArrayList？

可以用 Java.util.concurrent.CopyOnWriteArrayList 替换 ArrayList。

#### 为什么 ArrayList 的 elementData 加上 transient 修饰？

ArrayList 实现了 Serializable 接口，这意味着 ArrayList 支持序列化。
对于 elementData，由于 elementData 可能有多余的容量（容量不足会扩容成1.5倍大小，扩容之后除了新加
的元素以外还有多的空间没有放元素，比如现在容量是15，而集合里面只存了11个元素），不对 elementData 加上 transient 修饰，而是手动进行序列化，可以保证只序列化实际存储的那些元素（11个元素），而不是整个数组，从而节省空间和时间。

### ⭐ArrayList扩容机制

> **总结：**
>
> * 初始化如果使用无参构造器的话：
>
> 	在第一次添加元素的时候，将数组容量设置为10，然后进行添加。
>
> 	一直添加，直到要添加的元素超出当前数组容量了，进入扩容，将数组长度扩容为1.5倍长度，然后进行添加。
>
> * 初始化如果使用有参构造器的话：
>
> 	那么每次添加元素只需要判断是否需要扩容即可，不需要第一次的初始化为10的判断。

#### 1. 先从 ArrayList 的构造函数说起

**（JDK8）ArrayList 有三种方式来初始化，构造方法源码如下：**

```java
    /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;


    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     *默认构造函数，使用初始容量10构造一个空列表(无参数构造)
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 带初始容量参数的构造函数。（用户自己指定容量）
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {//初始容量大于0
            //创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {//初始容量等于0
            //创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {//初始容量小于0，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }


   /**
    *构造包含指定collection元素的列表，这些元素利用该集合的迭代器按顺序返回
    *如果指定的集合为null，throws NullPointerException。
    */
     public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

**以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为 10。** 

> 补充：JDK6 new 无参构造的 `ArrayList` 对象时，直接创建了长度是 10 的 `Object[]` 数组 elementData 。

#### 2. 一步一步分析 ArrayList 扩容机制

这里以无参构造函数创建的 ArrayList 为例分析

##### 2.1.  `add` 

```java
    /**
     * 将指定的元素追加到此列表的末尾。
     */
    public boolean add(E e) {
   // 添加元素之前，先调用ensureCapacityInternal方法
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e;
        return true;
    }
```

##### 2.2. `ensureCapacityInternal()` 

（JDK7）可以看到 `add` 方法 首先调用了`ensureCapacityInternal(size + 1)`

```java
   //得到最小扩容量
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            // 获取默认的容量和传入参数的较大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
```

**当 要 add 进第 1 个元素时，minCapacity 为 1，在 Math.max()方法比较后，minCapacity 为 10。**

> 此处和后续 JDK8 代码格式化略有不同，核心代码基本一样。

##### 2.3. `ensureExplicitCapacity()` 

如果调用 `ensureCapacityInternal()` 方法就一定会进入（执行）这个方法

```java
  //判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //调用grow方法进行扩容，调用此方法代表已经开始扩容了
            grow(minCapacity);
    }
```

我们来仔细分析一下：

- 当我们要 add 进第 1 个元素到 ArrayList 时，elementData.length 为 0 （因为还是一个空的 list），因为执行了 `ensureCapacityInternal()` 方法 ，所以 minCapacity 此时为 10。此时，`minCapacity - elementData.length > 0`成立，所以会进入 `grow(minCapacity)` 方法。
- 当 add 第 2 个元素时，minCapacity 为 2，此时 elementData.length(容量)在添加第一个元素后扩容成 10 了。此时，`minCapacity - elementData.length > 0` 不成立，所以不会进入 （执行）`grow(minCapacity)` 方法。
- 添加第 3、4···到第 10 个元素时，依然不会执行 grow 方法，数组容量都为 10。

直到添加第 11 个元素，minCapacity(为 11)比 elementData.length（为 10）要大。进入 grow 方法进行扩容。

##### 2.4. `grow()` 

```java
    /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        // oldCapacity为旧容量，newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
        //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
       // 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，
       //如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

**int newCapacity = oldCapacity + (oldCapacity >> 1),所以 ArrayList 每次扩容之后容量都会变为原来的 1.5 倍左右（oldCapacity 为偶数就是 1.5 倍，否则是 1.5 倍左右）** 

**我们再来通过例子探究一下grow() 方法 ：**

- 当 add 第 1 个元素时，oldCapacity 为 0，经比较后第一个 if 判断成立，newCapacity = minCapacity(为 10)。但是第二个 if 判断不会成立，即 newCapacity 不比 MAX_ARRAY_SIZE 大，则不会进入 `hugeCapacity` 方法。数组容量为 10，add 方法中 return true,size 增为 1。
- 当 add 第 11 个元素进入 grow 方法时，newCapacity 为 15，比 minCapacity（为 11）大，第一个 if 判断不成立。新容量没有大于数组最大 size，不会进入 hugeCapacity 方法。数组容量扩为 15，add 方法中 return true,size 增为 11。
- 以此类推······

##### 2.5. `hugeCapacity()` 

从上面 `grow()` 方法源码我们知道： 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，如果 minCapacity 大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。

```java
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        //对minCapacity和MAX_ARRAY_SIZE进行比较
        //若minCapacity大，将Integer.MAX_VALUE作为新数组的大小
        //若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组的大小
        //MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

### Set接口

#### HashSet 的实现原理？

HashSet 是基于 HashMap 实现的，HashSet的值存放于HashMap的key上，HashMap的value统一为Object常量PRESENT，由于 HashMap 的 key 不允许重复，而 HashSet 不允许重复的值，因此 HashSet 可以基于 HashMap 实现，基本上都是直接调用底层 HashMap 的相关方法来完成。

#### HashSet是如何保证数据不可重复的？

当你把对象加入 hashSet 时，hashSet 会先计算对象的 hashcode 值来判断对象加入的位置，如果发现有相同 hashcode 值的对象，这时会调用 equals()方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，表明已经有了相同的对象了，hashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。 

```java
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();
private transient HashMap<E,Object> map;

public HashSet() {
    map = new HashMap<>();
}

public boolean add(E e) {
    // 调用HashMap的put方法,PRESENT是一个至始至终都相同的虚值
	return map.put(e, PRESENT)==null;
}
```

#### 只重写 hashCode 可以吗？

两个对象的 hashCode 相同，但是内容可能不同，在 set 中明明应该是两个不同的对象，这时候只通过 hashCode 判断他俩相同会导致只插入一个元素。

#### 只重写 equals可以吗？

如果在重写 equals 时，不重写 hashCode，就会引起比如说将两个相等的自定义对象存储在 Set 集合时，会将这两个对象都进行插入。（无法去重）

## Map接口

### HashMap线程不安全的主要体现？

1. 在jdk1.7中，在多线程环境下，**扩容**时会造成死链。

	> 作者：磊哥
	> 链接：https://www.zhihu.com/question/394039290/answer/2314917909
	> 来源：知乎

	1.1、

	死循环是因为并发HashMap扩容导致的，并发扩容的第一步，线程T1和线程T2要对HashMap进行扩容操作，此时T1和T2指向的是链表的头结点元素A，而T1和T2的下一个节点，也就是T1.next和T2.next指向的是B节点，如下图所示： 

	![多线程扩容导致死循环问题-1](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/v2-e0b4f5ea00cb277bde490f315b9b387f_720w.jpg)

	1.2、

	死循环的第二步操作是，线程T2时间片用完进入休眠状态，而线程T1开始执行扩容操作，一直到线程T1扩容完成后，线程T2才被唤醒，扩容之后的场景如下图所示： 

	![多线程扩容导致死循环问题-2](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/v2-5f78d8ac8af1efc79e7d79a1f3fdc154_r.jpg)

	 从上图可知线程T1执行之后，**因为是头插法，所以HashMap的顺序已经发生了改变**，但线程T2对于发生的一切是不可知的，所以它的指向元素依然没变，如上图展示的那样，T2指向的是A元素，T2.next指向的节点是B元素。

	1.3、

	当线程T1执行完，而线程T2恢复执行时，死循环就建立了，如下图所示： 

	![img](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/v2-bd4a753db8c985119aa4dcd4356609cf_720w.jpg)

	 因为T1执行完扩容之后B节点的下一个节点是A，而T2线程指向的首节点是A，第二个节点是B，这个顺序刚好和T1扩完容完之后的节点顺序是相反的。**T1执行完之后的顺序是B到A，而T2的顺序是A到B，这样A节点和B节点就形成死循环了**，这就是HashMap死循环导致的原因。

2. 在jdk1.8中，在多线程环境下，**插入**时会发生数据覆盖的情况（**尾插法**）。

### HashMap在JDK1.7和JDK1.8中有哪些不同？

| 不同                         | JDK 1.7                                                      | JDK 1.8                                                      |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **存储结构**                 | 数组 + 链表                                                  | 数组 + 链表 + 红黑树                                         |
| **hash值计算方式**           | 扰动处理 = 4次位运算 + 5次异或运算                           | 扰动处理 = 1次位运算 + 1次异或运算                           |
| **存放数据的规则**           | 无冲突时，存放数组；<br />冲突时，存放链表                   | 无冲突时，存放数组；<br />冲突 & 链表长度 < 8：插入单链表；<br />链表长度>8，先尝试扩容数组，数组长度大于64<br />链表转为红黑树，插入到红黑树 |
| **插入数据方式**             | 头插法                                                       | 尾插法                                                       |
| **扩容后存储位置的计算方式** | 按照原来方法重新进行计算<br />hashCode、扰动函数、与运算h&length-1 | 根据元素的hash值进行判断 <br />扩容后的位置=原位置 or 原位置 + 旧容量 |

### HashMap的hash算法

#### 什么是哈希？

Hash，就是把任意长度的输入通过散列算法，变换成固定长度的输出，该输出就是散列值（哈希值）；

hash 值不同，输入一定不同；输入不同，hash 值可能相同（即哈希碰撞）

#### 什么是哈希冲突？

当两个不同的输入值，根据同一散列函数计算出相同的散列值的现象，我们就把它叫做哈希碰撞/冲突。

#### HashMap的数据结构

底层数据结构：数组+链表
**数组的特点是：方便查找；链表的特点是：方便插入删除**
所以我们将数组和链表结合在一起，发挥两者各自的优势，使用一种叫做**链地址法**的方式可以解决哈希冲突

> 解决哈希冲突的其他方法：
> 开放定址法（冲突了通过某种方式散列到另一个不冲突的位置上）；
> 再哈希法（构造多个不同的哈希函数）。

![HashMap数据结构](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy83ODk2ODkwLWM2NmE0YmQ4OTA3ZjQ5ZjYucG5n.png)

#### 扰动处理

由于计算下标时，通过 h & (length - 1)，由于hash值范围很大，而数组长度相对来说小得多，导致进行与运算时参与运算的只有hash值的低位，**高位是没有起到任何作用的**，这将会大大增加哈希碰撞的概率。所以我们的思路就是让hashCode值的高位也参与运算，进一步降低hash碰撞的概率，使得**数据分布更平均**，我们把这样的操作称为**扰动**，在**JDK 1.8**中的hash()函数如下：

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    // 与自己右移16位进行异或运算（高低位异或）
}
```

这比在**JDK 1.7**中，更为简洁，**相比在1.7中的4次位运算，5次异或运算（9次扰动），在1.8中，只进行了1次位运算和1次异或运算（2次扰动）**；

#### 下标计算

在得到hash值之后，我们要把这些hash值映射到相应的数组下标上，那么就可以考虑使用 hash % length

而 hash % length==hash & (length-1) 并且前提是 length 是2的 n 次方。
由于采用二进制位操作 &，相对于%能够提高运算效率，因此可以将 HashMap 的长度设计为 2 的幂次方。（初始化为16，如果指定初始化容量的值，会扩容至第一个大于该容量的 2 的幂次方作为数组容量）

```java
static int indexFor(int h, int length){
     return h & (length - 1);
}
```

#### 为什么要将数组长度设置为 2 的n次幂呢？

当数组长度为2的n次幂的时候，不同的key与1111（n个）进行与预算得到相同下标的几率较小，发生碰撞的几率小。 

#### JDK1.8新增红黑树

为了解决一条链上的元素过多，查找效率低的问题，将链表转化为红黑树，遍历复杂度降低至O(logn)；

#### HashMap是使用了哪些方法来有效解决哈希冲突的？

1. 使用链地址法（使用散列表）来链接拥有相同hash值的数据；
2. 使用2次扰动函数（1次位运算和1次异或运算）来降低哈希冲突的概率，使得数据分布更平均；
3. 引入红黑树进一步降低遍历的时间复杂度，使得遍历更快。

### HashMap的get方法的具体流程？

1. 先使用key的hashCode、扰动处理、与数组长度-1 进行与运算（对数组长度去余）得到槽位；
2. 判断首结点，如果首结点和key的 **hash值** （hashCode值经过扰动函数处理后的值）相等，并且两者的 **key值地址相等、equals相等**，则返回首结点；
3. 否则，则红黑树或者链表中进行查找，查找时判断两个key的hash值、key值地址相等、equals是否都相等。

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    
    //Node数组不为空，数组长度大于0，数组对应下标的Node不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        //也是通过 hash & (length - 1) 来替代 hash % length 的
        (first = tab[(n - 1) & hash]) != null) {
        
        //先和第一个结点比，hash值相等且key不为空，key的第一个结点的key的对象地址和值均相等
        //则返回第一个结点
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //如果key和第一个结点不匹配，则看.next是否为空，不为null则继续，为空则返回null
        if ((e = first.next) != null) {
            //如果此时是红黑树的结构，则进行处理getTreeNode()方法搜索key
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            //是链表结构的话就一个一个遍历，直到找到key对应的结点，
            //或者e的下一个结点为null退出循环
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### HashMap的put方法的具体流程？

1. 如果table未初始化或者长度为0空则执行resize()进行扩容，转向②
2. 根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，即数组该位置上没有元素，直接新建节点插入，转向⑥判断是否需要扩容，如果table[i]不为空，转向③；
3. 判断table[i]的首个元素是否和key一样，如果相同（hashCode相同、key地址相同、equals相同）直接覆盖value，否则转向④；
4. 判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向⑤；
5. 遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；
6. 插入成功后，判断实际存在的键值对数量size是否超出当前最大容量threshold，如果超过，进行扩容。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

//实现Map.put和相关方法
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 步骤①：tab为空则创建 
    // table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 步骤②：计算index，并对null做处理  
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素
    else {
        Node<K,V> e; K k;
        // 步骤③：节点key存在，直接覆盖value 
        // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
        // 步骤④：判断该链为红黑树 
        // hash值不相等，即key不相等；为红黑树结点
        // 如果当前元素类型为TreeNode，表示为红黑树，putTreeVal返回待存放的node, e可能为null
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 步骤⑤：该链为链表 
        // 为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                
                //判断该链表尾部指针是不是空的
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    //判断链表的长度是否达到转化红黑树的临界值，临界值为8
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        //链表结构转树形结构
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        //判断当前的key已经存在的情况下，再来一个相同的hash值、key值时，返回新来的value这个值
        if (e != null) { 
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 步骤⑥：超过最大容量就扩容 
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

### HashMap的扩容操作是怎么实现的？

①.在jdk1.8中，resize方法是在hashmap中的键值对（元素数量）大于阀值时（阈值为容量*负载因子）扩容为原容量的2倍，或者初始化时（扩容为16），就调用resize方法进行扩容；

②.在1.7中，扩容之后需要重新去计算其Hash值，根据Hash值对其进行分发，但在1.8版本中，则是根据在同一个桶的位置中进行判断(e.hash & oldCap)是否为0，重新进行hash分配后，该元素hash值与旧容量与运算为0则留在原始位置，hash值与旧容量与运算为1则移动到 原位置 + oldCap 这个位置上

![HashMap的resize操作](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes20190128152700351.png)

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;//oldTab指向hash桶数组
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {//如果oldCap不为空的话，就是hash桶数组不为空
        if (oldCap >= MAXIMUM_CAPACITY) {//如果大于最大容量了，就赋值为整数最大的阀值
            threshold = Integer.MAX_VALUE;
            return oldTab;//返回
        }//如果当前hash桶数组的长度在扩容后仍然小于最大容量 并且oldCap大于默认值16
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold 双倍扩容阀值threshold
    }
    // 旧的容量为0，但threshold大于零，代表有参构造有cap传入，threshold已经被初始化成最小2的n次幂
    // 直接将该值赋给新的容量
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    // 无参构造创建的map，给出默认容量和threshold 16, 16*0.75
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 新的threshold = 新的cap * 0.75
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    // 计算出新的数组长度后赋给当前成员变量table
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];//新建hash桶数组
    table = newTab;//将新数组的值复制给旧的hash桶数组
    // 如果原先的数组没有初始化，那么resize的初始化工作到此结束，否则进入扩容元素重排逻辑，使其均匀的分散
    if (oldTab != null) {
        // 遍历新数组的所有桶下标
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                // 旧数组的桶下标赋给临时变量e，并且解除旧数组中的引用，否则就数组无法被GC回收
                oldTab[j] = null;
                // 如果e.next==null，代表桶中就一个元素，不存在链表或者红黑树
                if (e.next == null)
                    // 用同样的hash映射算法把该元素加入新的数组
                    newTab[e.hash & (newCap - 1)] = e;
                // 如果e是TreeNode并且e.next!=null，那么处理树中元素的重排
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // e是链表的头并且e.next!=null，那么处理链表中元素重排
                else { // preserve order
                    // loHead,loTail 代表扩容后不用变换下标，见注1
                    Node<K,V> loHead = null, loTail = null;
                    // hiHead,hiTail 代表扩容后变换下标，见注1
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 遍历链表
                    do {             
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                // 初始化head指向链表当前元素e，e不一定是链表的第一个元素，初始化后loHead
                                // 代表下标保持不变的链表的头元素
                                loHead = e;
                            else                                
                                // loTail.next指向当前e
                                loTail.next = e;
                            // loTail指向当前的元素e
                            // 初始化后，loTail和loHead指向相同的内存，所以当loTail.next指向下一个元素时，
                            // 底层数组中的元素的next引用也相应发生变化，造成lowHead.next.next.....
                            // 跟随loTail同步，使得lowHead可以链接到所有属于该链表的元素。
                            loTail = e;                           
                        }
                        else {
                            if (hiTail == null)
                                // 初始化head指向链表当前元素e, 初始化后hiHead代表下标更改的链表头元素
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 遍历结束, 将tail指向null，并把链表头放入新数组的相应下标，形成新的映射。
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

### HashMap负载因子为什么设置为 0.75?

hashmap的源码中有一段注释解释了为什么设置为0.75，是**空间和时间效率的折中考虑**。

### 能否使用任何类作为 Map 的 key？

可以使用任何类作为 Map 的 key，前提是该类重写了hashCode和equals方法。重写hashCode() 是为了计算数据的存储位置，重写equals方法是为了确保key在哈希表中的唯一性。最终目的是为了保证 Map 中键key的唯一性。

### 为什么HashMap中String、Integer这样的包装类适合作为key？

重写了hashCode和equals方法，保证了 Map 中键**key的唯一性**。并且他们都不可变，可以将它们的hash值**缓存下来，提高效率**。

### HashMap 与 HashTable 有什么区别？

1. **线程安全**： HashMap 是非线程安全的，HashTable 是线程安全的，内部的方法基本都经过 `synchronized` 修饰；
2. **效率**： 因为线程安全的问题 synchronized，HashMap 要比 HashTable 效率高一点；
3. **对Null key 和Null value的支持**： HashMap 中，null 可以作为键，这样的键只有一个，可以有一个或多个键所对应的值为 null。但是在 HashTable 中 put 进的键值只要有一个 null，直接抛NullPointerException；
4. **初始容量大小和每次扩充容量大小的不同 **： 
	①创建时如果不指定容量初始值，HashTable 默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap 默认的初始化大小为16。之后每次扩充，容量变为原来的2倍；
	②创建时如果给定了容量初始值，那么 HashTable 会直接使用你给定的大小，而 HashMap 会将其扩充为2的幂次方大小。也就是说 HashMap 总是使用2的幂作为哈希表的大小；
5. **底层数据结构**： JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。HashTable 没有这样的机制。

### ConcurrentHashMap 和 HashTable 的区别？

ConcurrentHashMap 和 HashTable 的区别主要体现在实现线程安全的方式上不同。

- **底层数据结构**： 
	JDK1.7的 ConcurrentHashMap 底层采用 **分段的数组+链表** 实现，JDK1.8 采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑二叉树。
	HashTable 和 JDK1.8 之前的 HashMap 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；
- **实现线程安全的方式（重要）**： 
	① **在JDK1.7的时候，ConcurrentHashMap使用分段锁** 对整个桶数组进行了分割分段(Segment锁继承ReentrantLock)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。（默认分配16个Segment，比HashTable效率提高16倍。）
	**到了 JDK1.8 的时候已经摒弃了Segment的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作，synchronized只锁定当前链表或红黑树的首节点。（JDK1.6以后 对 synchronized锁做了很多优化）** 整个看起来就像是优化过且线程安全的 HashMap
	② **HashTable(全表锁)** :使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。
- ⭐**Jdk1.7采用分段锁，锁粒度较大，容易发生冲突，并发量低；因此 Jdk1.8 采用只锁定桶的首结点，锁的力度较低，不容易发生冲突，并发量高。**

**两者的对比图**：

HashTable:

![HashTable全表锁](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOC84LzIzLzE2NTY2NzdhNmYyNTYxOTY.png)

JDK1.7的ConcurrentHashMap：

![JDK1.7的ConcurrentHashMap分段锁](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOC84LzIzLzE2NTY2NzdhNmYwMDhiZTQ.png)

JDK1.8的ConcurrentHashMap（TreeBin: 红黑二叉树节点 Node: 链表节点）：

![JDK1.8的ConcurrentHashMap分段锁](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOC84LzIzLzE2NTY2NzdhNmYxNGUwMzk.png)
