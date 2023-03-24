---
title: "Linux 常用命令"
date: 2023-02-19 12:02:02
categories: ["技术总结"]
tags: ["Linux"]
url: linux-command

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
# tags: []
author: "Yaxing"

comment: false
postMetaInFooter: false
hiddenFromHomePage: false
contentCopyright: true
---

本文对 Linux 常用命令做简单小记。<!--more-->

## 文件管理命令

### cat 命令

cat 命令用于连接文件并打印到标准输出设备上。

cat 主要有三大功能：

1、一次显示整个文件:

```sh
cat filename
```

2、从键盘创建一个文件:

```shell
cat > filename
```

只能创建新文件，不能编辑已有文件。

3、将几个文件合并为一个文件：

```shell
cat file1 file2 > file
```

- -b 对非空输出行号
- -n 输出所有行号

**实例**：

（1）把 log2012.log 的文件内容加上行号后输入 log2013.log 这个文件里

```shell
cat -n log2012.log > log2013.log
```

（2）把 log2012.log 和 log2013.log 的文件内容加上行号（空白行不加）之后将内容附加到 log.log 里

```shell
cat -b log2012.log log2013.log > log.log
```

### chmod 命令

Linux/Unix 的文件调用权限分为三级 : 文件拥有者、群组、其他。利用 chmod 可以控制文件如何被他人所调用。

用于改变 linux 系统文件或目录的访问权限。用它控制文件或目录的访问权限。该命令有两种用法。一种是包含字母和操作符表达式的文字设定法；另一种是包含数字的数字设定法。

每一文件或目录的访问权限都有三组，每组用三位表示，分别为文件**属主**的读、写和执行权限；与属主**同组**的用户的读、写和执行权限；系统中**其他用户**的读、写和执行权限。可使用 ls -l test.txt 查找。

以文件 log2012.log 为例：

```shell
-rw-r--r-- 1 root root 296K 11-13 06:03 log2012.log
```

第一列共有 10 个位置，第一个字符指定了文件类型。在通常意义上，一个目录也是一个文件。如果第一个字符是横线，表示是一个非目录的文件。如果是 d，表示是一个目录。从第二个字符开始到第十个 9 个字符，3 个字符一组，分别表示了 3 组用户对文件或者目录的权限。权限字符用横线代表空许可，r 代表只读(4)，w 代表写(2)，x 代表可执行(1)。

**常用参数**：

```
-c 当发生改变时，报告处理信息
-R 处理指定目录以及其子目录下所有文件
```

权限范围：

```
u ：目录或者文件的当前的用户
g ：目录或者文件的当前的群组
o ：除了目录或者文件的当前用户或群组之外的用户或者群组
a ：所有的用户及群组
```

权限代号：

```
r ：读权限，用数字4表示
w ：写权限，用数字2表示
x ：执行权限，用数字1表示
- ：删除权限，用数字0表示
s ：特殊权限
```

**实例**：

（1）增加文件 t.log 所有用户可执行权限

```shell
chmod a+x t.log
```

（2）撤销原来所有的权限，然后使拥有者具有可读权限,并输出处理信息

```shell
chmod u=r t.log -c
```

（3）给 file 的属主分配读、写、执行(7)的权限，给 file 的所在组分配读、执行(5)的权限，给其他用户分配执行(1)的权限

```shell
chmod 751 t.log -c（或者：chmod u=rwx,g=rx,o=x t.log -c
```

（4）将 test 目录及其子目录所有文件添加可读权限

```shell
chmod u+r,g+r,o+r -R text/ -c
```

### chown 命令

chown 将指定文件的拥有者改为指定的用户或组，用户可以是用户名或者用户 ID；组可以是组名或者组 ID；文件是以空格分开的要改变权限的文件列表，支持通配符。

```
-c 显示更改的部分的信息
-R 处理指定目录及子目录
```

**实例**：

（1）改变拥有者和群组 并显示改变信息

```shell
chown -c mail:mail log2012.log
```

（2）改变文件群组

```shell
chown -c :mail t.log
```

（3）改变文件夹及子文件目录属主及属组为 mail

```shell
chown -cR mail: test/
```

### cp 命令

将源文件复制至目标文件，或将多个源文件复制至目标目录。

注意：命令行复制，如果目标文件已经存在会提示是否覆盖，而在 shell 脚本中，如果不加 -i 参数，则不会提示，而是直接覆盖！

```
-i 提示
-r 复制目录及目录内所有项目
-a 复制的文件与原文件时间一样
```

**实例**：

（1）复制 a.txt 到 test 目录下，保持原文件时间，如果原文件存在提示是否覆盖。

```shell
cp -ai a.txt test
```

（2）为 a.txt 建议一个链接（快捷方式）

```shell
cp -s a.txt link_a.txt
```

### find 命令

用于在文件树中查找文件，并作出相应的处理。

命令格式：

```shell
find pathname -options [-print -exec -ok ...]
```

命令参数：

```
pathname: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
-print： find命令将匹配的文件输出到标准输出。
-exec： find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {  } \;，注意{   }和\；之间的空格。
-ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。
```

**命令选项**：

```
-name 按照文件名查找文件
-perm 按文件权限查找文件
-user 按文件属主查找文件
-group  按照文件所属的组来查找文件。
-type  查找某一类型的文件，诸如：
   b - 块设备文件
   d - 目录
   c - 字符设备文件
   l - 符号链接文件
   p - 管道文件
   f - 普通文件
```

**实例**：

（1）查找 48 小时内修改过的文件

```shell
find -atime -2
```

（2）在当前目录查找 以 .log 结尾的文件。 **.** 代表当前目录

```shell
find ./ -name '*.log'
```

（3）查找 /opt 目录下 权限为 777 的文件

```shell
find /opt -perm 777
```

（4）查找大于 1K 的文件

```shell
find -size +1000c
```

查找等于 1000 字符的文件

```shell
find -size 1000c 
```

-exec 参数后面跟的是 command 命令，它的终止是以 ; 为结束标志的，所以这句命令后面的分号是不可缺少的，考虑到各个系统中分号会有不同的意义，所以前面加反斜杠。{} 花括号代表前面 find 查找出来的文件名。

### head 命令

head 用来显示档案的开头至标准输出中，默认 head 命令打印其相应文件的开头 10 行。

**常用参数**：

```
-n<行数> 显示的行数（行数为复数表示从最后向前数）
```

**实例**：

（1）显示 1.log 文件中前 20 行

```shell
head 1.log -n 20
```

（2）显示 1.log 文件前 20 字节

```shell
head 1.log -c 20 
1
```

（3）显示 1.log 最后 10 行

```shell
head 1.log -n -10
```

### less 命令

less 与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动，而且 less 在查看之前不会加载整个文件。

**常用命令参数**：

```
-i  忽略搜索时的大小写
-N  显示每行的行号
-o  <文件名> 将less 输出的内容在指定文件中保存起来
-s  显示连续空行为一行
/字符串：向下搜索“字符串”的功能
?字符串：向上搜索“字符串”的功能
n：重复前一个搜索（与 / 或 ? 有关）
N：反向重复前一个搜索（与 / 或 ? 有关）
-x <数字> 将“tab”键显示为规定的数字空格
b  向后翻一页
d  向后翻半页
h  显示帮助界面
Q  退出less 命令
u  向前滚动半页
y  向前滚动一行
空格键 滚动一行
回车键 滚动一页
[pagedown]： 向下翻动一页
[pageup]：   向上翻动一页
```

**实例**：

（1）ps 查看进程信息并通过 less 分页显示

```shell
ps -aux | less -N
```

（2）查看多个文件

```shell
less 1.log 2.log
```

可以使用 n 查看下一个，使用 p 查看前一个。

### ln 命令

功能是为文件在另外一个位置建立一个同步的链接，当在不同目录需要该问题时，就不需要为每一个目录创建同样的文件，通过 ln 创建的链接（link）减少磁盘占用量。

链接分类：软件链接及硬链接

软链接：

- 1.软链接，以路径的形式存在。类似于 Windows 操作系统中的快捷方式
- 2.软链接可以 跨文件系统 ，硬链接不可以
- 3.软链接可以对一个不存在的文件名进行链接
- 4.软链接可以对目录进行链接

硬链接:

- 1.硬链接，以文件副本的形式存在。但不占用实际空间。
- 2.不允许给目录创建硬链接
- 3.硬链接只有在同一个文件系统中才能创建

**需要注意**：

- 第一：ln 命令会保持每一处链接文件的同步性，也就是说，不论你改动了哪一处，其它的文件都会发生相同的变化；
- 第二：ln 的链接又分软链接和硬链接两种，软链接就是 ln –s 源文件 目标文件，它只会在你选定的位置上生成一个文件的镜像，不会占用磁盘空间，硬链接 ln 源文件 目标文件，没有参数-s， 它会在你选定的位置上生成一个和源文件大小相同的文件，无论是软链接还是硬链接，文件都保持同步变化。
- 第三：ln 指令用在链接文件或目录，如同时指定两个以上的文件或目录，且最后的目的地是一个已经存在的目录，则会把前面指定的所有文件或目录复制到该目录中。若同时指定多个文件或目录，且最后的目的地并非是一个已存在的目录，则会出现错误信息。

**常用参数**：

```
-b 删除，覆盖以前建立的链接
-s 软链接（符号链接）
-v 显示详细处理过程
```

**实例**：

（1）给文件创建软链接，并显示操作信息

```shell
ln -sv source.log link.log
```

（2）给文件创建硬链接，并显示操作信息

```shell
ln -v source.log link1.log
```

（3）给目录创建软链接

```shell
ln -sv /opt/soft/test/test3 /opt/soft/test/test5
```

### locate 命令

locate 通过搜寻系统内建文档数据库达到快速找到档案，数据库由 updatedb 程序来更新，updatedb 是由 cron daemon 周期性调用的。默认情况下 locate 命令在搜寻数据库时比由整个由硬盘资料来搜寻资料来得快，但较差劲的是 locate 所找到的档案若是最近才建立或 刚更名的，可能会找不到，在内定值中，updatedb 每天会跑一次，可以由修改 crontab 来更新设定值 (etc/crontab)。

locate 与 find 命令相似，可以使用如 *、? 等进行正则匹配查找

**常用参数**：

```
-l num（要显示的行数）
-f   将特定的档案系统排除在外，如将proc排除在外
-r   使用正则运算式做为寻找条件
```

**实例**：

（1）查找和 pwd 相关的所有文件(文件名中包含 pwd）

```shell
locate pwd
```

（2）搜索 etc 目录下所有以 sh 开头的文件

```shell
locate /etc/sh
```

（3）查找 /var 目录下，以 reason 结尾的文件

```shell
locate -r '^/var.*reason$'（其中.表示一个字符，*表示任务多个；.*表示任意多个字符）
```

### more 命令

功能类似于 cat, more 会以一页一页的显示方便使用者逐页阅读，而最基本的指令就是按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示。

**命令参数**：

```
+n      从笫 n 行开始显示
-n       定义屏幕大小为n行
+/pattern 在每个档案显示前搜寻该字串（pattern），然后从该字串前两行之后开始显示 
-c       从顶部清屏，然后显示
-d       提示“Press space to continue，’q’ to quit（按空格键继续，按q键退出）”，禁用响铃功能
-l        忽略Ctrl+l（换页）字符
-p       通过清除窗口而不是滚屏来对文件进行换页，与-c选项相似
-s       把连续的多个空行显示为一行
-u       把文件内容中的下画线去掉
```

**常用操作命令**：

```
Enter    向下 n 行，需要定义。默认为 1 行
Ctrl+F   向下滚动一屏
空格键  向下滚动一屏
Ctrl+B  返回上一屏
=       输出当前行的行号
:f     输出文件名和当前行的行号
V      调用vi编辑器
!命令   调用Shell，并执行命令
q       退出more
```

**实例**：

（1）显示文件中从第 3 行起的内容

```shell
more +3 text.txt
```

（2）在所列出文件目录详细信息，借助管道使每次显示 5 行

```shell
ls -l | more -5
```

按空格显示下 5 行。

### mv 命令

移动文件或修改文件名，根据第二参数类型（如目录，则移动文件；如为文件则重命令该文件）。

当第二个参数为目录时，第一个参数可以是多个以空格分隔的文件或目录，然后移动第一个参数指定的多个文件到第二个参数指定的目录中。

**实例**：

（1）将文件 test.log 重命名为 test1.txt

```shell
mv test.log test1.txt
```

（2）将文件 log1.txt、log2.txt、log3.txt 移动到根的 test3 目录中

```shell
mv llog1.txt log2.txt log3.txt /test3
```

（3）将文件 file1 改名为 file2，如果 file2 已经存在，则询问是否覆盖

```shell
mv -i log1.txt log2.txt
```

（4）移动当前文件夹下的所有文件到上一级目录

```shell
mv * ../
```

### rm 命令

删除一个目录中的一个或多个文件或目录，如果没有使用 -r 选项，则 rm 不会删除目录。如果使用 rm 来删除文件，通常仍可以将该文件恢复原状。

```shell
rm [选项] 文件…
```

**实例**：

（1）删除任何 .log 文件，删除前逐一询问确认：

```shell
rm -i *.log
```

（2）删除 test 子目录及子目录中所有档案删除，并且不用一一确认：

```shell
rm -rf test
```

（3）删除以 -f 开头的文件

```shell
rm -- -f*
```

### tail 命令

用于显示指定文件末尾内容，不指定文件时，作为输入信息进行处理。常用查看日志文件。

**常用参数**：

```
-f 循环读取（常用于查看递增的日志文件）
-n<行数> 显示行数（从后向前）
```

（1）循环读取逐渐增加的文件内容

```shell
ping 127.0.0.1 > ping.log &
```

后台运行：可使用 jobs -l 查看，也可使用 fg 将其移到前台运行。

```shell
tail -f ping.log
```

（查看日志）

### touch 命令

Linux touch 命令用于修改文件或者目录的时间属性，包括存取时间和更改时间。若文件不存在，系统会建立一个新的文件。

ls -l 可以显示档案的时间记录。

**语法**

```
touch [-acfm][-d<日期时间>][-r<参考文件或目录>] [-t<日期时间>][--help][--version][文件或目录…]
```

- **参数说明**：
- a 改变档案的读取时间记录。
- m 改变档案的修改时间记录。
- c 假如目的档案不存在，不会建立新的档案。与 --no-create 的效果一样。
- f 不使用，是为了与其他 unix 系统的相容性而保留。
- r 使用参考档的时间记录，与 --file 的效果一样。
- d 设定时间与日期，可以使用各种不同的格式。
- t 设定档案的时间记录，格式与 date 指令相同。
- –no-create 不会建立新档案。
- –help 列出指令格式。
- –version 列出版本讯息。

**实例**

使用指令"touch"修改文件"testfile"的时间属性为当前系统时间，输入如下命令：

```shell
$ touch testfile                #修改文件的时间属性 
```

首先，使用 ls 命令查看 testfile 文件的属性，如下所示：

```shell
$ ls -l testfile                #查看文件的时间属性  
#原来文件的修改时间为16:09  
-rw-r--r-- 1 hdd hdd 55 2011-08-22 16:09 testfile  
```

执行指令"touch"修改文件属性以后，并再次查看该文件的时间属性，如下所示：

```shell
$ touch testfile                #修改文件时间属性为当前系统时间  
$ ls -l testfile                #查看文件的时间属性  
#修改后文件的时间属性为当前系统时间  
-rw-r--r-- 1 hdd hdd 55 2011-08-22 19:53 testfile  
```

使用指令"touch"时，如果指定的文件不存在，则将创建一个新的空白文件。例如，在当前目录下，使用该指令创建一个空白文件"file"，输入如下命令：

```shell
$ touch file            #创建一个名为“file”的新的空白文件 
```

### vim 命令

Vim 是从 vi 发展出来的一个文本编辑器。代码补完、编译及错误跳转等方便编程的功能特别丰富，在程序员中被广泛使用。

- 打开文件并跳到第 10 行：`vim +10 filename.txt` 。
- 打开文件跳到第一个匹配的行：`vim +/search-term filename.txt` 。
- 以只读模式打开文件：`vim -R /etc/passwd` 。

基本上 vi/vim 共分为三种模式，分别是**命令模式（Command mode）**，**输入模式（Insert mode）\**和\**底线命令模式（Last line mode）**。

简单的说，我们可以将这三个模式想成底下的图标来表示：

![vim 工作模式](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/format,png-20230125135806349.png)

### whereis 命令

whereis 命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man 说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。whereis 及 locate 都是基于系统内建的数据库进行搜索，因此效率很高，而 find 则是遍历硬盘查找文件。

**常用参数**：

```
-b   定位可执行文件。
-m   定位帮助文件。
-s   定位源代码文件。
-u   搜索默认路径下除可执行文件、源代码文件、帮助文件以外的其它文件。
```

**实例**：

（1）查找 locate 程序相关文件

```shell
whereis locate
```

（2）查找 locate 的源码文件

```shell
whereis -s locate
```

（3）查找 lcoate 的帮助文件

```shell
whereis -m locate
```

### which 命令

在 linux 要查找某个文件，但不知道放在哪里了，可以使用下面的一些命令来搜索：

```
which     查看可执行文件的位置。
whereis 查看文件的位置。
locate  配合数据库查看文件位置。
find        实际搜寻硬盘查询文件名称。
```

which 是在 PATH 就是指定的路径中，搜索某个系统命令的位置，并返回第一个搜索结果。使用 which 命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。

**常用参数**：

```
-n 　指定文件名长度，指定的长度必须大于或等于所有文件中最长的文件名。
```

**实例**：

（1）查看 ls 命令是否存在，执行哪个

```shell
which ls
```

（2）查看 which

```shell
which which
```

（3）查看 cd

```shell
which cd（显示不存在，因为 cd 是内建命令，而 which 查找显示是 PATH 中的命令）
```

查看当前 PATH 配置：

```shell
echo $PATH
```

或使用 env 查看所有环境变量及对应值

## 文档编辑命令

### grep 命令

强大的文本搜索命令，grep(Global Regular Expression Print) 全局正则表达式搜索。

grep 的工作方式是这样的，它在一个或多个文件中搜索字符串模板。如果模板包括空格，则必须被引用，模板后的所有字符串被看作文件名。搜索的结果被送到标准输出，不影响原文件内容。

命令格式：

```
grep [option] pattern file|dir
```

**常用参数**：

```
-A n --after-context显示匹配字符后n行
-B n --before-context显示匹配字符前n行
-C n --context 显示匹配字符前后n行
-c --count 计算符合样式的列数
-i 忽略大小写
-l 只列出文件内容符合指定的样式的文件名称
-f 从文件中读取关键词
-n 显示匹配内容的所在文件中行数
-R 递归查找文件夹
```

grep 的规则表达式:

```
^  #锚定行的开始 如：'^grep'匹配所有以grep开头的行。 
$  #锚定行的结束 如：'grep$'匹配所有以grep结尾的行。 
.  #匹配一个非换行符的字符 如：'gr.p'匹配gr后接一个任意字符，然后是p。  
*  #匹配零个或多个先前字符 如：'*grep'匹配所有一个或多个空格后紧跟grep的行。
.*   #一起用代表任意字符。  
[]   #匹配一个指定范围内的字符，如'[Gg]rep'匹配Grep和grep。 
[^]  #匹配一个不在指定范围内的字符，如：'[^A-FH-Z]rep'匹配不包含A-R和T-Z的一个字母开头，紧跟rep的行。  
\(..\)  #标记匹配字符，如'\(love\)'，love被标记为1。   
\<      #锚定单词的开始，如:'\<grep'匹配包含以grep开头的单词的行。
\>      #锚定单词的结束，如'grep\>'匹配包含以grep结尾的单词的行。
x\{m\}  #重复字符x，m次，如：'0\{5\}'匹配包含5个o的行。 
x\{m,\}  #重复字符x,至少m次，如：'o\{5,\}'匹配至少有5个o的行。  
x\{m,n\}  #重复字符x，至少m次，不多于n次，如：'o\{5,10\}'匹配5--10个o的行。  
\w    #匹配文字和数字字符，也就是[A-Za-z0-9]，如：'G\w*p'匹配以G后跟零个或多个文字或数字字符，然后是p。  
\W    #\w的反置形式，匹配一个或多个非单词字符，如点号句号等。  
\b    #单词锁定符，如: '\bgrep\b'只匹配grep。
```

**实例**：

（1）查找指定进程

```shell
ps -ef | grep svn
```

（2）查找指定进程个数

```shell
ps -ef | grep svn -c
```

（3）从文件中读取关键词

```shell
cat test1.txt | grep -f key.log
```

（4）从文件夹中递归查找以 grep 开头的行，并只列出文件

```shell
grep -lR '^grep' /tmp
```

（5）查找非 x 开关的行内容

```shell
grep '^[^x]' test.txt
```

（6）显示包含 ed 或者 at 字符的内容行

```shell
grep -E 'ed|at' test.txt
```

### wc 命令

wc(word count)功能为统计指定的文件中字节数、字数、行数，并将统计结果输出

命令格式：

```
wc [option] file..
```

**命令参数**：

```
-c 统计字节数
-l 统计行数
-m 统计字符数
-w 统计词数，一个字被定义为由空白、跳格或换行字符分隔的字符串
```

**实例**：

（1）查找文件的 行数 单词数 字节数 文件名

```shell
wc text.txt
```

结果：

```shell
7     8     70     test.txt
```

（2）统计输出结果的行数

```shell
cat test.txt | wc -l
```

## 磁盘管理命令

### cd 命令

cd(changeDirectory) 命令语法：

```
cd [目录名]
```

说明：切换当前目录至 dirName。

**实例**：

（1）进入要目录

```shell
cd /
```

（2）进入 “home” 目录

```shell
cd ~
```

（3）进入上一次工作路径

```shell
cd -
```

（4）把上个命令的参数作为 cd 参数使用。

```shell
cd !$
```

### df 命令

显示磁盘空间使用情况。获取硬盘被占用了多少空间，目前还剩下多少空间等信息，如果没有文件名被指定，则所有当前被挂载的文件系统的可用空间将被显示。默认情况下，磁盘空间将以 1KB 为单位进行显示，除非环境变量 POSIXLY_CORRECT 被指定，那样将以 512 字节为单位进行显示：

```
-a 全部文件系统列表
-h 以方便阅读的方式显示信息
-i 显示inode信息
-k 区块为1024字节
-l 只显示本地磁盘
-T 列出文件系统类型
```

**实例**：

（1）显示磁盘使用情况

```shell
df -l
```

（2）以易读方式列出所有文件系统及其类型

```shell
df -haT
```

### du 命令

du 命令也是查看使用空间的，但是与 df 命令不同的是 Linux du 命令是对文件和目录磁盘使用的空间的查看：

命令格式：

```
du [选项] [文件]
1
```

**常用参数**：

```
-a 显示目录中所有文件大小
-k 以KB为单位显示文件大小
-m 以MB为单位显示文件大小
-g 以GB为单位显示文件大小
-h 以易读方式显示文件大小
-s 仅显示总计
-c或--total  除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和
```

**实例**：

（1）以易读方式显示文件夹内及子文件夹大小

```shell
du -h scf/
```

（2）以易读方式显示文件夹内所有文件大小

```shell
du -ah scf/
```

（3）显示几个文件或目录各自占用磁盘空间的大小，还统计它们的总和

```shell
du -hc test/ scf/
```

（4）输出当前目录下各个子目录所使用的空间

```shell
du -hc --max-depth=1 scf/
```

### ls命令

就是 list 的缩写，通过 ls 命令不仅可以查看 linux 文件夹包含的文件，而且可以查看文件权限(包括目录、文件夹、文件权限)查看目录信息等等。

**常用参数搭配**：

```
ls -a 列出目录所有文件，包含以.开始的隐藏文件
ls -A 列出除.及..的其它文件
ls -r 反序排列
ls -t 以文件修改时间排序
ls -S 以文件大小排序
ls -h 以易读大小显示
ls -l 除了文件名之外，还将文件的权限、所有者、文件大小等信息详细列出来
```

**实例**：

(1) 按易读方式按时间反序排序，并显示文件详细信息

```shell
ls -lhrt
```

(2) 按大小反序显示文件详细信息

```shell
ls -lrS
```

(3)列出当前目录中所有以"t"开头的目录的详细内容

```shell
ls -l t*
```

(4) 列出文件绝对路径（不包含隐藏文件）

```shell
ls | sed "s:^:`pwd`/:"
```

(5) 列出文件绝对路径（包含隐藏文件）

```shell
find $pwd -maxdepth 1 | xargs ls -ld
```

### mkdir 命令

mkdir 命令用于创建文件夹。

可用选项：

- **-m**: 对新建目录设置存取权限，也可以用 chmod 命令设置;
- **-p**: 可以是一个路径名称。此时若路径中的某些目录尚不存在,加上此选项后，系统将自动建立好那些尚不在的目录，即一次可以建立多个目录。

**实例**：

（1）当前工作目录下创建名为 t 的文件夹

```shell
mkdir t
```

（2）在 tmp 目录下创建路径为 test/t1/t 的目录，若不存在，则创建：

```shell
mkdir -p /tmp/test/t1/t
```

### pwd 命令

pwd 命令用于查看当前工作目录路径。

**实例**：

（1）查看当前路径

```shell
pwd
```

（2）查看软链接的实际路径

```shell
pwd -P
```

### rmdir 命令

从一个目录中删除一个或多个子目录项，删除某目录时也必须具有对其父目录的写权限。

**注意**：不能删除非空目录

**实例**：

（1）当 parent 子目录被删除后使它也成为空目录的话，则顺便一并删除：

```shell
rmdir -p parent/child/child11
```

## 网络通讯命令

### ifconfig 命令

- ifconfig 用于查看和配置 Linux 系统的网络接口。
- 查看所有网络接口及其状态：`ifconfig -a` 。
- 使用 up 和 down 命令启动或停止某个接口：`ifconfig eth0 up` 和 `ifconfig eth0 down` 。

### iptables 命令

iptables ，是一个配置 Linux 内核防火墙的命令行工具。功能非常强大，对于我们开发来说，主要掌握如何开放端口即可。例如：

- 把来源 IP 为 192.168.1.101 访问本机 80 端口的包直接拒绝：`iptables -I INPUT -s 192.168.1.101 -p tcp --dport 80 -j REJECT` 。

- 开启 80 端口，因为 web 对外都是这个端口

	```shell
	iptables -A INPUT -p tcp --dport 80 -j ACCEP
	```

- 另外，要注意使用 `iptables save` 命令，进行保存。否则，服务器重启后，配置的规则将丢失。

### netstat 命令

Linux netstat 命令用于显示网络状态。

利用 netstat 指令可让你得知整个 Linux 系统的网络情况。

语法

```
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
```

**参数说明**：

- -a 或–all 显示所有连线中的 Socket。
- -A<网络类型>或–<网络类型> 列出该网络类型连线中的相关地址。
- -c 或–continuous 持续列出网络状态。
- -C 或–cache 显示路由器配置的快取信息。
- -e 或–extend 显示网络其他相关信息。
- -F 或–fib 显示 FIB。
- -g 或–groups 显示多重广播功能群组组员名单。
- -h 或–help 在线帮助。
- -i 或–interfaces 显示网络界面信息表单。
- -l 或–listening 显示监控中的服务器的 Socket。
- -M 或–masquerade 显示伪装的网络连线。
- -n 或–numeric 直接使用 IP 地址，而不通过域名服务器。
- -N 或–netlink 或–symbolic 显示网络硬件外围设备的符号连接名称。
- -o 或–timers 显示计时器。
- -p 或–programs 显示正在使用 Socket 的程序识别码和程序名称。
- -r 或–route 显示 Routing Table。
- -s 或–statistice 显示网络工作信息统计表。
- -t 或–tcp 显示 TCP 传输协议的连线状况。
- -u 或–udp 显示 UDP 传输协议的连线状况。
- -v 或–verbose 显示指令执行过程。
- -V 或–version 显示版本信息。
- -w 或–raw 显示 RAW 传输协议的连线状况。
- -x 或–unix 此参数的效果和指定"-A unix"参数相同。
- –ip 或–inet 此参数的效果和指定"-A inet"参数相同。

**实例**

**如何查看系统都开启了哪些端口？**

```shell
[root@centos6 ~ 13:20 #55]# netstat -lnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      1035/sshd
tcp        0      0 :::22                       :::*                        LISTEN      1035/sshd
udp        0      0 0.0.0.0:68                  0.0.0.0:*                               931/dhclient
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node PID/Program name    Path
unix  2      [ ACC ]     STREAM     LISTENING     6825   1/init              @/com/ubuntu/upstart
unix  2      [ ACC ]     STREAM     LISTENING     8429   1003/dbus-daemon    /var/run/dbus/system_bus_socket
```

**如何查看网络连接状况？**

```shell
[root@centos6 ~ 13:22 #58]# netstat -an
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN
tcp        0      0 192.168.147.130:22          192.168.147.1:23893         ESTABLISHED
tcp        0      0 :::22                       :::*                        LISTEN
udp        0      0 0.0.0.0:68                  0.0.0.0:*
```

**如何统计系统当前进程连接数？**

- 输入命令 `netstat -an | grep ESTABLISHED | wc -l` 。
- 输出结果 `177` 。一共有 177 连接数。

**用 netstat 命令配合其他命令，按照源 IP 统计所有到 80 端口的 ESTABLISHED 状态链接的个数？**

> 严格来说，这个题目考验的是对 awk 的使用。

首先，使用 `netstat -an|grep ESTABLISHED` 命令。结果如下：

```shell
tcp        0      0 120.27.146.122:80       113.65.18.33:62721      ESTABLISHED
tcp        0      0 120.27.146.122:80       27.43.83.115:47148      ESTABLISHED
tcp        0      0 120.27.146.122:58838    106.39.162.96:443       ESTABLISHED
tcp        0      0 120.27.146.122:52304    203.208.40.121:443      ESTABLISHED
tcp        0      0 120.27.146.122:33194    203.208.40.122:443      ESTABLISHED
tcp        0      0 120.27.146.122:53758    101.37.183.144:443      ESTABLISHED
tcp        0      0 120.27.146.122:27017    23.105.193.30:50556     ESTABLISHED
```

### ping 命令

Linux ping 命令用于检测主机。

执行 ping 指令会使用 ICMP 传输协议，发出要求回应的信息，若远端主机的网络功能没有问题，就会回应该信息，因而得知该主机运作正常。

指定接收包的次数

```shell
ping -c 2 www.baidu.com
```

### telnet 命令

Linux telnet 命令用于远端登入。

执行 telnet 指令开启终端机阶段作业，并登入远端主机。

语法

```
telnet [-8acdEfFKLrx][-b<主机别名>][-e<脱离字符>][-k<域名>][-l<用户名称>][-n<记录文件>][-S<服务类型>][-X<认证形态>][主机名称或IP地址<通信端口>]
```

**参数说明**：

- -8 允许使用 8 位字符资料，包括输入与输出。
- -a 尝试自动登入远端系统。
- -b<主机别名> 使用别名指定远端主机名称。
- -c 不读取用户专属目录里的.telnetrc 文件。
- -d 启动排错模式。
- -e<脱离字符> 设置脱离字符。
- -E 滤除脱离字符。
- -f 此参数的效果和指定"-F"参数相同。
- -F 使用 Kerberos V5 认证时，加上此参数可把本地主机的认证数据上传到远端主机。
- -k<域名> 使用 Kerberos 认证时，加上此参数让远端主机采用指定的领域名，而非该主机的域名。
- -K 不自动登入远端主机。
- -l<用户名称> 指定要登入远端主机的用户名称。
- -L 允许输出 8 位字符资料。
- -n<记录文件> 指定文件记录相关信息。
- -r 使用类似 rlogin 指令的用户界面。
- -S<服务类型> 设置 telnet 连线所需的 IP TOS 信息。
- -x 假设主机有支持数据加密的功能，就使用它。
- -X<认证形态> 关闭指定的认证形态。

**实例**

登录远程主机

```shell
# 登录IP为 192.168.0.5 的远程主机
telnet 192.168.0.5 
```

## 系统管理命令

### date 命令

显示或设定系统的日期与时间。

命令参数：

```
-d<字符串> 　显示字符串所指的日期与时间。字符串前后必须加上双引号。
-s<字符串> 　根据字符串来设置日期与时间。字符串前后必须加上双引号。
-u 　显示GMT。
%H 小时(00-23)
%I 小时(00-12)
%M 分钟(以00-59来表示)
%s 总秒数。起算时间为1970-01-01 00:00:00 UTC。
%S 秒(以本地的惯用法来表示)
%a 星期的缩写。
%A 星期的完整名称。
%d 日期(以01-31来表示)。
%D 日期(含年月日)。
%m 月份(以01-12来表示)。
%y 年份(以00-99来表示)。
%Y 年份(以四位数来表示)。
```

**实例**：

（1）显示下一天

```shell
date +%Y%m%d --date="+1 day"  //显示下一天的日期
```

（2）-d 参数使用

```shell
date -d "nov 22"  今年的 11 月 22 日是星期三
date -d '2 weeks' 2周后的日期
date -d 'next monday' (下周一的日期)
date -d next-day +%Y%m%d（明天的日期）或者：date -d tomorrow +%Y%m%d
date -d last-day +%Y%m%d(昨天的日期) 或者：date -d yesterday +%Y%m%d
date -d last-month +%Y%m(上个月是几月)
date -d next-month +%Y%m(下个月是几月)
```

### free 命令

显示系统内存使用情况，包括物理内存、交互区内存(swap)和内核缓冲区内存。

**命令参数**：

```
-b 以Byte显示内存使用情况
-k 以kb为单位显示内存使用情况
-m 以mb为单位显示内存使用情况
-g 以gb为单位显示内存使用情况
-s<间隔秒数> 持续显示内存
-t 显示内存使用总合
```

**实例**：

（1）显示内存使用情况

```shell
free
free -k
free -m
```

（2）以总和的形式显示内存的使用信息

```shell
free -t
```

（3）周期性查询内存使用情况

```shell
free -s 10
```

### kill 命令

发送指定的信号到相应进程。不指定型号将发送 SIGTERM（15）终止指定进程。如果任无法终止该程序可用"-KILL" 参数，其发送的信号为 SIGKILL(9) ，将强制结束进程，使用 ps 命令或者 jobs 命令可以查看进程号。root 用户将影响用户的进程，非 root 用户只能影响自己的进程。

**常用参数**：

```
-l  信号，若果不加信号的编号参数，则使用“-l”参数会列出全部的信号名称
-a  当处理当前进程时，不限制命令名和进程号的对应关系
-p  指定kill 命令只打印相关进程的进程号，而不发送任何信号
-s  指定发送信号
-u  指定用户
```

**实例**：

（1）先使用 ps 查找进程 pro1，然后用 kill 杀掉

```shell
kill -9 $(ps -ef | grep pro1)
```

### ps 命令

ps(process status)，用来查看当前运行的进程状态，一次性查看，如果需要动态连续结果使用 top

linux 上进程有 5 种状态:

1. 运行(正在运行或在运行队列中等待)
2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号)
3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)
4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用 wait4()系统调用后释放)
5. 停止(进程收到 SIGSTOP, SIGSTP, SIGTIN, SIGTOU 信号后停止运行运行)

ps 工具标识进程的 5 种状态码:

```
D 不可中断 uninterruptible sleep (usually IO)
R 运行 runnable (on run queue)
S 中断 sleeping
T 停止 traced or stopped
Z 僵死 a defunct (”zombie”) process
```

**命令参数**：

```
-A 显示所有进程
a 显示所有进程
-a 显示同一终端下所有进程
c 显示进程真实名称
e 显示环境变量
f 显示进程间的关系
r 显示当前终端运行的进程
-aux 显示所有包含其它使用的进程
```

**实例**：

（1）显示当前所有进程环境变量及进程间关系

```shell
ps -ef
```

（2）显示当前所有进程

```shell
ps -A
```

（3）与 grep 联用查找某进程

```shell
ps -aux | grep apache
```

（4）找出与 cron 与 syslog 这两个服务有关的 PID 号码

```shell
ps aux | grep '(cron|syslog)'
```

### rpm 命令

Linux rpm 命令用于管理套件。

rpm(redhat package manager) 原本是 Red Hat Linux 发行版专门用来管理 Linux 各项套件的程序，由于它遵循 GPL 规则且功能强大方便，因而广受欢迎。逐渐受到其他发行版的采用。RPM 套件管理方式的出现，让 Linux 易于安装，升级，间接提升了 Linux 的适用度。

```shell
# 查看系统自带jdk
rpm -qa | grep jdk
# 删除系统自带jdk
rpm -e --nodeps 查看jdk显示的数据
# 安装jdk
rpm -ivh jdk-7u80-linux-x64.rpm
```

### top 命令

显示当前系统正在执行的进程的相关信息，包括进程 ID、内存占用率、CPU 占用率等

**常用参数**：

```
-c 显示完整的进程命令
-s 保密模式
-p <进程号> 指定进程显示
-n <次数>循环显示次数
```

**实例**：

```shell
top - 14:06:23 up 70 days, 16:44,  2 users,  load average: 1.25, 1.32, 1.35
Tasks: 206 total,   1 running, 205 sleeping,   0 stopped,   0 zombie
Cpu(s):  5.9%us,  3.4%sy,  0.0%ni, 90.4%id,  0.0%wa,  0.0%hi,  0.2%si,  0.0%st
Mem:  32949016k total, 14411180k used, 18537836k free,   169884k buffers
Swap: 32764556k total,        0k used, 32764556k free,  3612636k cached
PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND  
28894 root      22   0 1501m 405m  10m S 52.2  1.3   2534:16 java  
```

前五行是当前系统情况整体的统计信息区。

**第一行，任务队列信息，同 uptime 命令的执行结果，具体参数说明情况如下**：

14:06:23 — 当前系统时间

up 70 days, 16:44 — 系统已经运行了 70 天 16 小时 44 分钟（在这期间系统没有重启过的吆！）

2 users — 当前有 2 个用户登录系统

load average: 1.15, 1.42, 1.44 — load average 后面的三个数分别是 1 分钟、5 分钟、15 分钟的负载情况。

load average 数据是每隔 5 秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑 CPU 的数量，结果高于 5 的时候就表明系统在超负荷运转了。

**第二行，Tasks — 任务（进程），具体信息说明如下**：

系统现在共有 206 个进程，其中处于运行中的有 1 个，205 个在休眠（sleep），stoped 状态的有 0 个，zombie 状态（僵尸）的有 0 个。

**第三行，cpu状态信息，具体属性说明如下**：

```
5.9%us — 用户空间占用CPU的百分比。
3.4% sy — 内核空间占用CPU的百分比。
0.0% ni — 改变过优先级的进程占用CPU的百分比
90.4% id — 空闲CPU百分比
0.0% wa — IO等待占用CPU的百分比
0.0% hi — 硬中断（Hardware IRQ）占用CPU的百分比
0.2% si — 软中断（Software Interrupts）占用CPU的百分比
```

**备注**：在这里 CPU 的使用比率和 windows 概念不同，需要理解 linux 系统用户空间和内核空间的相关知识！

第四行，内存状态，具体信息如下：

```
32949016k total — 物理内存总量（32GB）
14411180k used — 使用中的内存总量（14GB）
18537836k free — 空闲内存总量（18GB）
169884k buffers — 缓存的内存量 （169M）
```

**第五行，swap交换分区信息，具体信息说明如下**：

```
32764556k total — 交换区总量（32GB）
0k used — 使用的交换区总量（0K）
32764556k free — 空闲交换区总量（32GB）
3612636k cached — 缓冲的交换区总量（3.6GB）
```

**第六行，空行。**

**第七行以下：各进程（任务）的状态监控，项目列信息说明如下**：

```
PID — 进程id
USER — 进程所有者
PR — 进程优先级
NI — nice值。负值表示高优先级，正值表示低优先级
VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
SHR — 共享内存大小，单位kb
S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
%CPU — 上次更新到现在的CPU时间占用百分比
%MEM — 进程使用的物理内存百分比
TIME+ — 进程使用的CPU时间总计，单位1/100秒
COMMAND — 进程名称（命令名/命令行）
```

**top 交互命令**

```
h 显示top交互命令帮助信息
c 切换显示命令名称和完整命令行
m 以内存使用率排序
P 根据CPU使用百分比大小进行排序
T 根据时间/累计时间进行排序
W 将当前设置写入~/.toprc文件中
o或者O 改变显示项目的顺序
```

### yum 命令

yum（ Yellow dog Updater, Modified）是一个在 Fedora 和 RedHat 以及 SUSE 中的 Shell 前端软件包管理器。

基於 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装。

yum 提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

- 1.列出所有可更新的软件清单命令：yum check-update
- 2.更新所有软件命令：yum update
- 3.仅安装指定的软件命令：yum install <package_name>
- 4.仅更新指定的软件命令：yum update <package_name>
- 5.列出所有可安裝的软件清单命令：yum list
- 6.删除软件包命令：yum remove <package_name>
- 7.查找软件包 命令：yum search
- 8.清除缓存命令:
	- yum clean packages: 清除缓存目录下的软件包
	- yum clean headers: 清除缓存目录下的 headers
	- yum clean oldheaders: 清除缓存目录下旧的 headers
	- yum clean, yum clean all (= yum clean packages; yum clean oldheaders) :清除缓存目录下的软件包及旧的 headers

**实例**

安装 pam-devel

```shell
[root@www ~]# yum install pam-devel
```

## 备份压缩命令

### bzip2 命令

- 创建 `*.bz2` 压缩文件：`bzip2 test.txt` 。
- 解压 `*.bz2` 文件：`bzip2 -d test.txt.bz2` 。

### gzip 命令

- 创建一个 `*.gz` 的压缩文件：`gzip test.txt` 。
- 解压 `*.gz` 文件：`gzip -d test.txt.gz` 。
- 显示压缩的比率：`gzip -l *.gz` 。

### tar 命令

用来压缩和解压文件。tar 本身不具有压缩功能，只具有打包功能，有关压缩及解压是调用其它的功能来完成。

弄清两个概念：打包和压缩。打包是指将一大堆文件或目录变成一个总的文件；压缩则是将一个大的文件通过一些压缩算法变成一个小文件

**常用参数**：

```
-c 建立新的压缩文件
-f 指定压缩文件
-r 添加文件到已经压缩文件包中
-u 添加改了和现有的文件到压缩包中
-x 从压缩包中抽取文件
-t 显示压缩文件中的内容
-z 支持gzip压缩
-j 支持bzip2压缩
-Z 支持compress解压文件
-v 显示操作过程
```

有关 gzip 及 bzip2 压缩:

```
gzip 实例：压缩 gzip fileName .tar.gz 和.tgz  解压：gunzip filename.gz 或 gzip -d filename.gz
          对应：tar zcvf filename.tar.gz     tar zxvf filename.tar.gz

bz2实例：压缩 bzip2 -z filename .tar.bz2 解压：bunzip filename.bz2或bzip -d filename.bz2
       对应：tar jcvf filename.tar.gz         解压：tar jxvf filename.tar.bz2
```

**实例**：

（1）将文件全部打包成 tar 包

```shell
tar -cvf log.tar 1.log,2.log 或tar -cvf log.*
```

（2）将 /etc 下的所有文件及目录打包到指定目录，并使用 gz 压缩

```shell
tar -zcvf /tmp/etc.tar.gz /etc
```

（3）查看刚打包的文件内容（一定加 z，因为是使用 gzip 压缩的）

```shell
tar -ztvf /tmp/etc.tar.gz
```

（4）要压缩打包 /home, /etc ，但不要 /home/dmtsai

```shell
tar --exclude /home/dmtsai -zcvf myfile.tar.gz /home/* /etc
```

### unzip 命令

- 解压 `*.zip` 文件：`unzip test.zip` 。
- 查看 `*.zip` 文件的内容：`unzip -l jasper.zip` 。
