---
title: Java八股文 - Redis
copyright: true
mathjax: false
categories:
  - Java八股文
date: 2023-01-25 13:43:59
tags:
toc: true
urlname: Redis
---

> 整理的 Redis 相关知识点和面试题，部分内容摘自网络，如有侵权请联系我～	<!--more-->

## 什么是NoSQL？

NoSQL = not only sql（不仅仅是 SQL）

关系型数据库：列+行，同一个表下数据的结构是一样的。

非关系型数据库：数据存储没有固定的格式，并且可以进行横向扩展。

NoSQL 泛指非关系型数据库，随着 web2.0 互联网的诞生，传统的关系型数据库很难对付 web2.0 大数据时代。尤其是超大规模的高并发的社区，暴露出来很多难以克服的问题，NoSQL 在当今大数据环境下发展的十分迅速，Redis 是发展最快的。

传统 RDBMS（关系数据库管理系统）和 NoSQL（非关系型数据库）比较：

RDBMS：
 - 组织化结构
 - 固定 SQL
 - 数据和关系都存在单独的表中（行列）
 - DML（数据操作语言）、DDL（数据定义语言）等
 - 严格的一致性（ACID）：原子性、一致性、隔离性、持久性
 - 基础的事务

NoSQL：
 - 不仅仅是数据
 - 没有固定查询语言
 - 键值对存储（Redis）、列存储（HBase）、文档存储（MongoDB）、图形数据库（不是存图形，放的是关系）（Neo4j）
 - 最终一致性（BASE）：基本可用、软状态/柔性事务、最终一致性

***

## Redis是什么？

### 定义

Redis = Remote Dictionary Server，即远程字典服务。

Redis 是一个用 C 语言开发的、基于内存结构进行**键值对**数据存储的、高性能的、非关系型 NoSQL 数据库

### 优缺点

**纯内存操作，性能非常出色**，每秒可以处理超过 10 万次读写操作，是已知性能最快的 Key-Value DB。支持保存多种数据结构，可以用来实现很多有用的功能。

Redis 的主要缺点是**数据库容量受到物理内存的限制，不能用作海量数据的高性能读写**，因此 Redis 适合的场景主要局限在较小数据量的高性能操作和运算上。

### 为什么要用缓存

**1、 高性能：**

假如用户第一次访问数据库中的某些数据。这个过程会比较慢，因为是从硬盘上读取的。将该用户访问的数据存在数缓存中，这样下一次再访问这些数据的时候就可以直接从缓存中获取了。操作缓存就是直接操作内存，所以速度相当快。如果数据库中的对应数据改变的之后，同步改变缓存中相应的数据即可。

**2、高并发：**

直接操作缓存能够承受的请求是远远大于直接访问数据库的，所以我们可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库，缓解数据库压力。

### Redis 与 Memcached 相比有哪些优势

Redis 支持更丰富的数据类型（支持更复杂的应用场景）

Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用，而 Memecache 把数据全部存在内存之中。

Redis 使用单线程的多路 IO 复用模型；Memcached 是多线程，非阻塞 IO 复用的网络模型。

***

## Redis五大基本类型

[Redis 命令大全](https://www.Redis.net.cn/order/)

[Redis 应用场景](https://juejin.cn/post/6857667542652190728)

### String（字符串）

> 1. String 类型是 Redis 的最基础的数据结构，也是最经常使用到的类型。而且其他的四种类型多多少少都是在字符串类型的基础上构建的，所以 String 类型是 Redis 的基础。
>
> 2. String 类型的值最大能存储 512MB，这里的 String 类型可以是简单字符串、  复杂的 xml/json 的字符串、二进制图像或者音频的字符串、以及可以是数字的字符串。

**应用场景**

1、缓存功能：String 字符串是最常用的数据类型，不仅仅是 Redis，各个语言都是最基本类型，因此，利用 Redis 作为缓存，配合其它数据库作为存储层，利用 Redis 支持高并发的特点，可以大大加快系统的读写速度、以及降低后端数据库的压力。

2、计数器：许多系统都会使用 Redis 作为系统的实时计数器，可以快速实现计数和查询的功能。而且最终的数据结果可以按照特定的时间落地到数据库或者其它存储介质当中进行永久保存。

3、共享用户 session：用户重新刷新一次界面，可能需要访问一下数据进行重新登录，或者访问页面缓存 cookie，这两种方式做有一定弊端，1）每次都重新登录效率低下 2）cookie 保存在客户端，有安全隐患。这时可以利用 Redis 将用户的 session 集中管理，在这种模式只需要保证 Redis 的高可用，每次用户 session 的更新和获取都可以快速完成。大大提高效率。（分布式会话）

### List（列表）

> 1. List 是简单的字符串列表
> 2. Redis 可以从列表的两端进行插入（pubsh）和弹出（pop）元素，支持读取指定范围的元素集，或者读取指定下标的元素等操作。Redis列表是一种比较灵活的链表数据结构，它可以充当队列或者栈的角色。

**应用场景**

1、消息队列：Redis 的链表结构，可以轻松实现阻塞队列，可以使用左进右出的命令组成来完成队列的设计。比如：数据的生产者可以通过 lpush 命令从左边插入数据，多个数据消费者，可以使用 rpop 命令阻塞的“抢”列表尾部的数据。

2、文章列表或者数据分页展示的应用。比如，我们常用的博客网站的文章列表，当用户量越来越多时，而且每一个用户都有自己的文章列表，而且当文章多时，都需要分页展示，这时可以考虑使用 Redis 的列表，列表不但有序同时还支持按照范围内获取元素，可以完美解决分页查询功能。大大提高查询效率。

### Set（集合）

> 1. Redis 集合 set 类型和 list 列表类型类似，都可以用来存储多个字符串元素的集合。
> 2. 但是和 list 不同的是 set 集合当中不允许重复的元素。而且 set 集合当中元素是没有顺序的，不存在元素下标。
> 3. Redis 的 set 类型是使用哈希表构造的，因此复杂度是 O(1)，它支持集合内的增删改查，并且支持多个集合间的交集、并集、差集操作。可以利用这些集合操作，解决程序开发过程当中很多数据集合间的问题。

**应用场景**

1、标签：比如我们博客网站常常使用到的兴趣标签，把一个个有着相同爱好，关注类似内容的用户利用一个标签把他们进行归并。

2、共同好友功能，共同喜好，或者可以引申到二度好友之类的扩展应用。

3、统计网站的独立 IP。利用 set 集合当中元素不唯一性，可以快速实时统计访问网站的独立 IP。

### sorted set（有序集合）

> Redis 有序集合也是集合类型的一部分，所以它保留了集合中元素不能重复的特性，但是不同的是，有序集合给每个元素多设置了一个分数，利用该分数作为排序的依据。

#### 应用场景

1、排行榜：有序集合经典使用场景。例如视频网站需要对用户上传的视频做排行榜，榜单维护可能是多方面：按照时间、按照播放量、按照获得的赞数等。

2、用 Sorted Sets 来做带权重的队列，比如普通消息的 score 为 1，重要消息的 score 为 2，然后工作线程可以选择按 score 的倒序来获取工作任务。让重要的任务优先执行。

### hash（哈希）

Redis hash 数据结构 是一个键值对（key-value）集合，它是一个 string 类型的 field 和 value 的映射表，Redis 本身就是一个 key-value 型数据库，因此 hash 数据结构相当于在 value 中又套了一层 key-value 型数据。所以 Redis 中 hash 数据结构特别适合存储关系型对象。

> 用户 ID 为查找的key，存储的 value 用户对象包含姓名，年龄，生日等信息，如果用普通的 key/value 结构来存储
>
> 1. 将用户 ID 作为 key，姓名属性+姓名数据整体作为 value。缺点是每次修改用户的某个属性，都需要先反序列化 属性+数据，修改好，然后将 属性+数据 序列化之后在修改回去，开销较大。
>
> ![ID -> 属性+数据](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650356860189.png)
>
> 2. 将用户 ID+姓名属性作为 key，姓名数据作为 value。缺点是用户 ID 使用多次，数据冗余
>
> ![ID+属性 -> 数据](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650356955204.png)
>
> 
>
> 3. 因此通过 key(用户 ID) + field(属性标签) 就可以操作对应属性数据了，既不需要重复存储数据，也不会带来序列化和并发修改控制的问题。
>
> ![ID -> 属性 -> 数据](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650356950290.png)

#### 应用场景

1、由于 hash 数据类型的 key-value 的特性，用来存储关系型数据库中表记录，是 Redis 中哈希类型最常用的场景。一条记录作为一个 key-value，把每列属性值对应成 field-value 存储在哈希表当中，然后通过 key 值来区分表当中的主键。

2、经常被用来存储用户相关信息。优化用户信息的获取，不需要重复从数据库当中读取，提高系统性能。

***

## 五大基本类型底层数据结构

Redis 整体的存储结构：

Redis 内部整体的存储结构是一个大的 hashmap，内部是数组实现的 hash，key 冲突通过挂链表去实现，每个 dictEntry 为一个 key/value 对象，value 为定义的 redisObject。

结构图如下：

![Redis 整体的存储结构](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650433179786.png)

dictEntry 是存储 key->value 的地方，其中包含指向具体的 redisObject 的指针。

redisObject 中*ptr 指向具体的数据结构的地址；type 表示该对象的类型，即 String、List、Hash、Set、Zset 中的一个

### String (arrayList)

String 的数据结构为简单动态字符串(Simple Dynamic String，缩写 SDS)。是可以修改的字符串，内部结构实现上类似于 Java 的 ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配。

![String 的数据结构](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650356547555.png)

如图中所示，内部为当前字符串实际分配的空间 capacity 一般要高于实际字符串长度 len。当字符串长度小于 1M 时，扩容都是加倍现有的空间，如果超过 1M，扩容时一次只会多扩 1M 的空间。需要注意的是字符串最大长度为 512M。

### List (ziplist - quickList)

1、Redis 3.2 之前的底层实现方式：压缩列表 ziplist 或者 双向循环链表 linkedlist

当 list 存储的数据量比较少且同时满足下面两个条件时，list 就使用 ziplist 存储数据：

- list 中保存的每个元素的长度小于 64 字节；
- 列表中数据个数少于 512 个。

2、Redis 3.2 及之后的底层实现方式：quicklist

quicklist 是一个基于 ziplist 的双向链表，quicklist 的每个节点都是一个 ziplist，结合了双向链表和 ziplist 的优点。

#### **ziplist**

ziplist 是压缩列表，它的好处是能节省内存空间，因为它所存储的内容都是在连续的内存区域当中的。当列表对象元素不大，每个元素也不大的时候，就采用 ziplist 存储。但当数据量过大时就 ziplist 就不是那么好用了。因为为了保证他存储内容在内存中的连续性，插入的复杂度是 O(N)，即每次插入都会重新进行 realloc 重新分配内存空间。

ziplist 结构如下：

![ziplist 结构图](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650419709341.png)

1、zlbytes：用于记录整个压缩列表占用字节数

2、zltail：记录要列表尾节点距离压缩列表的起始地址有多少字节，用于快速定位到最后一个节点，从而可以在 ziplist 尾部快速的执行 push，pop 操作

3、zllen：记录了压缩列表包含的节点数量。

4、entryX：压缩列表包含的各个节点

5、zlend：用于标记压缩列表的末端，用来快速定位到最后一个元素，然后倒着遍历。（entry 块的 prevlen 字段表示前一个 entry 的字节长度，当压缩列表倒着遍历时，需要通过这个字段来快速定位到下一个元素的位置。）

**为什么数据量大时不用 ziplist？**

因为 ziplist 是一段连续的内存，插入的时间复杂化度为 O(n)，而且每当插入新的元素需要 realloc 做内存扩展；而且如果超出 ziplist 内存大小，还会做重新分配的内存空间，并将内容复制到新的地址。如果数量大的话，重新分配内存和拷贝内存会消耗大量时间。所以不适合大型字符串，也不适合存储量多的元素。

#### quickList

快速列表是 ziplist 和 linkedlist 的混合体，是将 linkedlist 按段切分，每一段用 ziplist 来紧凑存储，多个 ziplist 之间使用双向指针链接。

**为什么不直接使用 linkedlist？**

linkedlist 的附加空间相对太高，prev 和 next 指针就要占去 16 个字节，而且每一个结点都是单独分配，会加剧内存的碎片化，影响内存管理效率。

quicklist 结构图如下：

![quicklist 结构图](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650419719663.png)

**ziplist的长度**

quicklist 内部默认单个 ziplist 长度为 8k 字节，超出了这个字节数，就会新起一个 ziplist。关于长度可以使用 list-max-ziplist-size 决定。

**压缩深度**

我们上面说到了 quicklist 下是用多个 ziplist 组成的，同时为了进一步节约空间，Redis 还会对 ziplist 进行压缩存储，使用 LZF 算法压缩，可以选择压缩深度。quicklist 默认的压缩深度是 0，也就是不压缩。压缩的实际深度由配置参数 list-compress-depth 决定。为了支持快速 push/pop 操作，quicklist 的首尾两个 ziplist 不压缩，此时深度就是 1。如果深度为 2，就表示 quicklist 的首尾第一个 ziplist 以及首尾第二个 ziplist 都不压缩。

### Hash (ziplist - dict)

#### ziplist

当 Hash 中数据项比较少的情况下，Hash 底层才用压缩列表 ziplist 进行存储数据，随着数据的增加，底层的 ziplist 就可能会转成 dict，具体配置如下

> hash-max-ziplist-entries 512
> hash-max-ziplist-value 64

在如下两个条件之一满足的时候，ziplist 会转成 dict：

- 当 hash 中的数据项的数目超过 512 的时候，也就是 ziplist 数据项超过 1024 的时候；
- 当 hash 中插入的任意一个 value 的长度超过了 64 的时候。

每当有新的键值对加入到哈希对象时，先压入键，再压入值，如图：

![ziplist实现的hash](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/20190703171927309.png)

当遍历时，从头到尾的遍历，且跳过值节点。

#### dict

字典的哈希表结构和 Java 的 HashMap 结构几乎是一样的，都是通过某个哈希函数从 key 计算得到在哈希表中的位置，采用链表法解决冲突。Redis 这里使用的是单链表，为了查询效率每次把新数据插入到链表头位置（使用头插法是因为新插入的数据被访问的概率大，可以更快访问到）。

我们可以看到每个 dict 中都有两个 hashtable，结构图如下：

![dict实现的hash](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650420119549.png)

虽然 dict 结构有两个 hashtable，但是通常情况下只有一个 hashtable 是有值的。但是在 dict 扩容缩容的时候，需要分配新的 hashtable，然后进行渐近式搬迁，这时候两个 hashtable 存储的旧的 hashtable 和新的 hashtable。搬迁结束后，旧 hashtable 删除，新的取而代之。

**渐进式 rehash**

所谓渐进式 rehash 是指我们的大字典的扩容是比较消耗时间的，需要重新申请新的数组，然后将旧字典所有链表的元素重新挂接到新的数组下面，是一个 O(n)的操作。但是因为我们的 Redis 是单线程的，无法承受这样的耗时过程，所以采用了渐进式 rehash 小步搬迁，虽然慢一点，但是可以搬迁完毕。

**扩容条件**

我们的扩容一般会在 Hash 表中的元素个数等于第一维数组的长度的时候，就会开始扩容。扩容的大小是原数组的两倍。不过在 Redis 在做 bgsave（RDB 持久化操作的过程），为了减少内存页的过多分离（Copy On Write），Redis 不会去扩容。但是如果 hash 表的元素个数已经到达了第一维数组长度的 5 倍的时候，就会强制扩容，不管你是否在持久化。

不扩容主要是为了尽可能减少内存页过多分离，系统需要更多的开销去回收内存。

**缩容条件**

当我们的 hash 表元素逐渐删除的越来越少的时候。Redis 于是就会对 hash 表进行缩容来减少第一维数组长度的空间占用。缩容的条件是元素个数低于数组长度的 10%，并且缩容不考虑是否在做 Redis 持久化。

不用考虑 bgsave 主要是因为我们的缩容的内存都是已经使用过的，缩容的时候可以直接置空，而且由于申请的内存比较小，同时会释放掉一些已经使用的内存，不会增大系统的压力。

**rehash 步骤**

![rehash 步骤](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650420130964.png)

### Set (intset - hashSet)

Redis 的集合相当于 Java 中的 HashSet，它内部的键值对是无序、唯一的。它的内部实现相当于一个特殊的字典，字典中所有的 value 都是一个值 NULL。集合 Set 类型底层编码包括 inset 和 hashtable。

当存储的数据同时满足下面这样两个条件的时候，Redis 就采用整数集合 intset 来实现 set 这种数据类型：

> 存储的数据都是整数
> 存储的数据元素个数小于512个

当不能同时满足这两个条件的时候，Redis 就使用 dict 来存储集合中的数据

**intset**

intset 是一个有序集合，查找元素的复杂度为 O(logN)（采用二分法），但插入时不一定为 O(logN)，因为有可能涉及到升级操作。比如当集合里全是 int16_t 型的整数，这时要插入一个 int32_t，那么为了维持集合中数据类型的一致，那么所有的数据都会被转换成 int32_t 类型，涉及到内存的重新分配，这时插入的复杂度就为 O(N)了。是 intset 不支持降级操作。

**intset 是有序不要和 zset 搞混，zset 是设置一个 score 来进行排序，而 intset 这里只是单纯的对整数进行升序而已**

### Sorted Set (ziplist - zset(dict + skipList))

Zset 有序集合和 set 集合有着必然的联系，他保留了集合不能有重复成员的特性，但不同的是，有序集合中的元素是可以排序的，但是它和列表的使用索引下标作为排序依据不同的是，它给每个元素设置一个分数 score，作为排序的依据。

当数据较少时，sorted set 是由 ziplist 来实现的。
当数据多的时候，sorted set 是由 zset 的数据结构来实现的，这个 zset 包含一个 dict + 一个 skiplist。dict 用来查询数据到分数(score)的对应关系，而 skiplist 用来根据指定分数 score 的范围来获取 value 列表。

**sorted set ziplist 实现**

在这里我们先来讨论一下前一种情况——基于 ziplist 实现的 sorted set。ziplist 就是由很多数据项组成的一大块连续内存。由于 sorted set 的每一项元素都由数据和 score 组成，因此，当使用 zadd 命令插入一个(数据, score)对的时候，底层在相应的 ziplist 上就插入两个数据项：数据在前，score 在后。

![基于 ziplist 实现的 sorted set](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650431975904.png)

ziplist 的主要优点是节省内存，但它上面的查找操作只能按顺序查找（可以正序也可以倒序）。因此，sorted set 的各个查询操作，就是在 ziplist 上从前向后（或从后向前）一步步查找，每一步前进两个数据项，跨域一个(数据, score)对。

随着数据的插入，sorted set 底层的这个 ziplist 就可能会转成 zset 的实现。那么到底插入多少才会转呢？

> zset-max-ziplist-entries 128	# 当sorted set中的元素个数，即(数据， score)对的数目超过128的时候
> zset-max-ziplist-value 64	# 当sorted set中插入的任意一个数据的长度超过了64的时候。

**sorted set ziplist实现**
Redis 的 zset 是一个复合结构，一方面它需要一个 hash 结构来存储 value 和 score 的对应关系，另一方面需要提供按照 score 来排序的功能，还需要能够指定 score 的范围来获取 value 列表的功能，这就需要另外一个结构「跳跃表」。

```c
typedef struct zset {
    dict *dict;
    zskiplist *zsl;
} zset;
```

跳表：链表+多级索引（空间换时间）

和其他结构的比较：

对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。Redis 采用的是跳跃表。跳跃表效率堪比红黑树，实现远比红黑树简单。

***

## 三大特殊数据类型

### geospatial（地理位置）

1. 查看附近的人
2. 微信位置共享
3. 地图上直线距离的展示

### Hyperloglog（基数）

基数：不重复的元素

网页统计 UV （浏览用户数量，同一天同一个 ip 多次访问算一次访问，目的是计数，而不是保存用户）

传统的方式，set 保存用户的 id，可以统计 set 中元素数量作为标准判断。

但如果这种方式保存大量用户 id，会占用大量内存，我们的目的是为了计数，而不是去保存 id。

### Bitmaps（位存储）

Redis 提供了 Bitmaps 这个“数据类型”可以实现对位的操作：

1. Bitmaps 本身不是一种数据类型， 实际上它就是字符串（key-value） ， 但是它可以对字符串的位进行操作。
2. Bitmaps 单独提供了一套命令， 所以在 Redis 中使用 Bitmaps 和使用字符串的方法不太相同。 可以把 Bitmaps 想象成一个以位为单位的数组， 数组的每个单元只能存储 0 和 1， 数组的下标在 Bitmaps 中叫做偏移量。

#### 命令

1. setbit

	`setbit<key><offset><value>` 设置 Bitmaps 中某个偏移量的值（0 或 1）

	可以将 idx 位置为 1 表示 idx 用户访问过

2. getbit

	`getbit<key><offset>` 获取 Bitmaps 中某个偏移量的值

	通过 idx 位是否为 1 判断 idx 用户是否访问过

3. bitcount

	`bitcount<key>[start end]` 统计字符串从 start 字节到 end 字节比特值为 1 的数量

4. bitop

	`bitop and(or/not/xor) <destkey> [key…]` bitop 是一个复合操作， 它可以做多个 Bitmaps 的 and（交集） 、 or（并集） 、 not（非） 、 xor（异或） 操作并将结果保存在 destkey 中。

#### **应用场景**

**两种状态的统计**都可以使用 bitmaps，例如：统计用户活跃与非活跃数量、登录与非登录、上班打卡等等。

***

## Redis事务与锁机制

### 事务

事务本质：一组命令的集合

#### **数据库的事务**

数据库事务通过 ACID（原子性、一致性、隔离性、持久性）来保证。

数据库中除查询操作以外，插入(Insert)、删除(Delete)和更新(Update)这三种操作都会对数据造成影响，因为事务处理能够保证一系列操作可以完全地执行或者完全不执行，因此在一个事务被提交以后，该事务中的任何一条 SQL 语句在被执行的时候，都会生成一条撤销日志(Undo Log)。

#### Redis 事务

Redis 事务提供了一种“将多个命令打包， 然后一次性、按顺序地执行”的机制， 并且事务在执行的期间不会主动中断 —— 服务器在执行完事务中的所有命令之后， 才会继续处理其他客户端的其他命令。事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚。

Redis 中一个事务从开始到执行会经历**开始**事务（Muiti）、命令**组队**和**执行**事务(Exec)三个阶段，事务中的命令在加入时都没有被执行，直到提交时才会开始执行(Exec)一次性完成。

组队的过程中可以通过 discard 来放弃组队。

**事务的错误处理**

组队阶段某个命令报错，执行时整个队列都会被取消。

![组队阶段某个命令报错](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650372301056.png)

如果执行阶段某个命令报错，则只有报错的命令不会被执行，而其他的命令都会执行，不会回滚。（不保证事务的原子性）

![执行阶段某个命令报错](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650372359361.png)

**为什么 Redis 不支持回滚来保证原子性**

Redis 命令只会因为错误的语法而失败，然而没有任何机制能避免程序员自己造成的错误， 并且这类错误通常不会在生产环境中出现， 所以 Redis 选择了更简单、更快速的无回滚方式来处理事务。

### Redis 锁

#### 悲观锁

**悲观锁(Pessimistic Lock)**， 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。**传统的关系型数据库里边就用到了很多这种锁机制**，比如**行锁**，**表锁**等，**读锁**，**写锁**等，都是在做操作之前先上锁。

![悲观锁](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650417157625.png)

> 线程 1 拿到账户后把账户锁住，然后操作，这时候线程 2 也想拿账户，发现加了锁拿不到，只能阻塞等到锁释放。

#### 乐观锁

**乐观锁(Optimistic Lock)，** 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在**更新的时候判断**一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。**乐观锁适用于多读的应用类型，这样可以提高吞吐量**。Redis 就是利用这种机制实现事务的。

![乐观锁](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650417222072.png)

> 给账户数据加一个版本号，假如线程 1、2 都拿到账户，线程 1 使用账户前先进行版本号比较，v1.0 = v1.0，则扣减账户余额，账户版本号变更为 v1.1，线程 2 准备操作数据库，发现自己预期的账户版本号 v1.0 和当前账户版本号 v1.1 不同（CAS），则不能操作。 

***

## Redis持久化

Redis 是一种内存型数据库，一旦服务器进程退出，数据库的数据就会丢失，为了解决这个问题 Redis 供了两种持久化的方案，将内存中的数据保存到磁盘中，避免数据的丢失。

两种持久化方式：快照（RDB 文件）和追加式文件（AOF 文件），下面分别为大家介绍两种方式的原理。

- RDB 持久化方式会在一个特定的间隔保存那个时间点的数据快照。

- AOF 持久化方式则会记录每一个服务器收到的写操作。在服务启动时，这些记录的操作会逐条执行从而重建出原来的数据。写操作命令记录的格式跟 Redis 协议一致，以追加的方式进行保存。

- 两种方式的持久化是可以同时存在的，但是当 Redis 重启时，AOF 文件会被优先用于重建数据。

### RDB(Redis Database)

> 在满足特定的  Redis 操作条件（时间周期、写操作次数）时，将内存中的数据以 **数据快照** 的形式存储到 db 文件中

#### **工作原理**

> [Redis_RDB持久化之写时复制技术的应用](https://www.cnblogs.com/zyf98/p/15934058.html)

RDB 是一次的全量备份，即周期性的把 Redis 当前内存中的全量数据写入到一个快照文件中。Redis 是单线程程序，这个线程要同时负责多个客户端的读写请求，还要负责周期性的把当前内存中的数据写到快照文件中 RDB 中，那么就会带来以下两个问题：

1. 数据写到 RDB 文件是 IO 操作，IO 操作会严重影响 Redis 的性能，甚至在持久化的过程中，读写请求会阻塞
2. 假如 Redis 正在进行持久化一个大的数据结构，在这个过程中客户端发送一个删除请求，把这个大的数据结构删掉了，这时候持久化的动作还没有完成。

于是 Redis 使用操作系统的多进程 **写时复制(Copy On Write)机制** 来实现快照的持久化，在持久化过程中调用**glibc**(Linux 下的 C 函数库)的函数 **fork()** 产生一个子进程，该子进程和父进程共享内存里面的代码段和数据段，快照持久化完全交给子进程来处理，父进程继续处理客户端的读写请求。

> 子进程将数据写入到一个临时 RDB 文件中。当子进程完成持久化时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。

在子进程持久化的过程中，如果主进程收到的客户端的读写请求，需要修改某块数据，那么这块数据就会被复制一份到内存，生成该数据的副本，主进程在该副本上进行修改操作。所以即使对某个数据进行了修改，Redis 持久化到 RDB 中的数据也是未修改的数据，这也是把 RDB 文件称为"快照"文件的原因，子进程所看到的数据在它被创建的一瞬间就固定下来了，父进程修改的某个数据只是该数据的复制品。这里再深入一点，Redis 内存中的全量数据由一个个的"数据段页面"组成，每个数据段页面的大小为 4K，客户端要修改的数据在哪个页面中，就会复制一份这个页面到内存中，这个复制的过程称为"页面分离"，在持久化过程中，随着分离出的页面越来越多，内存就会持续增长，但是不会超过原内存的 2 倍，因为在一次持久化的过程中，几乎不会出现所有的页面都会分离的情况，读写请求针对的只是原数据中的小部分，大部分 Redis 数据还是"冷数据"。

正因为修改的部分数据会被额外的复制一份，所以会占用额外的内存，当在进行 RDB 持久化操作的过程中，与此同时如果持续往 Redis 中写入的数据量越多，就会导致占用的额外内存消耗越大。

![RDB 持久化](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes2093494-20220226233822789-2067752089.png)

**优点**：

1. RDB 的内容为二进制的数据，占用内存更小，更紧凑，更适合做为备份文件；
2. RDB 对灾难恢复非常有用，它是一个紧凑的文件，可以更快的传输到远程服务器进行 Redis 服务恢复；
3. RDB 可以更大程度的提高 Redis 的运行速度，因为每次持久化时 Redis 主进程都会 fork() 一个子进程，进行数据持久化到磁盘，Redis 主进程并不会执行磁盘 I/O 等操作；
4. 与 AOF 格式的文件相比，RDB 文件可以更快的重启。

**缺点**：

1. 因为 RDB 只能保存某个时间间隔的数据，如果中途 Redis 服务被意外终止了，则会丢失一段时间内的 Redis 数据。
2. RDB 需要经常 fork() 才能使用子进程将其持久化在磁盘上。如果数据集很大，fork() 可能很耗时，并且如果数据集很大且 CPU 性能不佳，则可能导致 Redis 停止为客户端服务几毫秒甚至一秒钟。

### AOF(Append Only File)

以日志的形式来记录每个写的操作，将 Redis 执行过的所有指令记录下来（读操作不记录），只许追加文件但不可以改写文件，Redis 启动之初会读取该文件重新构建数据，换言之，Redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

#### **AOF配置项**

```
# 默认不开启aof  而是使用rdb的方式
appendonly no

# 默认文件名
appendfilename "appendonly.aof"

# 每次修改都会sync 消耗性能
# appendfsync always
# 每秒执行一次 sync 可能会丢失这一秒的数据
appendfsync everysec
# 不执行 sync ，这时候操作系统自己同步数据，速度最快
# appendfsync no 
```

AOF 的整个流程大体来看可以分为两步，第一步是命令的实时写入（如果是 appendfsync everysec 配置，会有 1s 损耗），第二步是对 aof 文件的重写。

#### **AOF 重写机制**

随着 Redis 的运行，AOF 的日志会越来越长，如果实例宕机重启，那么重放整个 AOF 将会变得十分耗时，而在日志记录中，又有很多无意义的记录，比如我现在将一个数据 incr 一千次，那么就不需要去记录这 1000 次修改，只需要记录最后的值即可。所以就需要进行 AOF 重写。

Redis 提供了 bgrewriteaof 指令用于对 AOF 日志进行重写，该指令运行时会开辟一个子进程对内存进行遍历，然后将其转换为一系列的 Redis 的操作指令，再序列化到一个日志文件中。完成后再替换原有的 AOF 文件，至此完成。

同样的也可以在 Redis.config 中对重写机制的触发进行配置：

通过将 no-appendfsync-on-rewrite 设置为 yes，开启重写机制；auto-aof-rewrite-percentage 100 意为比上次从写后文件大小增长了 100%再次触发重写；

auto-aof-rewrite-min-size 64mb 意为当文件至少要达到 64mb 才会触发制动重写。

**优点**：

1、数据安全，AOF 是对指令文件进行 **增量更新**，更适合实时性持久化。

2、通过 append 模式写文件，即使中途服务器宕机，可以通过 Redis-check-aof 工具解决数据一致性问题。

3、AOF 机制的 rewrite 模式。AOF 文件没被 rewrite 之前（文件过大时会对命令 进行合并重写），可以删除其中的某些命令（比如误操作的 flushall）)

**缺点**：

1、AOF 文件比 RDB 文件大，且恢复速度慢。

2、数据集大的时候，比 RDB 启动效率低。

### RDB与AOF对比

Redis 官方建议同时开启两种持久化策略，AOF 和 RDB 同时开启，系统默认取 AOF 的数据（数据不会存在丢失）

| 比较项     | RDB    | AOF          |
| ---------- | ------ | ------------ |
| 启动优先级 | 低     | 高           |
| 体积       | 小     | 大           |
| 恢复速度   | 快     | 慢           |
| 数据安全性 | 丢数据 | 根据策略决定 |

***

## 发布与订阅

Redis 发布与订阅是一种消息通信的模式：发送者（pub）发送消息，订阅者（sub）接收消息。

Redis 通过 PUBLISH 和 SUBSCRIBE 等命令实现了订阅与发布模式，这个功能提供两种信息机制，分别是订阅/发布到频道、订阅/发布到模式的客户端。

### 频道（channel）

**订阅**

![频道订阅](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650436261125.png)

**发布**

![频道发布](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650436267143.png)

#### **完整流程**

发布者向频道 channel:1 发布消息 hi

```
127.0.0.1:6379> publish channel:1 hi(integer) 1
```

订阅者订阅消息

```
127.0.0.1:6379> subscribe channel:1
Reading messages... (press Ctrl-C to quit)
1) "subscribe" // 消息类型
2) "channel:1" // 频道
3) "hi" // 消息内容
```

执行 subscribe 后客户端会进入订阅状态，仅可以使 subscribe、unsubscribe、psubscribe 和 punsubscribe 这四个属于"发布/订阅"之外的命令

订阅频道后的客户端可能会收到三种消息类型

- subscribe：表示订阅成功的反馈信息。第二个值是订阅成功的频道名称，第三个是当前客户端订阅的频道数量。

- message：表示接收到的消息，第二个值表示产生消息的频道名称，第三个值是消息的内容。

- unsubscribe：表示成功取消订阅某个频道。第二个值是对应的频道名称，第三个值是当前客户端订阅的频道数量，当此值为 0 时客户端会退出订阅状态，之后就可以执行其他非"发布/订阅"模式的命令了。



#### **数据结构**

基于 channel 的发布订阅模式是通过字典数据类型实现的

> 类似hashmap，键为channel，值为订阅了该channel的client，用链表链接起来

其中，字典的键为正在被订阅的频道， 而字典的值则是一个链表， 链表中保存了所有订阅这个频道的客户端。

![频道模式数据结构模型](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650420774862.png)

**订阅**

当使用 subscribe 订阅时，在字典中找到频道 key（如没有则创建），并将订阅的 client 关联在链表后面。

当 client 10 执行 `subscribe channel1 channel2 channel3` 时，会将 client 10 分别加到 channel1 channel2 channel3 关联的链表尾部。

![频道订阅](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650420784179.png)

**发布**

发布时，根据 key，找到字典汇总 key 的地址，然后将 msg 发送到关联的链表每一台机器。

**退订**

遍历关联的链表，将指定的地址删除即可。



### 模式（pattern）

pattern 使用了通配符的方式来订阅

通配符中 ? 表示 1 个占位符，\* 表示任意个占位符(包括 0)，?* 表示 1 个以上占位符。

所以当使用 publish 命令发送信息到某个频道时， 不仅所有订阅该频道的客户端会收到信息， 如果有某个/某些模式和这个频道匹配的话， 那么所有订阅这个/这些频道的客户端也同样会收到信息。

![模式](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650420803487.png)

#### **订阅发布完整流程**

发布者发布消息

```
127.0.0.1:6379> publish b m1
(integer) 1
127.0.0.1:6379> publish b1 m1
(integer) 1
127.0.0.1:6379> publish b11 m1
(integer) 1
```

订阅者订阅消息

```
127.0.0.1:6379> psubscribe b*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "b*"
3) (integer) 3
1) "pmessage"
2) "b*"
3) "b"
4) "m1"
1) "pmessage"
2) "b*"
3) "b1"
4) "m1"
1) "pmessage"
2) "b*"
3) "b11"
4) "m1"
```

#### **数据结构**

pattern 属性是一个链表，链表中保存着所有和模式相关的信息。

> 类似链表，每个pattern节点，包含pattern以及订阅该pattern的client

数据结构图如下：

![模式数据结构](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650420905733.png)

**订阅**

当有信的订阅时，会将订阅的客户端和模式信息添加到链表后面。

![模式订阅](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650420913717.png)

**发布**

当发布者发布消息时，首先会发送到对应的频道上，在遍历模式列表，根据 key 匹配模式，匹配成功将消息发给对应的订阅者。

**退订**

使用 punsubscribe，可以将订阅者退订，将改客户端移除出链表。

***

## 主从复制

**什么是主从复制**

在多个 Redis 实例建立起主从关系，当 主 Redis 中的数据发生变化，从 Redis 中的数据也会同步变化

- 通过主从配置可以实现 Redis 数据的备份（从 Redis 就是对 主 Redis 的备份），保证数据安全性
- 主从库之间采用的是读写分离的方式，读操作：主库、从库都可以接收；写操作：首先到主库执行，然后主库将写操作同步给从库

**主从复制的作用**

- 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。

- 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。

- 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写 Redis 数据时应用连接主节点，读 Redis 数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器的并发量。

- 高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是 Redis 高可用的基础。

主从库采用的是读写分离的方式：

![主从库读写分离](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650420963646.png) 

### 原理

分为全量复制与增量复制

全量复制：发生在第一次复制时

增量复制：只会把主从库网络断连期间主库收到的命令，同步给从库

### 全量复制的三个阶段

![全量复制](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNoteskjk7gr48ft.jpeg)

第一阶段是主从库间建立连接、协商同步的过程。

> 主要是为全量复制做准备。从库和主库建立起连接，并告诉主库即将进行同步，主库确认回复后，主从库间就可以开始同步了。
>
> 具体来说，从库给主库发送 psync 命令，表示要进行数据同步，主库根据这个命令的参数来启动复制。psync 命令包含了主库的 runID 和复制进度 offset 两个参数。runID，是每个 Redis 实例启动时都会自动生成的一个随机 ID，用来唯一标记这个实例。当从库和主库第一次复制时，因为不知道主库的 runID，所以将 runID 设为“？”。offset，此时设为 -1，表示第一次复制。主库收到 psync 命令后，会用 FULLRESYNC 响应命令带上两个参数：主库 runID 和主库目前的复制进度 offset，返回给从库。从库收到响应后，会记录下这两个参数。这里有个地方需要注意，FULLRESYNC 响应表示第一次复制采用的全量复制，也就是说，主库会把当前所有的数据都复制给从库。

第二阶段，主库将所有数据同步给从库。从库收到数据后，在本地完成数据加载。这个过程依赖于内存快照生成的 RDB 文件。

> 具体来说，主库执行 bgsave 命令，生成 RDB 文件，接着将文件发给从库。从库接收到 RDB 文件后，会先清空当前数据库，然后加载 RDB 文件。这是因为从库在通过 replicaof 命令开始和主库同步前，可能保存了其他数据。为了避免之前数据的影响，从库需要先把当前数据库清空。在主库将数据同步给从库的过程中，主库不会被阻塞，仍然可以正常接收请求。否则，Redis 的服务就被中断了。但是，这些请求中的写操作并没有记录到刚刚生成的 RDB 文件中。为了保证主从库的数据一致性，主库会在内存中用专门的 replication buffer，记录 RDB 文件生成后收到的所有写操作。

第三个阶段，主库会把第二阶段执行过程中新收到的写命令，再发送给从库。

> 具体的操作是，当主库完成 RDB 文件发送后，就会把此时 replication buffer 中的修改操作发给从库，从库再重新执行这些操作。这样一来，主从库就实现同步了。

### 断网增量更新

当主从库断连后，主库会把断连期间收到的写操作命令，写入 replication buffer，同时也会把这些操作命令也写入 repl_backlog_buffer 这个缓冲区。

![断网增量更新](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesg0eezljwqy.jpeg)

## 哨兵机制

> 哨兵的核心功能是主节点的自动故障转移
>
> 1. 监控 主库，判断 主库 是否宕机
> 2. 主库 如果宕机，则从 从库 中 选举成为 主库（因此需要奇数个哨兵）
> 3. 更改主从配置

下图是一个典型的哨兵集群监控的逻辑图：

![哨兵机制](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650420977306.png)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

 

Redis Sentinel 包含了若个 Sentinel 节点，这样做也带来了两个好处：

1. 对于节点的故障判断是由多个 Sentinel 节点共同完成，这样可以有效地防止误判
2. 即使个别 Sentinel 节点不可用，整个 Sentinel 集群依然是可用的。

哨兵实现了以下功能

1. 监控：每个 Sentinel 节点会对数据节点（Redis master/slave 节点）和其余 Sentinel 节点进行监控
2. 通知：Sentinel 节点会将故障转移的结果通知给应用方
3. 故障转移：实现 slave 晋升为 master，并维护后续正确的主从关系
4. 配置中心：在 Redis Sentinel 模式中，客户端在初始化的时候连接的是 Sentinel 节点集合，从中获取主节点信息

其中，监控和自动故障转移功能，使得哨兵可以及时发现主节点故障并完成转移；而配置中心和通知功能，则需要在与客户端的交互中才能体现。

### 监控

Sentinel 节点需要监控 master、slave 以及其它 Sentinel 节点的状态。这一过程是通过 Redis 的 pub/sub 系统实现的。Redis Sentinel 一共有三个定时监控任务，完成对各个节点发现和监控：

1. 监控主从拓扑信息：每隔 10 秒，每个 Sentinel 节点，会向 master 和 slave 发送 INFO 命令获取最新的拓扑结构
2. Sentinel 节点信息交换：每隔 2 秒，每个 Sentinel 节点，会向 Redis 数据节点的`sentinel:hello` 频道上，发送自身的信息，以及对主节点的判断信息。这样，Sentinel 节点之间就可以交换信息
3. 节点状态监控：每隔 1 秒，每个 Sentinel 节点，会向 master、slave、其余 Sentinel 节点发送 PING 命令做心跳检测，来确认这些节点当前是否可达

### 主观/客观下线

#### 主观下线

每个 Sentinel 节点，每隔 1 秒会对数据节点发送 ping 命令做心跳检测，当这些节点超过 down-after-milliseconds 没有进行有效回复时，Sentinel 节点会对该节点做失败判定，这个行为叫做主观下线。

#### 客观下线

客观下线，是指当大多数 Sentinel 节点，都认为 master 节点宕机了，那么这个判定就是客观的，叫做客观下线。

那么这个大多数是指多少呢？这其实就是分布式协调中的 quorum 判定了，大多数就是过半数，比如哨兵数量是 5，那么大多数就是 5/2+1=3 个，哨兵数量是 10 大多数就是 10/2+1=6 个。

注：Sentinel 节点的数量至少为 3 个，否则不满足 quorum 判定条件。

### 哨兵选举

如果发生了客观下线，那么哨兵节点会选举出一个 Leader 来进行实际的故障转移工作。Redis 使用了 Raft 算法来实现哨兵领导者选举，大致思路如下：

1. 每个 Sentinel 节点都有资格成为领导者，当它主观认为某个数据节点宕机后，会向其他 Sentinel 节点发送 sentinel is-master-down-by-addr 命令，要求自己成为领导者；
2. 收到命令的 Sentinel 节点，如果没有同意过其他 Sentinel 节点的 sentinelis-master-down-by-addr 命令，将同意该请求，否则拒绝（每个 Sentinel 节点只有 1 票）；
3. 如果该 Sentinel 节点发现自己的票数已经大于等于 MAX(quorum, num(sentinels)/2+1)，那么它将成为领导者；
4. 如果此过程没有选举出领导者，将进入下一次选举。

### **故障转移**

选举出的 Leader Sentinel 节点将负责故障转移，也就是进行 master/slave 节点的主从切换。故障转移，首先要从 slave 节点中筛选出一个作为新的 master，主要考虑以下 slave 信息：

1. 跟 master 断开连接的时长：如果一个 slave 跟 master 的断开连接时长已经超过了 down-after-milliseconds 的 10 倍，外加 master 宕机的时长，那么该 slave 就被认为不适合选举为 master；
2. slave 的优先级配置：slave priority 参数值越小，优先级就越高；
3. 复制 offset：当优先级相同时，哪个 slave 复制了越多的数据（offset 越靠后），优先级越高；
4. run id：如果 offset 和优先级都相同，则哪个 slave 的 run id 越小，优先级越高。

接着，筛选完 slave 后， 会对它执行 slaveof no one 命令，让其成为主节点。

最后，Sentinel 领导者节点会向剩余的 slave 节点发送命令，让它们成为新的 master 节点的从节点，复制规则与 parallel-syncs 参数有关。

Sentinel 节点集合会将原来的 master 节点更新为 slave 节点，并保持着对其关注，当其恢复后命令它去复制新的主节点。

注：Leader Sentinel 节点，会从新的 master 节点那里得到一个 configuration epoch，本质是个 version 版本号，每次主从切换的 version 号都必须是唯一的。其他的哨兵都是根据 version 来更新自己的 master 配置。

## 集群配置

> 高可用：保证 Redis 一直处于可用状态，即使出现了故障也有备用方案保证可用性
>
> 高并发：一个 Redis 实例已经可以支持多达 11w并发读操作或 8.1w并发写操作；但是如果有更高并发需求的应用来说，可以通过 读写分离、集群配置 来解决高并发问题

- Redis 集群中每个节点是对等的，无中心结构
- 数据按照 slots 分布式存储在不同的 Redis 节点上，节点中的数据可共享，可以动态调整数据分布
- 可扩展性强，可以动态增删节点，最多可扩展至 1000+ 节点
- 集群的每个节点通过 `主从（哨兵模式）`保证其高可用性
- Redis 并不能保证数据的强一致性，这意味这在实际中集群在特定的条件下可能会丢失写操作
- Redis 集群之间是如何复制的？异步复制

> [Redis_数据分布算法：传统哈希-一致性哈希-哈希slot](https://blog.csdn.net/qq_43846090/article/details/123721066)

### 传统哈希

最简单的数据分布算法，对进来的 key 进行 hash，然后对节点数据进行取模，就知道分布到哪个节点上了；
缺点就是: 如果一个节点宕机，所有缓存的位置都要发生改变，当服务器数量发生改变时，所有缓存在一定时间内是失效的，会引起缓存的雪崩，可能会引起整体系统压力过大而崩溃（大量缓存同一时间失效）

### 一致性哈希

实际上就是引入了一个圆环的概念， 让每个数据的 hash 分布到整个圆环，然后顺时针去找离他最近的节点存储

<img src="https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650525022243.png" alt="一致性哈希" style="zoom: 50%;" />

hash 算法的取模法是对服务器的数量进行取模，而一致性哈希算法是对 2^32 取模，
dubbo 的复杂均衡有用到该算法；

好处:

使用 hash 算法，服务器数量发生改变时，所有服务器的所有缓存在同一时间失效了；
而使用一致性哈希算法时，服务器的数量如果发生改变，并不是所有缓存都会失效，而是只有部分缓存会失效。上图中如果节点 A 失效，则 C → A 之间的数据会重新缓存到节点 B 上。

hash 环偏斜：

<img src="https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650525154544.png" alt="hash 环偏斜" style="zoom: 50%;" />

1 号、2 号、3 号、4 号、6 号数据均被缓存在了服务器 A 上；只有 5 号被缓存在了服务器 B 上；服务器 C 上甚至没有缓存任何图片
如果出现上图中的情况，A、B、C 三台服务器并没有被合理的平均的充分利用，缓存分布的极度不均匀，而且，如果此时服务器 A 出现故障，那么失效缓存的数量也将达到最大值，在极端情况下，仍然有可能引起系统的崩溃；

上图中的情况则被称之为 hash 环的偏斜 ；
我们应该怎样防止 hash 环的偏斜？
增加虚拟节点：

<img src="https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650525203417.png" alt="增加虚拟节点防止 hash 环的偏斜" style="zoom:50%;" />

为了解决这个数据负载均衡的问题，搞出来虚拟节点，把真实节点搞一堆虚拟节点分布到环，那么整个区间的数据会落到这些虚拟节点上；

虚拟节点越多，hash 环上的节点就越多，缓存被均匀分布的概率就越大。

### 哈希槽

Redis 集群（cluster）并没有选用上面一致性哈希，而是采用了哈希槽（slot）的这种概念。主要的原因就是上面所说的，一致性哈希算法对于数据分布、节点位置的控制并不是很友好。

首先哈希槽其实是两个概念，第一个是哈希算法。Redis cluster 的 hash 算法不是简单的 hash()，而是 crc16 算法，一种校验算法。另外一个就是槽位的概念，空间分配的规则。其实哈希槽的本质和一致性哈希算法非常相似，不同点就是对于哈希空间的定义。一致性哈希的空间是一个圆环，节点分布是基于圆环的，无法很好的控制数据分布。而 Redis cluster 的槽位空间是自定义分配的，类似于 windows 盘分区的概念。这种分区是可以自定义大小，自定义位置的。

Redis cluster 包含了 16384 个哈希槽，每个 key 通过计算后都会落在具体一个槽位上，而这个槽位是属于哪个存储节点的，则由用户自己定义分配。例如机器硬盘小的，可以分配少一点槽位，硬盘大的可以分配多一点。如果节点硬盘都差不多则可以平均分配。所以哈希槽这种概念很好地解决了一致性哈希的弊端。

**为什么哈希槽的大小是固定的16384？**

> `CRC16`算法产生的hash值有16bit，该算法可以产生2^16-=65536个值。换句话说，值是分布在0~65535之间。那作者在做`mod`运算的时候，为什么不`mod`65536，而选择`mod`16384？

**不能太大，主要从节点间的通信性能方面考虑：**

每个节点之间是需要进行一个通信的，也就是不断地 pingpong 机制，让每个节点连接对方的信息；发送的消息头中有一个 bitmap，其中每一位代表一个槽，如果该位为 1，表示这个槽是属于当前节点，如果槽位过大，那么这个 bitmap 就会太大，心跳包过于庞大，浪费带宽。

**不能太小，主要从压缩率方面考虑：**

由于 bitmap 在传输过程中会进行压缩，bitmap 的填充率用 slots/N 表示（N 即节点数），对于同样的 slots，如果 N 太小，导致其填充率过高，压缩率就很低。

***

## 缓存穿透、击穿、雪崩

### 缓存穿透

**问题来源**

缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求。由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。在流量大时，可能 DB 就挂掉了，要是有人利用不存在的 key 频繁攻击我们的应用，每次都去 DB 判断然而又不能复制到 Redis 中，这就是漏洞。

例如发起为 id 为“-1”的数据或 id 为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

**解决方案**

1. **接口校验**：接口层增加校验，例如对 id<=0 的直接拦截。

2. **缓存空值**：从缓存取不到的数据，在数据库中也没有取到，这时也可以将 key-value 对写为 key-null，缓存有效时间可以设置短点，如 10 秒（设置太长会导致正常情况也没法使用）。

3. **布隆过滤器**：布隆过滤器用于快速判某个元素是否存在于集合中，由一个很长的二进制向量（位图）和一系列哈希函数两部分组成。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的错误识别率，添加到集合中的元素越多，误报的可能性就越大。

	位数组中的每个元素都只占用 1 bit ，并且每个元素只能是 0 或者 1。这样申请一个 100w 个元素的位数组只占用 1000000Bit / 8 = 125000 Byte = 125000/1024 kb ≈ 122kb 的空间。

	> [布隆过滤器](https://javaguide.cn/cs-basics/data-structure/bloom-filter.html)
	>
	> [缓存穿透-布隆过滤器](https://javaguide.cn/database/Redis/Redis-questions-01.html#%E7%BC%93%E5%AD%98%E7%A9%BF%E9%80%8F)

	具体而言：

	把 **所有可能存在的请求的值** 都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。布隆过滤器判断不存在的话，则肯定不存在，布隆过滤器判断存在，则进一步查 Redis，Redis 中存在则返回，Redis 中不存在则查数据库。

	为什么布隆过滤器会出现误判的情况呢? 

	**当一个元素加入布隆过滤器中的时候，会进行以下操作：**

	1. 使用布隆过滤器中的哈希函数对元素值进行计算，得到哈希值（有几个哈希函数得到几个哈希值）。

		> hashVal1 = hashMethod1(key)， hashVal2= hashMethod2(key)...

	2. 根据得到的哈希值，在位数组中把对应下标的值置为 1。

		> 把位数组中的 hashVal1 和 hashVal2 处置为 1

	**当我们需要判断一个元素是否存在于布隆过滤器的时候，会进行以下操作：**

	1. 对给定元素再次进行相同的哈希计算；

		> hashVal1 = hashMethod1(key)， hashVal2= hashMethod2(key)...

	2. 得到值之后判断位数组中的每个元素是否都为 1，如果值都为 1，那么说明这个值在布隆过滤器中，如果存在一个值不为 1，说明该元素不在布隆过滤器中。

		> 判断位数组中的 hashVal1 和 hashVal2 处是否全部为 1，全部为 1说明当前元素存在于位数组中。

	然而会出现这样一种情况：**不同的字符串可能哈希出来的位置相同。** （可以适当增加位数组大小或者调整我们的哈希函数来降低概率）

	> **⭐例如**：val1 经过两次hash得到下标 1， 3，将1、3处置为1，val2经过两次hash得到下标2、4，将2、4处置为1，对于新元素val3经过两次hash得到下标1、2，判断这两个位置都为1，误判以为val3存在，实际上不存在。


### 缓存击穿

**问题来源**

缓存击穿是指一个热点的 key，有大并发集中对其进行访问，突然间这个 key 失效了，导致大并发全部打在数据库上，导致数据库压力剧增。这种现象就叫做缓存击穿。

**解决方案**

> 热点key失效 → 打到数据库上，因此可以从这两方面进行解决

1、设置热点数据永远不过期。

2、接口限流、降级与熔断。重要的接口一定要做好限流策略，防止用户恶意刷接口，同时要降级准备，当接口中的某些服务不可用时候，进行熔断，失败快速返回机制。

3、加互斥锁：如果缓存失效的情况，只有拿到锁才可以查询数据库，分布式场景下可以使用分布式锁。

### 缓存雪崩

**问题来源**

缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至 down 机。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是指很多不同的数据都过期了，都查不到从而查数据库。

**解决方案**

1. **缓存数据的过期时间设置随机**，防止同一时间大量数据过期现象发生。
2. 设置热点数据永远不过期。
3. 在即将发生大并发访问之前，进行**缓存预热**将可能大量访问的数据加载到缓存。
4. 接口限流、降级与熔断。重要的接口一定要做好限流策略，防止用户恶意刷接口，同时要降级准备，当接口中的某些服务不可用时候，进行熔断，失败快速返回机制。
5. 为了防止 Redis 宕机导致缓存雪崩的问题，可以**搭建Redis集群**，提高 Redis 的容灾性。
6. 提高数据库的容灾能力，可以使用**数据库分库分表**，读写分离的策略。

## Redis 淘汰策略

### Redis 内存淘汰策略

> Redis 是基于内存结构进行数据缓存的，当内存资源消耗完毕，当将要有新的数据缓存进来时，为了腾出空间放新的数据，需要将内存中的一些数据释放掉，这种释放数据的策略称为 Redis 的淘汰策略。

LRU：Least Recently Used

LFU：Least Frequently Used

LRU 淘汰的是最久未访问到的数据，而 LFU 是淘汰的是最不经常使用的数据（若两个或多个数据的使用频率相同时，LFU 会再选择最久未访问到的数据淘汰）。

```conf
# volatile-lru -> 在设置了过期时间的数据中使用 LRU
# allkeys-lru -> 在所有数据中使用 LRU
# volatile-lfu -> 在设置了过期时间的数据中使用 LFU
# allkeys-lfu -> 在所有数据中使用 LFU
# volatile-random -> 在设置了过期时间的数据中随机淘汰
# allkeys-random -> 在所有数据中随机淘汰
# volatile-ttl -> 越早过期的数据 越先被淘汰
# noeviction -> 不淘汰任何数据，当内存不够时直接抛出异常
```

### Redis 缓存失效策略

**定时过期策略**

每个设置过期时间的 key 都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，对内存很友好，有效地减少了因为过期键带来的内存浪费；但是会占用大量的 CPU 资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。

**惰性过期策略**

只有当访问一个 key 时，才会判断该 key 是否已过期，过期则清除。该策略可以最大化地节省 CPU 资源，却对内存非常不友好。极端情况可能出现大量的过期 key 没有再次被访问，从而不会被清除，占用大量内存。

**定期过期策略**

每隔一定的时间，会扫描一定数量的数据库的 expires 字典中一定数量的 key，并清除其中已过期的 key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得 CPU 和内存资源达到最优的平衡效果。

**Redis中同时使用了定期过期和惰性过期两种过期策略。**

所谓定期删除，指的是 Redis 默认是每隔 100ms 就随机抽取一些设置了过期时间的 key，检查其是否过期，如果过期就删除。

**假设 Redis 里放了 10w 个 key，都设置了过期时间，**你每隔几百毫秒，就检查 10w 个 key，那 Redis 基本上就死了，cpu 负载会很高的，消耗在你的检查过期 key 上了。**注意，**这里可不是每隔 100ms 就遍历所有的设置过期时间的 key，那样就是一场性能上的灾难。实际上 Redis 是每隔 100ms 随机抽取一些 key 来检查和删除的。

**但是问题是，定期删除可能会导致很多过期 key 到了时间并没有被删除掉，那咋整呢？**所以就是惰性删除了。这就是说，在你获取某个 key 的时候，Redis 会检查一下 ，这个 key 如果设置了过期时间那么是否过期了？如果过期了此时就会删除，不会给你返回任何东西。

**获取 key 的时候，如果此时 key 已经过期，就删除，不会返回任何东西。**但是实际上这还是有问题的，如果定期删除漏掉了很多过期 key，然后你也没及时去查，也就没走惰性删除，此时会怎么样？如果大量过期 key 堆积在内存里，导致 Redis 内存快耗尽了，咋整？**答案是：走内存淘汰机制。**

***

## Redis分布式并发问题

> 单体项目中，购物车下单可以使用 synchronized 关键字锁住实例对象，只让一个线程下单并扣减库存
>
> 而对于集群而言，在每台服务器上锁住实例对象，但是每台之间是无法感知的，仍然可能存在商品超卖问题

### 使用 Redis 实现分布式锁

```java
@Transactional
@Override
public Map<String, String> addOrder(String cids, Orders orders) {
    log.info("add order begin...");

    Map<String, String> map = new HashMap<>();

    // 1、根据cids查询当前订单中关联的购物车记录详情（包括库存）
    String[] arr = cids.split("，");
    List<Integer> cidsList = new ArrayList<>();
    for (String s : arr) {
        cidsList.add(Integer.parseInt(s));
    }
    // 获取所有skuId
    List<ShoppingCartVO> list1 = shoppingCartMapper.selectShopCartByCids(cidsList);

    boolean allLocked = true;
    List<String> lockedSkuIds = new ArrayList<>();
    Map<String, String> values = new HashMap<>();
    for (ShoppingCartVO sc : list1) {
        String skuId = sc.getSkuId()， value = UUID.randomUUID().toString();
        Boolean thisLocked = srt.boundValueOps(sc.getSkuId()).setIfAbsent(value, 10, TimeUnit.SECONDS);
        if (!thisLocked) {   // 返回 false 说明之前有该数据，即有线程在操作这个 商品库存
            allLocked = false;  // 因此这个sku就没有锁住，不能继续操作
            break;
        } else {
            lockedSkuIds.add(skuId);
            values.put(skuId, value);
        }
    }
    if (allLocked) {
        // 2、校验库存
        List<ShoppingCartVO> list = shoppingCartMapper.selectShopCartByCids(cidsList);
        boolean f = true;
        StringBuilder sb = new StringBuilder();
        for (ShoppingCartVO sc : list) {
            if (Integer.parseInt(sc.getCartNum()) > sc.getSkuStock()) {
                f = false;
            }
            //获取所有商品的名称，以逗号拼接成字符串
            sb.append(sc.getProductName()).append("，");
        }
        String untitled = sb.toString();

        if (f) {
            log.info("product stock is ok ... ");
            // 3、表示库存充足----保存订单
            // a.userId   b.untitled  c.收货人信息：姓名、电话、地址
            // d.总价格   e.支付方式   f.订单的创建时间  g.订单初始状态
            orders.setUntitled(untitled);
            orders.setCreateTime(new Date());
            orders.setStatus("1");

            // 生成订单编号
            String orderId = UUID.randomUUID().toString().replace("-"， "");
            orders.setOrderId(orderId);

            // 保存订单
            ordersMapper.insert(orders);
            // 4、生成商品快照
            for (ShoppingCartVO sc : list) {
                int cnum = Integer.parseInt(sc.getCartNum());
                String itemId = System.currentTimeMillis() + "" + (new Random().nextInt(89999) + 10000);
                OrderItem orderItem = new OrderItem(itemId, orderId, sc.getProductId()， sc.getProductName()， sc.getProductImg()， sc.getSkuId()，
                                                    sc.getSkuName()， BigDecimal.valueOf(sc.getSellPrice())， cnum, new BigDecimal(sc.getSellPrice() * cnum)， new Date()， new Date()， 0);
                orderItemMapper.insert(orderItem);
            }

            // 5、扣减库存：根据套餐Id修改套餐库存量
            for (ShoppingCartVO sc : list) {
                String skuId = sc.getSkuId();
                int newStock = sc.getSkuStock() - Integer.parseInt(sc.getCartNum());

                ProductSku productSku = new ProductSku();
                productSku.setSkuId(skuId);
                productSku.setStock(newStock);
                //根据主键来修改属性值
                productSkuMapper.updateByPrimaryKeySelective(productSku);
            }

            // 6、删除购物车：当购物车中的记录购买成功之后，购物车中对应做删除操作
            for (int cid : cidsList) {
                shoppingCartMapper.deleteByPrimaryKey(cid);
            }

            // 释放锁
            for (String lockedSkuId : lockedSkuIds) {
                String value = srt.boundValueOps(lockedSkuId).get();
                if (value != null && value.equals(values.get(lockedSkuId))) {
                    srt.delete(lockedSkuId);
                }
            }

            log.info("add order finished... ");

            map.put("orderId"， orderId);
            map.put("productNames"， untitled);
            return map;
        } else {
            // 表示库存不足
            return null;
        }
    } else {    // 加锁失败
        // 要将部分锁定的sku释放锁，即从Redis中移除锁住的skuId
        for (String lockedSkuId : lockedSkuIds) {
            srt.delete(lockedSkuId);
        }
        return null;
    }
}
```

1. 如果订单中部分商品加锁成功，但是由于一些商品加锁失败导致无法提交订单，此时要注意将加锁的商品恢复，即从 Redis 中移除该商品

	> 在 allLocked == false 中，释放加锁的商品

2. 第一次根据 购物车 Id 查到的购物车记录（包含商品库存）到加锁提交订单之前，可能有别的线程 操作了其中某个商品的库存并操作完了，为了防止当前线程还对之前库存进行修改，应当在加锁后操作前重新查询库存。

3. 当前线程加锁成功之后，在执行添加订单的过程中如果出现了异常导致无法释放锁

	> 可以在对商品加锁时设置过期时间，则到期自动释放锁

4. 档给锁设置了过期时间之后，如果当前线程 t1 因为特殊原因，在锁过期之前没有完成业务执行，释放掉了锁，此时线程 t2 加锁成功，如果线程 t1 执行结束，将会释放 线程 t2 的锁，导致 线程 t2 在无锁状态

	> 对于加成功并提交完订单之后，对当前商品的锁释放，要判断 Redis 中的value是否和自己记录的相同，即是否是自己加的锁
	>
	> ```java
	> // 释放锁
	> for (String lockedSkuId : lockedSkuIds) {
	>    String value = srt.boundValueOps(lockedSkuId).get();
	>    if (values.get(lockedSkuId).equals(value)) {
	>      	srt.delete(lockedSkuId);
	>    }
	> }
	> ```

5. 当前线程 t1 判断是自己加的锁，在准备删除之前，锁过期了并且被其他线程 t2 拿到了成功加锁，这时候线程 t1 会释放掉线程 t2 的锁（并发量很大仍然需要考虑）

	> 保证 查询锁是否是自己加的和删除自己加的锁 的原子性（使用 lua 脚本，让多个 Redis 操作具有原子性，查询+删除）

6. 使用看门狗线程解决线程 t1 释放相乘 t2 的锁的问题

	> 线程t1 加锁并设置过期时间，此时启动 看门狗线程（守护线程）监控线程 t1过期时间，过期时间要到了但是线程t1 业务还在执行，则重置过期时间，这样可以保证业务执行完之前不会释放锁

### 分布式锁框架 Resisson

- 添加依赖

	```xml
	<!-- https://mvnrepository.com/artifact/org.redisson/redisson -->
	<dependency>
	    <groupId>org.redisson</groupId>
	    <artifactId>redisson</artifactId>
	    <version>3.16.7</version>
	</dependency>
	```

- 配置 yaml 文件

	```yaml
	redisson:
	  addr:
	    singleAddr:
	      host: Redis://47.110.247.63:6389
	      password: FYX123fyx.
	      database: 0
	```

- 配置 RedissonClient（也可以配置成集群模式、主从模式）

	```java
	@Configuration
	public class RedissonConfig {
	    @Value("${redisson.addr.singleAddr.host}")
	    private String host;
	
	    @Value("${redisson.addr.singleAddr.password}")
	    private String password;
	
	    @Value("${redisson.addr.singleAddr.database}")
	    private int database;
	
	    @Bean
	    public RedissonClient redissonClient() {
	        Config config = new Config();
	        config.useSingleServer().setAddress(host)
	                .setPassword(password)
	                .setDatabase(database);
	        return Redisson.create(config);
	    }
	}
	```

- 操作

	```java
	@Transactional
	@Override
	public Map<String, String> addOrder(String cids, Orders orders) {
	    log.info("add order begin...");
	
	    // 1、根据cids查询当前订单中关联的购物车记录详情（包括库存）
	    String[] arr = cids.split("，");
	    List<Integer> cidsList = new ArrayList<>();
	    for (String s : arr) {
	        cidsList.add(Integer.parseInt(s));
	    }
	    // 获取所有skuId
	    List<ShoppingCartVO> list = shoppingCartMapper.selectShopCartByCids(cidsList);
	    List<String> lockedSkuIds = new ArrayList<>();
	    Map<String, RLock> locks = new HashMap<>(); // {"1":lock1, "2":lock2}
	    boolean allLocked = true;
	    for (ShoppingCartVO sc : list) {
	        String skuId = sc.getSkuId();
	        boolean b = false;
	        RLock lock = redissonClient.getLock(skuId);
	        try {
	            b = lock.tryLock(10, 3, TimeUnit.SECONDS);
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	        if (!b) {
	            allLocked = false;
	            break;
	        } else {
	            lockedSkuIds.add(skuId);
	            locks.put(skuId, lock);
	        }
	    }
	
	    Map<String, String> map = null;
	    try {
	        if (allLocked) {
	            // 2、校验库存
	            list = shoppingCartMapper.selectShopCartByCids(cidsList);
	            boolean f = true;
	            StringBuilder sb = new StringBuilder();
	            for (ShoppingCartVO sc : list) {
	                if (Integer.parseInt(sc.getCartNum()) > sc.getSkuStock()) {
	                    f = false;
	                }
	                //获取所有商品的名称，以逗号拼接成字符串
	                sb.append(sc.getProductName()).append("，");
	            }
	            String untitled = sb.toString();
	
	            if (f) {
	                log.info("product stock is ok ... ");
	                // 3、表示库存充足----保存订单
	                // a.userId   b.untitled  c.收货人信息：姓名、电话、地址
	                // d.总价格   e.支付方式   f.订单的创建时间  g.订单初始状态
	                orders.setUntitled(untitled);
	                orders.setCreateTime(new Date());
	                orders.setStatus("1");
	
	                // 生成订单编号
	                String orderId = UUID.randomUUID().toString().replace("-"， "");
	                orders.setOrderId(orderId);
	
	                // 保存订单
	                ordersMapper.insert(orders);
	                // 4、生成商品快照
	                for (ShoppingCartVO sc : list) {
	                    int cnum = Integer.parseInt(sc.getCartNum());
	                    String itemId = System.currentTimeMillis() + "" + (new Random().nextInt(89999) + 10000);
	                    OrderItem orderItem = new OrderItem(itemId, orderId, sc.getProductId()， sc.getProductName()， sc.getProductImg()， sc.getSkuId()，
	                                                        sc.getSkuName()， BigDecimal.valueOf(sc.getSellPrice())， cnum, new BigDecimal(sc.getSellPrice() * cnum)， new Date()， new Date()， 0);
	                    orderItemMapper.insert(orderItem);
	                }
	
	                // 5、扣减库存：根据套餐Id修改套餐库存量
	                for (ShoppingCartVO sc : list) {
	                    String skuId = sc.getSkuId();
	                    int newStock = sc.getSkuStock() - Integer.parseInt(sc.getCartNum());
	
	                    ProductSku productSku = new ProductSku();
	                    productSku.setSkuId(skuId);
	                    productSku.setStock(newStock);
	                    //根据主键来修改属性值
	                    productSkuMapper.updateByPrimaryKeySelective(productSku);
	                }
	
	                // 6、删除购物车：当购物车中的记录购买成功之后，购物车中对应做删除操作
	                for (int cid : cidsList) {
	                    shoppingCartMapper.deleteByPrimaryKey(cid);
	                }
	
	                log.info("add order finished... ");
	
	                map = new HashMap<>();
	                map.put("orderId"， orderId);
	                map.put("productNames"， untitled);
	
	            }
	
	        }
	    } catch (Exception e) {
	        e.printStackTrace();
	    } finally {
	        // 释放锁
	        for (String lockedSkuId : lockedSkuIds) {
	            locks.get(lockedSkuId).unlock();
	        }
	    }
	    return map;
	}
	```

### 分布式锁特点

1. 互斥性

	保证在不同节点间不同线程的互斥（只有一个节点当然可以）

2. 可重入性

	同一个节点上的同一个线程如果获取锁之后可以再次获取到这个锁

3. 锁超时

	加锁成功之后设置超时时间，以防止线程故障导致不释放锁而引起死锁

4. 高效、高可用

	支持单节点 Redis、主从 Redis、集群 Redis

5. 支持阻塞和非阻塞

	tryLock 非阻塞 和 tryLock(long timeOut) 阻塞

### Redisson 的使用

1. 获取锁（公平锁 / 非公平锁）

	```java
	// 公平锁
	RLock lock = redissonClient.getFairLock(skuId);
	
	// 非公平锁
	RLock lock = redissonClient.getLock(skuId);
	```

2. 加锁（阻塞锁 / 非阻塞锁）

	```java
	// 阻塞锁，设置加锁成功后的超时时间 20s，不加时间默认 30s
	lock.lock(20, TimeUnit.SECONDS);
	
	// 非阻塞锁（设置等待时间 3s，加锁成功后的超时时间 10s，不加则默认 30s）
	lock.tryLock(10, 3, TimeUnit.SECONDS);
	```

3. 释放锁

	```java
	lock.unlock();
	```

4. 实例

	```java
	// 公平非阻塞锁
	RLock lock = redissonClient.getLock(skuId);
	lock.tryLock(10, 3, TimeUnit.SECONDS);
	```

***

## 缓存与数据库双写一致性

总结：

读数据：先读缓存，如果有直接返回，如果没有查数据库查到数据存到缓存中并返回

写数据：先写数据库，再删缓存，这个阶段可能删缓存失败，可以考虑使用一下三种方案：

1. 如果删除缓存失败，需要将数据写入重试表，然后使用 elastic-job 等定时任务进行重试。
2. 如果删除缓存失败，需要将数据发送 mq 消息到 mq 服务器，在 mq 的 consumer 中处理。
3. 订阅 mysql 的 binlog，在订阅者中，如果发现了更新数据请求，则删除相应的缓存。

> [如何保证数据库和缓存双写一致性？-苏三说技术](https://www.zhihu.com/question/319817091/answer/2432904728)

### 问题引入

通常情况下，我们使用缓存的主要目的是为了提升查询的性能。大多数情况下，我们是这样使用缓存的：

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650526232523.png)

1. 用户请求过来之后，先查缓存有没有数据，如果有则直接返回。
2. 如果缓存没数据，再继续查数据库。
3. 如果数据库有数据，则将查询出来的数据，放入缓存中，然后返回该数据。
4. 如果数据库也没数据，则直接返回空。

这是缓存非常常见的用法。一眼看上去，好像没有啥问题。

但你忽略了一个非常重要的细节：**如果数据库中的某条数据，放入缓存之后，又立马被更新了，那么该如何更新缓存呢？**

不更新缓存行不行？

答：当然不行，如果不更新缓存，在很长的一段时间内（决定于缓存的过期时间），用户请求从缓存中获取到的都可能是旧值，而非数据库的最新值。这不是有数据不一致的问题？

那么，我们该如何更新缓存呢？

目前有以下 4 种方案：

1. 先写缓存，再写数据库
2. 先写数据库，再写缓存
3. 先删缓存，再写数据库
4. 先写数据库，再删缓存

接下来，我们详细说说这 4 种方案。

### 先写缓存，再写数据库

对于更新缓存的方案，很多人第一个想到的可能是在写操作中直接更新缓存（写缓存），更直接明了。

那么，问题来了：在写操作中，到底是先写缓存，还是先写数据库呢？

我们在这里先聊聊先写缓存，再写数据库的情况，因为它的问题最严重。

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650526427819.png)

某一个用户的每一次写操作，如果刚写完缓存，突然网络出现了异常，导致写数据库失败了。

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650526436447.png)

其结果是缓存更新成了最新数据，但数据库没有，这样缓存中的数据不就变成脏数据了？如果此时该用户的查询请求，正好读取到该数据，就会出现问题，因为该数据在数据库中根本不存在，这个问题非常严重。

我们都知道，缓存的主要目的是把数据库的数据临时保存在内存，便于后续的查询，提升查询速度。

但如果某条数据，在数据库中都不存在，你缓存这种“`假数据`”又有啥意义呢？

因此，先写缓存，再写数据库的方案是不可取的，在实际工作中用得不多。

### 先写数据库，再写缓存

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650526499487.png)

用户的写操作，先写数据库，再写缓存，可以避免之前“假数据”的问题。但它却带来了新的问题。

#### 写缓存失败了

如果把写数据库和写缓存操作，放在同一个事务当中，当写缓存失败了，我们可以把写入数据库的数据进行回滚。

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650526558901.png)

如果是并发量比较小，对接口性能要求不太高的系统，可以这么玩。

但如果在高并发的业务场景中，写数据库和写缓存，都属于远程操作。为了防止出现大事务，造成的死锁问题，通常建议写数据库和写缓存不要放在同一个事务中。

也就是说在该方案中，如果写数据库成功了，但写缓存失败了，数据库中已写入的数据不会回滚。

这就会出现：数据库是`新数据`，而缓存是`旧数据`，两边`数据不一致`的情况。

#### 高并发下的问题

假设在高并发的场景中，针对同一个用户的同一条数据，有两个写数据请求：a 和 b，它们同时请求到业务系统。

其中请求 a 获取的是旧数据，而请求 b 获取的是新数据，如下图所示：

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650526645914.png)

1. 请求 a 先过来，刚写完了数据库。但由于网络原因，卡顿了一下，还没来得及写缓存。
2. 这时候请求 b 过来了，先写了数据库。
3. 接下来，请求 b 顺利写了缓存。
4. 此时，请求 a 卡顿结束，也写了缓存。

很显然，在这个过程当中，请求 b 在缓存中的`新数据`，被请求 a 的`旧数据`覆盖了。

也就是说：在高并发场景中，如果多个线程同时执行先写数据库，再写缓存的操作，可能会出现数据库是新值，而缓存中是旧值，两边数据不一致的情况。

#### 浪费系统资源

有些业务场景比较特殊：`写多读少`。

如果在这类业务场景中，每个用的写操作，都需要写一次缓存，有点得不偿失。

还不如不用 Redis，反正写数据库都要写，我读数据又比较少，Redis 使用频率低，却还要一直更新 Redis 中数据。

### 先删缓存，再写数据库

不然是先写缓存还是先写数据库都有很多问题，那么考虑不是 `更新缓存` 而是 `删除缓存`。

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650526855730.png)

在用户的写操作中，先执行删除缓存操作，再去写数据库

#### 高并发下的问题

假设在高并发的场景中，同一个用户的同一条数据，有一个读数据请求 c，还有另一个写数据请求 d（一个更新操作），同时请求到业务系统。如下图所示：

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650527043982.png)

1. 请求 d 先过来，把缓存删除了。但由于网络原因，卡顿了一下，还没来得及写数据库。
2. 这时请求 c 过来了，先查缓存发现没数据，再查数据库，有数据，但是旧值。
3. 请求 c 将数据库中的旧值，更新到缓存中。
4. 此时，请求 d 卡顿结束，把新值写入数据库。

在这个过程当中，请求 d 的新值并没有被请求 c 写入缓存，同样会导致缓存和数据库的数据不一致的情况。

#### 缓存双删

在上面的业务场景中，一个读数据请求，一个写数据请求。当写数据请求把缓存删了之后，读数据请求，可能把当时从数据库查询出来的旧值，写入缓存当中。

有人说还不好办，请求 d 在写完数据库之后，把缓存重新删一次不就行了？

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650528097218.png)

这就是我们所说的`缓存双删`，即在写数据库之前删除一次，写完数据库后，再删除一次。

该方案有个非常关键的地方是：第二次删除缓存，并非立马就删，而是要在一定的`时间间隔`之后。

我们再重新回顾一下，高并发下一个读数据请求，一个写数据请求导致数据不一致的产生过程：

1. 请求 d 先过来，把缓存删除了。但由于网络原因，卡顿了一下，还没来得及写数据库。
2. 这时请求 c 过来了，先查缓存发现没数据，再查数据库，有数据，但是旧值。
3. 请求 c 将数据库中的旧值，更新到缓存中。
4. 此时，请求 d 卡顿结束，把新值写入数据库。
5. 一段时间之后，比如：500ms，请求 d 将缓存删除。

这样来看确实可以解决缓存不一致问题。

那么，为什么一定要间隔一段时间之后，才能删除缓存呢？

请求 d 卡顿结束，把新值写入数据库后，请求 c 将数据库中的旧值，更新到缓存中。

此时，如果请求 d 删除太快，在请求 c 将数据库中的旧值更新到缓存之前，就已经把缓存删除了，这次删除就没任何意义。必须要在请求 c 更新缓存之后，再删除缓存，才能把旧值及时删除了。

所以需要在请求 d 中加一个时间间隔，确保请求 c，或者类似于请求 c 的其他请求，如果在缓存中设置了旧值，最终都能够被请求 d 删除掉。

接下来，还有一个问题：如果第二次删除缓存时，删除失败了该怎么办？

### 先写数据库，再删缓存

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650528137396.png)

在高并发的场景中，有一个读数据请求，有一个写数据请求，更新过程如下：

1. 请求 e 先写数据库，由于网络原因卡顿了一下，没有来得及删除缓存。
2. 请求 f 查询缓存，发现缓存中有数据，直接返回该数据。
3. 请求 e 删除缓存。

在这个过程中，只有请求 f 读了一次旧数据，后来旧数据被请求 e 及时删除了，看起来问题不大。

但如果是读数据请求先过来呢？

1. 请求 f 查询缓存，发现缓存中有数据，直接返回该数据。
2. 请求 e 先写数据库。
3. 请求 e 删除缓存。

这种情况看起来也没问题呀？

答：对的。

但就怕出现下面这种情况，即缓存自己失效了。如下图所示：

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650528267733.png)

1. 缓存过期时间到了，自动失效。
2. 请求 f 查询缓存，发缓存中没有数据，查询数据库的旧值，但由于网络原因卡顿了，没有来得及更新缓存。
3. 请求 e 先写数据库，接着删除了缓存。
4. 请求 f 更新旧值到缓存中。

这时，缓存和数据库的数据同样出现不一致的情况了。

但这种情况还是比较少的，需要同时满足以下条件才可以：

1. 缓存刚好自动失效。
2. 请求 f 从数据库查出旧值，更新缓存的耗时，比请求 e 写数据库，并且删除缓存的还长。

我们都知道查询数据库的速度，一般比写数据库要快，更何况写完数据库，还要删除缓存。所以绝大多数情况下，写数据请求比读数据情况耗时更长。

由此可见，系统同时满足上述两个条件的概率非常小。

> 推荐大家使用先写数据库，再删缓存的方案，虽说不能100%避免数据不一致问题，但出现该问题的概率，相对于其他方案来说是最小的。

但在该方案中，如果删除缓存失败了该怎么办呢？

### 删缓存失败怎么办？

其实先写数据库，再删缓存的方案，跟缓存双删的方案一样，有一个共同的风险点，即：如果缓存删除失败了，也会导致缓存和数据库的数据不一致。

那么，删除缓存失败怎么办呢？

答：需要加`重试机制`。

在接口中如果更新了数据库成功了，但更新缓存失败了，可以立刻重试 3 次。如果其中有任何一次成功，则直接返回成功。如果 3 次都失败了，则写入数据库，准备后续再处理。

当然，如果你在接口中直接`同步重试`，该接口并发量比较高的时候，可能有点影响接口性能。

这时，就需要改成`异步重试`了。

异步重试方式有很多种，比如：

1. 每次都单独起一个线程，该线程专门做重试的工作。但如果在高并发的场景下，可能会创建太多的线程，导致系统 OOM 问题，不太建议使用。
2. 将重试的任务交给线程池处理，但如果服务器重启，部分数据可能会丢失。
3. 将重试数据写表，然后使用 elastic-job 等定时任务进行重试。
4. 将重试的请求写入 mq 等消息中间件中，在 mq 的 consumer 中处理。
5. 订阅 mysql 的 binlog，在订阅者中，如果发现了更新数据请求，则删除相应的缓存。

#### 定时任务

使用`定时任务重试`的具体方案如下：

1. 当用户操作写完数据库，但删除缓存失败了，需要将用户数据写入重试表中。如下图所示：

	![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650528474531.png)

2. 在定时任务中，异步读取重试表中的用户数据。重试表需要记录一个重试次数字段，初始值为 0。然后重试 5 次，不断删除缓存，每重试一次该字段值+1。如果其中有任意一次成功了，则返回成功。如果重试了 5 次，还是失败，则我们需要在重试表中记录一个失败的状态，等待后续进一步处理。

	![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650528480542.png)

3. 在高并发场景中，定时任务推荐使用`elastic-job`。相对于 xxl-job 等定时任务，它可以分片处理，提升处理速度。同时每片的间隔可以设置成：1,2,3,5,7 秒等。

使用定时任务重试的话，有个缺点就是实时性没那么高，对于实时性要求特别高的业务场景，该方案不太适用。但是对于一般场景，还是可以用一用的。

但它有一个很大的优点，即数据是落库的，不会丢数据。

#### mq

在高并发的业务场景中，mq（消息队列）是必不可少的技术之一。它不仅可以异步解耦，还能削峰填谷。对保证系统的稳定性是非常有意义的。

mq 的生产者，生产了消息之后，通过指定的 topic 发送到 mq 服务器。然后 mq 的消费者，订阅该 topic 的消息，读取消息数据之后，做业务逻辑处理。

使用`mq重试`的具体方案如下：

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650528506474.png)

1. 当用户操作写完数据库，但删除缓存失败了，产生一条 mq 消息，发送给 mq 服务器。
2. mq 消费者读取 mq 消息，重试 5 次删除缓存。如果其中有任意一次成功了，则返回成功。如果重试了 5 次，还是失败，则写入`死信队列`中。
3. 推荐 mq 使用`rocketmq`，重试机制和死信队列默认是支持的。使用起来非常方便，而且还支持顺序消息，延迟消息和事务消息等多种业务场景。

当然在该方案中，删除缓存可以完全走异步。即用户的写操作，在写完数据库之后，不用立刻删除一次缓存。而直接发送 mq 消息，到 mq 服务器，然后有 mq 消费者全权负责删除缓存的任务。

因为 mq 的实时性还是比较高的，因此改良后的方案也是一种不错的选择。

#### binlog

前面我们聊过的，无论是定时任务，还是 mq（消息队列），做重试机制，对业务都有一定的侵入性。

在使用定时任务的方案中，需要在业务代码中增加额外逻辑，如果删除缓存失败，需要将数据写入重试表。

而使用 mq 的方案中，如果删除缓存失败了，需要在业务代码中发送 mq 消息到 mq 服务器。

其实，还有一种更优雅的实现，即`监听binlog`，比如使用：`canal`等中间件。

具体方案如下：

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650528533430.png)

1. 在业务接口中写数据库之后，就不管了，直接返回成功。
2. mysql 服务器会自动把变更的数据写入 binlog 中。
3. binlog 订阅者获取变更的数据，然后删除缓存。

这套方案中业务接口确实简化了一些流程，只用关心数据库操作即可，而在 binlog 订阅者中做缓存删除工作。

但如果只是按照图中的方案进行删除缓存，只删除了一次，也可能会失败。

如何解决这个问题呢？

答：这就需要加上前面聊过的`重试机制`了。如果删除缓存失败，写入重试表，使用定时任务重试。或者写入 mq，让 mq 自动重试。

在这里推荐使用`mq自动重试机制`。

![图片](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes640-1650528538202.png)

在 binlog 订阅者中如果删除缓存失败，则发送一条 mq 消息到 mq 服务器，在 mq 消费者中自动重试 5 次。如果有任意一次成功，则直接返回成功。如果重试 5 次后还是失败，则该消息自动被放入死信队列，后面可能需要人工介入。

***

## Redis 线程模型

### 单线程的Redis为什么这么快

1. 纯内存操作

2. 单线程操作，避免了频繁的上下文切换

3. 采用了非阻塞 I/O 多路复用机制

### Redis 为什么是单线程的

官方 FAQ 表示，因为 Redis 是基于内存的操作，CPU 不是 Redis 的瓶颈，Redis 的瓶颈最有可能是机器内存的大小或者网络带宽。

既然单线程容易实现，而且 CPU 不会成为瓶颈，那就顺理成章地采用单线程的方案了（毕竟采用多线程会有很多麻烦）。Redis 利用队列将并发访问变为串行访问

1、绝大部分请求是纯粹的内存操作（非常快速）

2、采用单线程，避免了不必要的上下文切换和竞争条件

3、非阻塞 IO 优点：

- 速度快，因为数据存在内存中，类似于 HashMap，HashMap 的优势就是查找和操作的时间复杂度都是 O(1)
- 支持丰富数据类型，支持 string，list，set，sorted set，hash
- 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行
- 丰富的特性：可用于缓存，消息，按 key 设置过期时间，过期后将会自动删除如何解决 Redis 的并发竞争 key 问题

**同时有多个子系统去set一个key。这个时候要注意什么呢？**

不推荐使用 Redis 的事务机制。因为我们的生产环境，基本都是 Redis 集群环境，做了数据分片操作。你一个事务中有涉及到多个 key 操作的时候，这多个 key 不一定都存储在同一个 Redis-server 上。因此，Redis 的事务机制，十分鸡肋。

- 如果对这个 key 操作，不要求顺序：准备一个分布式锁，大家去抢锁，抢到锁就做 set 操作即可
- 如果对这个 key 操作，要求顺序：分布式锁+时间戳。假设这会系统 B 先抢到锁，将 key1 设置为{valueB 3:05}。接下来系统 A 抢到锁，发现自己的 valueA 的时间戳早于缓存中的时间戳，那就不做 set 操作了。以此类推。
- 利用队列，将 set 方法变成串行访问也可以 Redis 遇到高并发，如果保证读写 key 的一致性

对 Redis 的操作都是具有原子性的，是线程安全的操作，你不用考虑并发问题,Redis 内部已经帮你处理好并发的问题了。

### 为什么Redis的操作是原子性的，怎么保证原子性的？

对于 Redis 而言，命令的原子性指的是：一个操作的不可以再分，操作要么执行，要么不执行。

Redis 的操作之所以是原子性的，是因为 Redis 是单线程的。（Redis 新版本已经引入多线程，这里基于旧版本的 Redis）

Redis 本身提供的所有 API 都是原子操作，Redis 中的事务其实是要保证批量操作的原子性。

多个命令在并发中也是原子性的吗？

不一定， 将 get 和 set 改成单命令操作，incr 。使用 Redis 的事务，或者使用 Redis+Lua 的方式实现.

### 讲解下Redis线程模型

文件事件处理器包括分别是套接字、 I/O 多路复用程序、 文件事件分派器（dispatcher）、 以及事件处理器。使用 I/O 多路复用程序来同时监听多个套接字， 并根据套接字目前执行的任务来为套接字关联不同的事件处理器。

当被监听的套接字准备好执行连接应答（accept）、读取（read）、写入（write）、关闭（close）等操作时， 与操作相对应的文件事件就会产生， 这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件。

I/O 多路复用程序负责监听多个套接字， 并向文件事件分派器传送那些产生了事件的套接字。

**工作原理：**

I/O 多路复用程序负责监听多个套接字， 并向文件事件分派器传送那些产生了事件的套接字。

尽管多个文件事件可能会并发地出现， 但 I/O 多路复用程序总是会将所有产生事件的套接字都入队到一个队列里面， 然后通过这个队列， 以有序（sequentially）、同步（synchronously）、每次一个套接字的方式向文件事件分派器传送套接字：

当上一个套接字产生的事件被处理完毕之后（该套接字为事件所关联的事件处理器执行完毕）， I/O 多路复用程序才会继续向文件事件分派器传送下一个套接字。如果一个套接字又可读又可写的话， 那么服务器将先读套接字， 后写套接字。

## 其他参考

[原创 一洺 [阿里开发者](javascript:void(0);) 2022-03-24 08:00](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247507942&idx=1&sn=e00fd16ec56d0e10cc830678ee25c417&chksm=e92ae2e9de5d6bff6d227646c12685a5c45a4c3322e2186954555a14ebcd9510e8ae579a4941&mpshare=1&scene=24&srcid=0324gng7YtX5XX0z9bf7aObR&sharer_sharetime=1648080975816&sharer_shareid=ebfad4e5798246e3d3928c1f4ee9048e#rd)

https://blog.csdn.net/qq_31387317/category_7391502.html
