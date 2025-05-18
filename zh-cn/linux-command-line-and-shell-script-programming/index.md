# Linux Command Line And Shell Script Programming


<!--more-->
# Linux命令行与shell脚本编程大全.第3版
## 第一部分 Linux 命令行 
### linux介绍
- Linux可划分为以下四部分：
>1. Linux内核
>2. GNU工具
>3. 图形化桌面环境
>4. 应用软件 

- 内核主要负责以下四种功能：
>1. 系统内存管理
>2. 软件程序管理
>3. 硬件设备管理
>4. 文件系统管理

- 内存管理：
内核可以使用**物理内存**，也可以创建和管理**虚拟内存**。
内核通过硬盘上的存储空间来实现虚拟内存，这块区域称为**交换空间**(swap space)。
内存存储单元按组划分成很多块，这些块称作**页面**(page)。内核将每个内存页面放在物理内存或交换空间。
内核会记录哪些内存页面正在使用中，并自动把一段时间未访问的内存页面复制到交换空间区域(称为**换出**，swapping out)——即使还有可用内存。
当程序要访问一个已被换出的内存页面时，内核必须从物理内存换出另外一个内存页面给它让出空间，然后从交换空间换入请求的内存页面。
只要Linux系统在运行，为运行中的程序换出内存页面的过程就不会停歇。
- 软件管理：
Linux操作系统将运行中的程序称为**进程**。[配置防火墙](https：//p3terx.com/archives/installing-and-configuring-ufw-in-debian.html "配置防火墙教程")
进程可以在前台运行，将输出显示在屏幕上，也可以在后台运行，隐藏到幕后。内核控制着Linux系统如何管理运行在系统上的所有进程。
**内核创建了第一个进程**(称为init进程)来启动系统上所有其他进程。当内核启动时，它会将init进程加载到虚拟内存中。内核在启动任何其他进程时，都会在虚拟内存中给新进程分配一块专有区域来存储该进程用到的数据和代码。
管理**自启动进程**(即启动或停止脚本)的表位于`/etc/init.d`中。(Ubuntu)。这些脚本通过/etc/rcX.d目录下的入口(entry)启动，
这里的X代表运行级(run level)。
Linux操作系统的init系统采用了运行级。运行级决定了init进程运行`/etc/inittab`文件或`/etc/rcX.d`目录中定义好的某些特定类型的进程。Linux操作系统有5个启动运行级。
运行级为1时，只启动基本的系统进程以及一个控制台终端进程。我们称之为单用户模式。单用户模式通常用来在系统有问题时进行紧急的文件系统维护。
标准的启动运行级是3。在这个运行级上，大多数应用软件，比如网络支持程序，都会启动。
另一个Linux中常见的运行级是5。在这个运行级上系统会启动图形化的X Window系统，允许用户通过图形化桌面窗口登录系统。
- 硬件设备管理：
内核的另一职责是管理硬件设备。任何Linux系统需要与之通信的设备，都需要在内核代码中加入其驱动程序代码。驱动程序代码相当于应用程序和硬件设备的中间人，允许内核与设备之间交换数据。
在Linux内核中有两种方法用于插入设备驱动代码：
>- 编译进内核的设备驱动代码
>- 可插入内核的设备驱动模块

开发人员提出了**内核模块**的概念。它允许将驱动代码插入到运行中的内核而无需重新编译内核。同时，当设备不再使用时也可将内核模块从内核中移走。这种方式极大地简化和扩展了硬件设备在Linux上的使用。
Linux系统将硬件设备当成特殊的文件，称为设备文件。
设备文件有3种分类：
>- 字符型设备文件
>- 块设备文件
>- 网络设备文件

**字符型设备文件**是指处理数据时每次只能处理一个字符的设备。如大多数的调制解调器和终端。
**块设备文件**是指处理数据时每次能处理大块数据的设备。如硬盘。
**网络设备文件**是指采用数据包发送和接收数据的设备。，包括各种网卡和一个特殊的回环设备。这个回环设备允许Linux系统使用常见的网络编程协议同自身通信。
Linux为系统上的每个设备都创建一种称为**节点**的特殊文件。与设备的所有通信都通过设备节点完成。每个节点都有唯一的数值对供Linux内核标识它。数值对包括一个主设备号和一个次设备号。类似的设备被划分到同样的主设备号下。次设备号用于标识主设备组下的某个特定设备。
- 文件系统管理
Linux内核支持通过不同类型的文件系统从硬盘中读写数据。除了自有的诸多文件系统(多达20多种)外，Linux还支持从其他操作系统(比如Microsoft Windows)采用的文件系统中读写数据。
Linux服务器所访问的所有硬盘都必须格式化成上述20种文件系统类型中的一种。
Linux内核采用虚拟文件系统(Virtual File System，VFS)作为和每个文件系统交互的接口。这为Linux内核同任何类型文件系统通信提供了一个标准接口。当每个文件系统都被挂载和使用时，VFS将信息都缓存在内存中。
-  GNU 工具
除了由内核控制硬件设备外，操作系统还需要工具来执行一些标准功能，比如控制文件和
程序。
通常将Linux内核和GNU工具的结合体称为Linux。
GNU coreutils软件包由三部分构成：
>- 用以处理文件的工具
>- 用以操作文本的工具
>- 用以管理进程的工具

GNU/Linux shell是一种特殊的交互式工具。
它为用户提供了启动程序、管理文件系统中的文件以及运行在Linux系统上的进程的途径。**shell的核心是命令行提示符**。命令行提示符是shell负责交互的部分。它允许你输入文本命令，然后解释命令，并在内核中执行。
shell包含了一组内部命令，用这些命令可以完成诸如复制文件、移动文件、重命名文件、显示和终止系统中正运行的程序等操作。shell也允许你在命令行提示符中输入程序的名称，它会将程序名传递给内核以启动它。
你也可以**将多个shell命令放入文件中作为程序执行**。这些文件被称作**shell脚本**。你在命令行上执行的任何命令都可放进一个shell脚本中作为一组命令执行。这为创建那种需要把几个命令放在一起来工作的工具提供了便利。 

**GNOME**(the GNU Network Object Model Environment，GNU网络对象模型环境)。
### shell介绍
在图形化桌面出现之前，与Unix系统进行交互的唯一方式就是借助由shell所提供的文本命令行界面(command line interface，CLI)。CLI只能接受文本输入，也只能显示出文本和基本的图形输出。
**GNOME 終端快捷键**：
**F11** 全屏
**CTRL+SHIFT+N** 启动一个新的shell会话
**CTRL+SHIFT+T** 在现在的窗口启动一个新的shell会话
**CTRL+SHIFT+W** 关闭当前的shell会话中的活动标签
**CTRL+SHIFT+Q** 关闭当前shell会话
**CTRL+SHIFT+V** 粘贴到shell会话
**CTRL+SHIFT+C** 复制选择内容到shell会话剪切板
**CTRL+SHIFT+F** 搜索文本 
**CTRL+SHIFT+H** 从当前位置向前搜索文本 
**CTRL+SHIFT+G** 从当前位置向后搜索文本 
### 基本bash shell命令
本节介绍处理文件和目录相关的命令。
`/etc/passwd`文件包含了所有系统用户**账户列表**以及每个用户的**基本配置信息**。
默认bash shell提示符是美元符号($)。
`man`命令用来访问存储在Linux系统上的手册页面。
要查找与终端相关的命令，可以输入`man -k terminal`。
除了对节按照惯例进行命名，手册页还有对应的内容区域。每个内容区域都分配了一个数字，
从1开始，一直到9。
|区域号|所涵盖的内容
|:---:|---|
|1|可执行程序或shell命令|
|2|系统调用|
|3|库调用|
|4|特殊文件|
|5|文件格式与约定|
|6|游戏|
|7|概述、约定及杂项|
|8|超级用户和系统管理员命令|
|9|内核例程|

`info 命令`也可查看命令说明。
`-help`或`--help`也可查看命令说明。
常见Linux目录名称：

|目录|用途|
|:---:|---|
|/|虚拟目录的根目录，通常不会在这里存储文件|
|/bin|二进制目录，存放许多用户的GNU工具|
|/boot|启动目录，存放启动文件|
|/dev|设备目录，Linux在这里创建设备节点|
|/etc|系统配置文件目录|
|/home|主目录，Linux在这里创建用户目录|
|/lib|库目录，存放系统和应用程序的库文件|
|/media|媒体目录，可移动媒体设备的常用挂载点|
|/mnt|挂载目录，另一个可移动媒体设备的常用挂载点|
|/opt|可选目录，常用于存放第三方软件包和数据文件|
|/proc|进程目录，存放现有硬件及当前进程的相关信息|
|/root|root用户主目录|
|/sbin|系统二进制目录，存放许多管理员级GNU工具|
|/run|运行目录，存放系统运行时的运行时数据|
|/srv|服务目录，存放本地服务的相关文件|
|/sys|系统目录，存放系统硬件信息的相关文件|
|/tmp|临时目录，可以在该目录中创建和删除临时工作文件|
|/usr|用户二进制目录，大量用户级的GNU工具和数据文件都存储在这里|
|/var|可变目录，用于存放经常变化的文件，比如日志文件|
 
`cd destination`切换目录命令
`cd`命令可接受单个参数destination，用以指定想切换到的目录名。如果没有为`cd`命令指
定目标路径，它将切换到**用户主目录**。
destination参数可以用两种方式表示：一种是使用绝对文件路径(以"/"起始的路径)，另一种是使用相对文件
路径。
`pwd`命令可以显示出shell会话的当前目录，这个目录被称为当前工作目录。
有两个特殊字符可用于相对文件路径中：
单点符(.)，表示当前目录;
双点符(..)，表示当前目录的父目录。
`ls`命令最基本的形式会显示当前目录下的文件和目录。
`ls -F`-F参数在目录名后加了正斜线(/)，在可执行文件后加(*)号。
11
`ls -R` 参数是`ls`命令可用的另一个参数，叫作递归选项。
`ls -r`逆序。
创建文件`touch filename`。
`touch filename`命令还可用来**改变文件的修改时间**。这个操作并不需要改变文件的内容。
改变访问时间，可用`-a`参数。
复制文件 `cp source destination`
当source和destination参数都是文件名时， cp命令将源文件复制成一个新文件，并且以destination命名。
**链接文件**
**链接**是目录中指向文件真实位置的占位符。
在Linux中有两种不同类型的文件链接：
符号链接
硬链接
**符号链接**就是一个实实在在的文件，它指向存放在虚拟目录结构中某个地方的另一个文件。这两个通过符号链接在一起的文件，彼此的内容并不相同。
创建符号链接命令`ln -s soure destination`。
**硬链接**会创建独立的虚拟文件，其中包含了原始文件的信息及位置。但是它们从根本上而言是**同一个文件**要创建硬链接，原始文件也必须事先存在`ln source destination`。
**只能对处于同一存储媒体的文件创建硬链接。要想在不同存储媒体的文件之间创建链接，只能使用符号链接。**
 重命名文件
`mv oldname newname`修改文件名称。
`mv source destination`也可以使用`mv`来移动文件的位置。
`mv source/oldfilename destination/newfilename`也可以使用mv命令移动文件位置并修改文件名称。
 删除文件
`rm -f` 强制删除文件。
`rm -rf`可用来强制删除目录。
创建目录
`mkdir`创建目录。
同时创建多个目录和子目录，需要加入`-p`参数`mkidr -p d1/d2/d3`。
删除目录的基本命令是`rmdir`。默认情况下，只删除空目录。
查看文件类型
`flie filename`查看文件属性。
查看整个文件
`cat filename`查看文件内容。
`cat -n `给所有行加入行号。
`cat -b `只给有文本的行加上行号，空白行不加行号。
`cat -T `制表位显示为`^T`。
more命令
`more`命令会显示文本文件的内容，但会在显示每页数据之后停下来。`more`命令是分页工具。
less命令
`less filename`。less命令为more命令的升级版。
查看部分文件
`tail`命令会显示文件最后几行的内容(文件的“尾部”)。
`tail filename`默认显示文件尾部10行内容。
`tail -n 2 filename`或者`tail -2 filename`指定显示末尾2行内容。
`tail -f `允许在其他进程使用该文件时查看文件的内容。
tail 命令会保持活动状态，并不断显示添加到文件中的内容。
head命令
类似tail用法。不支持`-f`参数，因为通常文件开头不会改变。
### 更多bash shell命令
本章介绍系统管理命令。
- 监测进程(某个特定时间点)
查看进程命令`ps`。
默认情况下， `ps`命令只会显示运行在当前控制台下的属于当前用户的进程。
Linux系统中使用的GNU ps命令支持3种不同类型的命令行参数：
>- Unix风格的参数，前面加单破折线;
>- BSD风格的参数，前面不加破折线;
>- GNU风格的长参数，前面加双破折线。

Unix风格参数：
`ps -aux/aux`查看所有进程。
`ps -ef`以完整格式(-f)显示所有(-e)运行的进程
|名称|描述|
|:---:|---|
|UID|启动这些进程的用户|
|PID|进程的进程ID|
|PPID|父进程的进程号(如果该进程是由另一个进程启动的)|
|C|进程生命周期中的CPU利用率|
|STIME|进程启动时的系统时间|
|TTY|进程启动时的终端设备|
|TIME|运行进程需要的累计CPU时间|
|CMD|启动的程序名称|

`ps -l`后面会增加如下几项：
|名称|描述|
|:---:|---|
|F|内核分配给进程的系统标记|
|S|进程的状态(O代表正在运行；S代表在休眠；R代表可运行，正等待运行；Z代表僵化，进程已结束但父进程已不存在；T代表停止)|
|PRI|进程的优先级(越大的数字代表越低的优先级)|
|NI|谦让度值用来参与决定优先级|
|ADDR|进程的内存地址|
|SZ|假如进程被换出，所需交换空间的大致大小|
|WCHAN|进程休眠的内核函数的地址|

BSD风格参数：
`ps l`命令大部分输出与Unix风格类似。不同部分如下：

|名称|解释|
|:---:|---|
|VSZ|进程在内存中的大小，以千字节(KB)为单位|
|RSS|进程在未换出时占用的物理内存|
|STAT|代表当前进程状态的双字符状态码|

第一个字符采用了和Unix风格S列相同的值，表明进程是在休眠、运行还是等待。第二个参数进一步说明进程的状态，如`Ssl+`
>- <：该进程运行在高优先级上。
>- N：该进程运行在低优先级上。
>- L：该进程有页面锁定在内存中。
>- s：该进程是控制进程。
>- l：该进程是多线程的。
>- +：该进程运行在前台。

GNU长参数：
如`--help`。
`ps --forest`**会显示进程的层级信息，并以ASCLL字符绘画成可爱的图表。**
**这种格式让跟踪子进程和父进程变得十分容易。**
- 实时监测进程
`top`命令。
`top -u username`监测特定用户进程。
`top -p 11345`监测特定进程号进程。
`top`命令输出，第一行显示了当前时间、系统的运行时间、登录的用户数以及系统的平均负载。
`top`命令的输出中将进程叫作任务(task)：有多少进程处在
运行(running)、休眠(sleeping)、停止(stopped)或是僵化状态(zombie，僵化状态是指进程完成了，但父进程没有响应) 。
最后显示运行进程列表参数解释如下：

|名称|描述|
|:---:|---|
|PID|进程的ID|
|USER|进程属主名字|
|PR|进程的优先级|
|NI|进程的谦让度值|
|VIRT|进程占用的虚拟内存总量|
|RES|进程占用的物理内存总量|
|SHR|进程和其他进程共享的内存总量|
|S|进程状态(D代表可中断的休眠状态，R代表在运行状态，S代表休眠状态，T代表跟踪状态或停止状态，Z代表僵化状态)|
|%CPU|进程使用的CPU时间比例|
|%MEM|进程使用的内存占可用内存的比例|
|TME+|自进程启动到目前为止的CPU时间总量|
|COMMAND|进程所对应的命令行名称，也就是启动的程序名|

- 结束进程
在Linux中，进程之间通过信号来通信。
进程如何处理信号是由开发人员通过编程来决定的。
`kill`命令
`kill`命令可通过进程ID(PID)给进程发信号。默认情况下， kill命令会向命令行中列出的全部PID发送一个TERM信号。`kill`命令**只能用进程的PID**而不能用命令名。
linux进程信号：

|信号|名称|描述|
|:---:|:---:|---|
|1|HUP|挂起|
|2|INT|中断|
|3|QUIT|结束运行|
|9|KILL|无条件终止|
|11|SEGV|段错误|
|15|TERM|尽可能终止|
|17|STOP|无条件停止运行，但不终止|
|18|TSTP|停止或暂停，但继续在后台运行|
|19|CONT|在STOP和TSTP之后恢复执行|

如果要强制终止，`-s`参数支持指定其他信号(用信号名或信号值)，如：
`kill -s HUP 3490`
`killall`命令
`killall`命令非常强大，它**支持通过进程名**而不是PID来结束进程。`killall`命令也**支持通配符**，这在系统因负载过大而变得很慢时很有用。
- 监测磁盘空间
`mount`命令。
>`mount`命令提供如下四部分信息：
>- 媒体的设备文件名
>- 媒体挂载到虚拟目录的挂载点
>- 文件系统类型
>- 已挂载媒体的访问状态

要手动在虚拟目录中挂载设备，需要以root用户身份登录，或是以root用户身份运行`sudo`命令。下面是手动挂载媒体设备的基本命令：
`mount -t type device directory`
type参数指定了磁盘被格式化的文件系统类型。和Windows PC共用这些存储设备。
>通常得使用下列文件系统类型：
>- vfat：Windows长文件系统。
>- ntfs：Windows NT、XP、Vista以及Windows 7中广泛使用的高级文件系统。
>- iso9660：标准CD-ROM文件系统。

-o参数允许在挂载文件系统时添加一些以逗号分隔的额外选项。
>以下为常用的选项：
>- ro：以只读形式挂载。
>- rw：以读写形式挂载。
>- user：允许普通用户挂载文件系统。
>- check=none：挂载文件系统时不进行完整性校验。
>- loop：挂载一个文件。

`umount`命令，卸载。
`umount [directory | device ]`
`df`命令，查看挂载磁盘使用情况。
`df -h`用户易读形式显示。
`df`命令输出显示的时Linux系统认为的当前值。有可能系统上有运行的进程已经创建或删除了某个文件，但尚未释放文件。
这个值是不会算进闲置空间的。
`du`命令
`du`命令可以显示某个特定目录(默认情况下是当前目录)的磁盘使用情况(查看内容包含所有文件及子目录)。主要用来查看该目录下是否有超大文件。

>常用可选参数：
>- -c:显示所有已列出文件总的大小。
>- -h:按用户易读的格式输出大小，即用K替代千字节，用M替代兆字节，用G替代吉字节。
-> -s:显示每个输出参数的总计。

- 处理大数据文件命令
**排序数据**
`sort`命令会按照**默认语言**的排序规则对文本文件数据进行排序。
默认情况下，`sort`命令会把**数字当作字符来执行标准的字符排序**。解决该问题，则需执行`sort -n filename`，按值排序。
`sort -M`用三字符月份名按月份排序。
`sort -r`逆序。

|单破折号|双破折号|描述|
|:---:|---|---|
|-b|--ignore-leading-blanks| 排序时忽略起始的空白|
|-C|--check=quiet| 不排序，如果数据无序也不要报告|
|-c|--check|不排序，但检查输入数据是不是已排序;未排序的话，报告|
|-d|--dictionary-order|仅考虑空白和字母，不考虑特殊字符|
|-f|--ignore-case|默认情况下，会将大写字母排在前面;这个参数会忽略大小写|
|-g|--general-number-sort|按通用数值来排序(跟-n不同，把值当浮点数来排序，支持科学计数法表示的值)|
|-i|--ignore-nonprinting|在排序时忽略不可打印字符|
|-k|--key=POS1[，POS2]|排序从POS1位置开始;如果指定了POS2的话，到POS2位置结束|
|-M|--month-sort|用三字符月份名按月份排序|
|-m|--merge|将两个已排序数据文件合并|
|-n|--numeric-sort|按字符串数值来排序(并不转换为浮点数)|
|-o|--output=file|将排序结果写出到指定的文件中|
|-R|--random-sort|按随机生成的散列表的键值排序|
||--random-source=FILE|指定-R 参数用到的随机字节的源文件|
|-r|--reverse|反序排序(升序变成降序)|
|-S|--buffer-size=SIZE|指定使用的内存大小|
|-s|--stable|禁用最后重排序比较|
|-T|--temporary-directory=DIR|指定一个位置来存储临时工作文件|
|-t|--field-separator=SEP|指定一个用来区分键位置的字符|
|-u|--unique|和-c 参数一起使用时，检查严格排序;不和-c参数一起用时，仅输出第一例相似的两行|
|-z|--zero-terminated|用NULL字符作为行尾，而不是用换行符|

`-k`和`-t`参数在对按字段分隔的数据进行排序时非常有用，例如`/etc/passwd`文件。可以用`-t`参数来指定字段分隔符，然后用`-k`参数来指定排序的字段。
`$ sort -t ':' -k 3 -n /etc/passwd`
`$ du -sh * | sort -nr `
**搜索数据**
`grep`命令
命令格式`grep [options] pattern [file]`
`grep -v t file1`反向搜索包含 t 字符的文本。
`grep -n `显示所在行行号。
`grep -c`只显示统计出现的总行数。
`grep -e t -e f file1`指定多个匹配模式。
精确搜索则匹配模式使用正则表达式。
`fgrep`则是另外一个版本，支持将匹配模式指定为用换行符分隔的一列固定长度的字符串。这样就可以把这列字符串放到一个文件中，然后在`fgrep`命令中用其在一个大型文件中搜索字符串了。
压缩数据
|工具|文件扩展名|描述|
|:---:|---|---|
|bzip2|.bz2|采用Burrows-Wheeler块排序文本压缩算法和霍夫曼编码|
|compress|.Z|最初的Unix文件压缩工具,已经快没人用了|
gzip|.gz|GNU压缩工具,用Lempel-Ziv编码|
zip|.zip|Windows上PKZIP工具的Unix实现|

gzip软件包
> 这个软件包含有下面的工具：
>- gzip:用来压缩文件。
>- gzcat:用来查看压缩过的文本文件的内容。
>- gunzip:用来解压文件。
这些工具基本上跟bzip2工具的用法一样。

`gzip my*`用通配符一次性批量压缩文件。
**归档数据**
Unix 和 Linux 标准归档工具 `tar`命令。
命令格式：
`tar function [options] object1 object2 ...`
|功能|长名称|描述|
|:-----:|:----:|-----|
|-A|--concatenate|将一个已有tar归档文件追加到另一个已有tar归档文件|
|-c|--create|创建一个新的tar归档文件|
|-d|--diff|检查归档文件和文件系统的不同之处|
||--delete|从已有tar归档文件中删除|
|-r|--append|追加文件到已有tar归档文件末尾|
|-t|--list|列出已有tar归档文件的内容|
|-u|--update|将比tar归档文件中已有的同名文件新的文件追加到该tar归档文件中|
|-x|--extract|从已有tar归档文件中提取文件|

|选项|描述|
|---|---|
|-C dir|切换到指定目录|
|-f file|输出结果到文件或设备file|
|-j|将输出重定向给bzip2命令来压缩内容|
|-p|保留所有文件权限|
|-v|在处理文件时显示文件|
|-z|将输出重定向给gzip命令来压缩内容|

`tar -cvf test.tar test/ test2/`创建一个归档文件
`tar -tf test.tar`列出 tar 文件test.tar的内容(但并不提取文件)
`tar -xvf test.tar`在当前目录下提取内容
`*.tgz`文件，是`gzip`压缩过的 tar 文件可以用命令`tar -zxvf filename.tgz`来解压。
### 理解shell
CLI，Command-line interface，命令行界面
`ps --forest`查看shell进程层级关系
`exit`退出子shell，在父shell中还可以登出当前的虚拟控制台终端或终端仿真器软件。
运行shell脚本也能够创建出子shell。
**进程列表**
`$ (pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls)`"()"和命令构成一个进程列表，**生成了一个子shell来执行对应的命令**。
进程列表是一种**命令分组**(command grouping)。另一种命令分组是将命令放入花括号中,并在命令列表尾部加上分号";"。语法为{ command; }。使用花括号进行命令分组并**不会像进程列表那样创建出子shell**。
`echo $BASH_SUBSHELL`返回0则表示无子shell，返回1或者更大的数则表明存在子shell。
**子shell并非真正的多进程处理**，因为终端控制着子shell的I/O。
**子shell用法**
- 后台模式
要想将命令置入后台模式，可以在命令末尾加上字符 & 。
`jobs`命令可以显示出当前运行在后台模式中的所有用户的进程(作业) 。
`jobs -l`查看后台进程，能显示命令PID。
- 协程
协程可以同时做两件事。它在后台生成一个子shell，并在这个子shell中执行命令。
```bash
 coproc sleep 2           ##命令输入
  coproc COPROC sleep 2&  ##结果显示
```
 `COPROC`是`coproc`命令给进程起的名字。你可以使用命令的扩展语法自己设置这个名字。
```bash
corproc my_jobs { sleep 2; }    ##命令输入
  coproc My_Job { sleep 10; }&  ##结果显示
```
只有在拥有**多个协程**的时候才需要对协程进行**命名**，因为你得和它们进行通信。
将协程与进程列表结合起来产生嵌套的子shell。只需要输入进程列表,然后把命令coproc放在前面就行了。
```bash
  coproc ( sleep 2; sleep 10; )
```
**生成子shell的成本不低，而且速度还慢。创建嵌套子shell更是火上浇油!**
**内建命令**
- 外部命令
外部命令，有时候也被称为**文件系统命令**，是存在于bash shell之外的程序。
外部命令程序通常位于 `/bin`、`/usr/bin`、`/sbin`或`/usr/sbin`中。
```bash
which ps
/bin/ps
type -a ps
ps is /bin/ps #每次执行ps都会生成一个新的子进程查看当前系统进程，然后自动退出。
ls -l /bin/ps
-rwxr-xr-x 1 root root 137688 10月 31 2023 /bin/ps
```
当外部命令执行时，会创建出一个子进程。这种操作被称为衍生(forking)。
- 内建命令
**内建命令和外部命令的区别在于前者不需要使用子进程来执行**。它们已经和shell编译成了一体，作为shell工具的组成部分存在。不需要借助外部程序文件来运行。

`cd`和`exit`命令都内建于bash shell。可以利用`type`命令来了解某个命令是否是内建的。
```bash
$ type cd
cd is a shell builtin
$ type ps
ps is hashed (/bin/ps)
```
有些命令有多种实现。例如`echo`和`pwd`既有内建命令也有外部命令。
```bash
$ type echo     #仅显示内建命令
echo is a shell builtin
$ type -a echo  #显示内建命令和外部命令
echo is a shell builtin
echo is /bin/echo
$ which echo    #仅显示外部命令
/bin/echo
```
`history`查看历史记录也是shell内建命令。
不带选项的`history`通常会显示最近的1000命令。通过修改环境变量`HESTSIZE`的值来修改保存的历史命令数量。
`!!`唤出刚刚使用的命令。
命令历史记录被保存在隐藏文件`.bash_history`中，它位于用户的主目录中。这里要注意的是，bash命令的历史记录是先存放在内存中，当shell退出时才被写入到历史文件中。
`history -a`可将历史命令强制写入`.bash_history`文件中而不需等到退出shell。
`!20`可将历史记录编号为20的命令取出。
`alias`命名别名也是shell内建命令。
命令别名允许你为常用的命令(及其参数)创建另一个名称，从而将输入量减少到最低。
`alias -p`要查看当前可用的别名。
### 使用linux环境变量
**环境变量**
bash shell用一个叫作环境变量(environment variable)的特性来存储有关shell会话和工作环境的信息(这也是它们被称作环境变量的原因)。这项特性允许你在内存中存储数据，以便程序或shell中运行的脚本能够轻松访问到它们。
环境变量分类：

>- 全局变量
>- 局部变量

**全局环境变量**
全局环境变量对于shell会话和所有生成的子shell都是可见的。局部变量则只对创建它们的shell可见。
**系统环境变量基本上都是使用全大写字母，以区别于普通用户的环境变量。**
`env/printenv`查看全局变量命令。
`printenv HOME`显示个别环境变量的值。也可以`echo $HOME`。
**局部环境变量**
局部环境变量只能在定义它们的进程中可见。
**Linux系统也默认定义了标准的局部环境变量**。
在Linux系统并没有一个只显示局部环境变量的命令。`set`命令会显示为某个特定进程设置的所有环境变量，包括局部变量、全局变量。
`set`命令会显示出全局变量、局部变量以及用户定义变量。它还会按照字母顺序对结果进行**排序**。
`env`和 `printenv`命令则不会排序，也不会输出局部变量和用户定义变量。
**设置用户定义变量**
```bash
$ echo $myself  #创建用户变量

$ myself=hello  #给变量赋值，所赋值含空格等特殊符号意义符号时需用引号。
$ echo $myself  #显示变量值
hello
```
**变量名、等号和值之间没有空格**。
在子shell中无法使用用户定义变量。退出子进程后用户变量仍存在。
**设置全局变量**
创建全局环境变量的方法是先创建一个局部环境变量，然后再把它导出到全局环境中。
```bash
my_value="hello world"
export my_value
```
**子**shell甚至**无法**使用`export`命令**改变父**shell中全局环境变量的值。
**删除环境变量**
```bash
echo $my_value  #创建变量
unset my_value  #删除变量，不加$
```
和修改变量一样，在子shell中删除全局变量后，你无法将效果反映到父shell中。
所以要删除变量，应该在创建变量的shell进程中执行。
**默认的shell环境变量**
bash shell支持的Bourne变量。

|变量|描述|
|----|----|
|CDPATH|冒号分隔的目录列表,作为cd 命令的搜索路径|
|HOME| 当前用户的主目录|
|IFS| shell用来将文本字符串分割成字段的一系列字符|
|MAIL|当前用户收件箱的文件名(bash shell会检查这个文件,看看有没有新邮件)|
|MAILPATH|冒号分隔的当前用户收件箱的文件名列表(bash shell会检查列表中的每个文件,看看有没有新邮件)|
|OPTARG|getopts命令处理的最后一个选项参数值|
|OPTIND|getopts命令处理的最后一个选项参数的索引号|
|PATH|shell查找命令的目录列表,由冒号分隔|
|PS1| shell命令行界面的主提示符|
|PS2| shell命令行界面的次提示符|

除了默认的Bourne的环境变量,bash shell还提供一些自有的变量。

|变量|描述|
|----|----|
|BASH|当前shell实例的全路径名|
|BASH_ALIASES|含有当前已设置别名的关联数组|
|BASH_ARGC|含有传入子函数或shell脚本的参数总数的数组变量|
|BASH_ARCV|含有传入子函数或shell脚本的参数的数组变量|
|BASH_CMDS|关联数组,包含shell执行过的命令的所在位置|
|BASH_COMMAND|shell正在执行的命令或马上就执行的命令|
|BASH_ENV|设置了的话,每个bash脚本会在运行前先尝试运行该变量定义的启动文件|
|BASH_EXECUTION_STRING|使用bash -c 选项传递过来的命令|
|BASH_LINENO|含有当前执行的shell函数的源代码行号的数组变量|
|BASH_REMATCH|只读数组,在使用正则表达式的比较运算符=~进行肯定匹配(positive match)时,包含了匹配到的模式和子模式|
|BASH_SOURCE|含有当前正在执行的shell函数所在源文件名的数组变量|
|BASH_SUBSHELL|当前子shell环境的嵌套级别(初始值是0)|
|BASH_VERSINFO|含有当前运行的bash shell的主版本号和次版本号的数组变量|
|BASH_VERSION|当前运行的bash shell的版本号|
|BASH_XTRACEFD|若设置成了有效的文件描述符(0、 1、2),则 'set -x'调试选项生成的跟踪输出可被重定向。通常用来将跟踪输出到一个文件中|
|BASHOPTS|当前启用的bash shell选项的列表|
|BASHPID|当前bash进程的PID|
|COLUMNS|当前bash shell实例所用终端的宽度|
|COMP_CWORD|COMP_WORDS 变量的索引值,后者含有当前光标的位置|
|COMP_LINE|当前命令行|
|COMP_POINT|当前光标位置相对于当前命令起始的索引|
|COMP_KEY|用来调用shell函数补全功能的最后一个键|
|COMP_TYPE|一个整数值,表示所尝试的补全类型,用以完成shell函数补全|
|COMP_WORDBREAKS|Readline库中用于单词补全的词分隔字符|
|COMP_WORDS|含有当前命令行所有单词的数组变量|
|COMPREPLY|含有由shell函数生成的可能填充代码的数组变量|
|COPROC|占用未命名的协进程的I/O文件描述符的数组变量|
|DIRSTACK| 含有目录栈当前内容的数组变量|
|EMACS|设置为't' 时,表明emacs shell缓冲区正在工作,而行编辑功能被禁止|
|ENV|如果设置了该环境变量,在bash shell脚本运行之前会先执行已定义的启动文件(仅用于当bash shell以POSIX模式被调用时)|
|EUID|当前用户的有效用户ID(数字形式)|
|FCEDIT|供fc命令使用的默认编辑器|
|FIGNORE|在进行文件名补全时可以忽略后缀名列表,由冒号分隔|
|FUNCNAME|当前执行的shell函数的名称|
|FUNCNEST|当设置成非零值时,表示所允许的最大函数嵌套级数(一旦超出,当前命令即被终止)|
|GLOBIGNORE|冒号分隔的模式列表,定义了在进行文件名扩展时可以忽略的一组文件名|
|GROUPS|含有当前用户属组列表的数组变量|
|histchars|控制历史记录扩展,最多可有3个字符|
|HISTCMD|当前命令在历史记录中的编号|
|HISTCONTROL|控制哪些命令留在历史记录列表中|
|HISTFILE|保存shell历史记录列表的文件名(默认是.bash_history)|
|HISTFILESIZE|最多在历史文件中存多少行|
|HISTTIMEFORMAT|如果设置了且非空,就用作格式化字符串,以显示bash历史中每条命令的时间戳|
|HISTIGNORE|由冒号分隔的模式列表,用来决定历史文件中哪些命令会被忽略|
|HISTSIZE|最多在历史文件中存多少条命令|
|HOSTFILE|shell在补全主机名时读取的文件名称|
|HOSTNAME|当前主机的名称|
|HOSTTYPE|当前运行bash shell的机器|
|IGNOREEOF|shell在退出前必须收到连续的 EOF 字符的数量(如果这个值不存在,默认是1)|
|INPUTRC|Readline初始化文件名(默认是.inputrc)|
|LANG|shell的语言环境类别|
|LC_ALL|定义了一个语言环境类别,能够覆盖LANG变量|
|LC_COLLATE|设置对字符串排序时用的排序规则|
|LC_CTYPE|决定如何解释出现在文件名扩展和模式匹配中的字符|
|LC_MESSAGES|在解释前面带有$的双引号字符串时,该环境变量决定了所采用的语言环境设置|
|LC_NUMERIC|决定着格式化数字时采用的语言环境设置|
|LINENO|当前执行的脚本的行号|
|LINES|定义了终端上可见的行数|
|MACHTYPE|用“CPU-公司-系统”(CPU-company-system)格式定义的系统类型|
|MAPFILE|一个数组变量,当mapfile命令未指定数组变量作为参数时,它存储了mapfile所读入的文本|
|MAILCHECK|shell查看新邮件的频率(以秒为单位,默认值是60)|
|OLDPWD|shell之前的工作目录|
|OPTERR|设置为1时,bash shell会显示getopts命令产生的错误|
|OSTYPE|定义了shell所在的操作系统|
|PIPESTATUS|含有前台进程的退出状态列表的数组变量|
|POSIXLY_CORRECT|设置了的话,bash会以POSIX模式启动|
|PPID|bash shell父进程的PID|
|PROMPT_COMMAND|设置了的话,在命令行主提示符显示之前会执行这条命令|
|PROMPT_DIRTRIM|用来定义当启用了 \w或\W提示符字符串转义时显示的尾部目录名的数量。被删除的目录名会用一组英文句点替换|
|PS3|select命令的提示符|
|PS4|如果使用了bash的-x 选项,在命令行之前显示的提示信息|
|PWD|当前工作目录|
|RANDOM|返回一个0~32767的随机数(对其的赋值可作为随机数生成器的种子)|
|READLINE_LINE|当使用bind -x 命令时,存储Readline缓冲区的内容|
|READLINE_POINT|当使用bind -x 命令时,表示Readline缓冲区内容插入点的当前位置|
|REPLY|read命令的默认变量|
|SECONDS|自从shell启动到现在的秒数(对其赋值将会重置计数器)|
|SHELL|bash shell的全路径名|
|SHELLOPTS|已启用bash shell选项列表,列表项之间以冒号分隔|
|SHLVL|shell的层级;每次启动一个新bash shell,该值增加1|
|TIMEFORMAT|指定了shell的时间显示格式|
|TMOUT|select和read命令在没输入的情况下等待多久(以秒为单位)。默认值为0,表示无限长|
|TMPDIR|目录名,保存bash shell创建的临时文件|
|UID|当前用户的真实用户ID(数字形式)|

**设置 PATH 环境变量**
PATH环境变量定义了用于进行命令和程序查找的目录。
```bash
$ echo $PATH
/home/mrhou/bin:/home/mrhou/.local/bin:/usr/local/sbin:/usr/local/bin:
/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```
PATH中的目录使用冒号分隔。
如果命令或者程序的位置没有包括在PATH变量中，那么如果不使用绝对路径的话，shell是没法找到的。
```bash
$ PATH=$PATH:/home/christine/Scripts #引用原目录，以:隔开后追加新目录
```
程序员通常的办法是将单点符(即当前目录)也加入PATH环境变量。
```bash
  $PATH=PATH:.
```
对PATH变量的修改只能持续到退出或重启系统。
**定位系统环境变量**
在你登入Linux系统启动一个bash shell时，默认情况下bash会在几个文件中查找命令。这些文件叫作启动文件或环境文件。bash检查的启动文件取决于你启动bash shell的方式。
  启动bash shell有3种方式:
>- 登录时作为默认登录shell 
>- 作为非登录shell的交互式shell
>- 作为运行脚本的非交互shell

登录 shell
当你登录Linux系统时，bash shell会作为登录shell启动。
登录shell会从5个不同的启动文件里读取命令:
>- `/etc/profile`
>- `$HOME/.bash_profile`
>- `$HOME/.bashrc`
>- `$HOME/.bash_login`
>- `$HOME/.profile`

`/etc/profile`文件是系统上默认的bash shell的主启动文件。系统上的每个用户登录时都会执行这个启动文件。
另外4个启动文件是针对用户的,可根据个人需求定制。
1. `/etc/profile`文件
`/etc/profile`文件是bash shell默认的的主启动文件。只要你登录了Linux系统,bash就会执行`/etc/profile`启动文件中的命令。
不同的Linux发行版在这个文件里放了不同的命令。
大部分应用都会创建两个启动文件:一个供bash shell使用(使用.sh扩展名)，一个供c shell使用(使用.csh扩展名)。
2. `$HOME`目录下的启动文件
剩下的启动文件都起着同一个作用:提供一个用户专属的启动文件来定义该用户所用到的环境变量。
大多数Linux发行版只用这四个启动文件中的一到两个:
>- $HOME/.bash_profile
>- $HOME/.bashrc
>- $HOME/.bash_login
>- $HOME/.profile

这四个文件都以点号开头，这说明它们是隐藏文件(不会在通常的ls命令输出列表中出现)。
它们位于用户的HOME目录下，所以每个用户都可以编辑这些文件并添加自己的环境变量，这些环境变量会在每次启动bash shell会话时生效。
shell会按照按照下列顺序，运行第一个被找到的文件，余下的则被忽略:
`$HOME/.bash_profile`
`$HOME/.bash_login`
`$HOME/.profile`
注意，这个列表中并没有`$HOME/.bashrc`文件。这是因为该文件通常通过其他文件运行的。
交互式 shell 进程
如果你的bash shell不是登录系统时启动的(比如是在命令行提示符下敲入bash时启动)，那么你启动的shell叫作交互式shell。
交互式shell启动的，它就不会访问`/etc/profile`文件，只会检查用户HOME目录中的`.bashrc`文件。
非交互式 shell
系统执行shell脚本时用的就是这种shell。它没有命令行提示符。
如果父shell是登录shell，在`/etc/profile`、`/etc/profile.d/*.sh`和`$HOME/.bashrc`文件中设置并导出了变量，用于执行脚本的子shell就能够继承这些变量。
**由父shell设置但并未导出的变量都是局部变量。子shell无法继承局部变量。**
对于那些不启动子shell的脚本，变量已经存在于当前shell中了。所以就算没有设置BASH_ENV，也可以使用当前shell的局部变量和全局变量。
环境变量持久化
最好是在`/etc/profile.d`目录中创建一个以`.sh`结尾的文件。把所有新的或修改过的全局环境变量设置放在这个文件中。
在大多数发行版中，存储个人用户永久性bash shell变量的地方是`$HOME/.bashrc`文件。这一点适用于所有类型的shell进程。但如果设置了 BASH_ENV 变量，那么记住，除非它指向的是`$HOME/.bashrc`,否则你应该将非交互式shell的用户变量放在别的地方。
**数组变量**
**环境变量**有一个很酷的特性就是，它们**可作为数组使用**。

给某个环境变量设置多个值，可以把值放在括号里，值与值之间用空格分隔。

```bash
mytest=(one two three four five)
```
**环境变量数组的索引值都是从零开始**。
```bash
echo $mytest      #只会显示第一个元素值
echo ${mytest[3]} #显示第四个元素值
```
显示整个数组变量，可用星号作为通配符放在索引值的位置。
```bash
echo ${mytest[*]}
mytest[2]=seven #修改某个位置的值
```
`unset`删除值
```bash
 unset mytest[2] #删除索引为2的值
echo ${mytest[*]}
one two four five
echo ${mytest[2]} #此处索引值被删除了，但索引看起来存在。

echo ${mytest[3]}
four
unset mytest
echo ${mytest[*]}

```
有时**数组变量**会让事情很麻烦，所以在shell脚本**编程时并不常用**。
### 理解Linux文件权限
Linux安全系统的**核心**是**用户账户**。每个能进入Linux系统的用户都会被分配**唯一**的用户账户。用户对系统中各种对象的访问权限取决于他们登录系统时用的账户。
用户权限是通过创建用户时分配的用户ID(User ID，通常缩写为UID)来跟踪的。UID是数值，每个用户都有唯一的UID，但在登录系统时用的不是UID，而是登录名。
登录名是用户用来登录系统的最长八字符的字符串(字符可以是数字或字母)，同时会关联一个对应的密码。
- `/etc/passwd` 文件
Linux系统使用一个专门的文件来将用户的登录名匹配到对应的UID值。这个文件就是`/etc/passwd`文件，它包含了一些与用户有关的信息。
root用户账户是Linux系统的管理员，固定分配给它的UID是0。
Linux系统会为各种各样的功能创建不同的用户账户，而这些账户并不是真的用户。这些账户叫作**系统账户**，是系统上运行的各种服务进程访问资源用的特殊账户。所有运行在后台的服务都需要用一个系统用户账户登录到Linux系统上。
Linux为系统账户预留了500以下的UID值。
`/etc/passwd`文件中还有很多用户登录名和UID之外的信息。
` /etc/passwd`文件的字段包含了如下信息:
>- 登录用户名
>- 用户密码 (用x表示)
>- 用户账户的UID(数字形式)
>- 用户账户的组ID(GID)(数字形式)
>- 用户账户的文本描述(称为备注字段)
>- 用户HOME目录的位置
>- 用户的默认shell
 
 绝大多数Linux系统都将用户密码保存在另一个单独的文件中(叫作shadow文件，位置在`/etc/shadow`)。只有特定的程序(比如登录程序)才能访问这个文件。

`/etc/passwd`是一个标准的文本文件。你可以用任何文本编辑器在`/etc/password`文件里直接手动进行用户管理(比如添加、修改或删除用户账户)。

- `/etc/shadow`文件
`/etc/shadow`文件对Linux系统密码管理提供了更多的控制。只有root用户才能访问`/etc/shadow`文件，这让它比起`/etc/passwd`安全许多。
```bash
rich:$1$.FfcK0ns$f1UgiyHQ25wrB/hykCn020:11627:0:99999:7:::
```
在`/etc/shadow`文件的每条记录中都有9个字段:
>- 与`/etc/passwd`文件中的登录名字段对应的登录名
>- 加密后的密码
>- 自上次修改密码后过去的天数密码(自1970年1月1日开始计算)
>- 多少天后才能更改密码
>- 多少天后必须更改密码
>- 密码过期前提前多少天提醒用户更改密码
>- 密码过期后多少天禁用用户账户
>- 用户账户被禁用的日期(用自1970年1月1日到当天的天数表示)
>- 预留字段给将来使用
-  添加新用户
用来向Linux系统添加新用户的主要工具是`useradd`。
可以使用加入了 -D选项的`useradd`命令查看所用Linux系统中的这些默认值。
```bash
useradd -D
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/sh
SKEL=/etc/skel
CREATE_MAIL_SPOOL=no
```
列出的默认值如下:
>- 新用户会被添加到GID为 100的公共组
>- 新用户的HOME目录将会位于/home/loginname
>- 新用户账户密码在过期后不会被禁用
>- 新用户账户未被设置过期日期
>- 新用户账户将bash shell作为默认shell
>- 系统会将/etc/skel目录下的内容复制到用户的HOME目录下
>- 系统为该用户账户在mail目录下创建一个用于接收邮件的文件。

 `useradd`命令不会创建HOME目录，但是`-m`命令行选项会使其创建HOME目录。
 useradd命令行参数，参考`useradd --help`查看。
- 删除用户
`userdel username`系统中删除用户，默认情况下，`userdel`命令会只删除`/etc/passwd`文件中的用户信息，而不会删除系统中属于该账户的任何文件。
果加上`-r`参数，`userdel`会删除用户的HOME目录以及邮件目录。
然而，系统上仍可能存有已删除用户的其他文件。这在有些环境中会造成问题。
- 修改用户
用户账户修改工具:

|命令|描述|
|----|----|
|usermod|修改用户账户的字段,还可以指定主要组以及附加组的所属关系|
|passwd|修改已有用户的密码|
|chpasswd|从文件中读取登录名密码对,并更新密码|
|chage|修改密码的过期日期|
|chfn|修改用户账户的备注信息|
|chsh|修改用户账户的默认登录shell|

`usermod`命令
```bash
usermod -l  #修改用户账户的登录名
usermod -L  #锁定账户，是用户无法登录
usermod -p  #修改账户的密码
usermod -U  #解除锁定,使用户能够登录
#....,其他详见--help
```
`passwd`和`chpasswd`命令
```bash
passwd username       #只能修改自己
passwd -e username    #强制用户下次登录时修改密码
sudo passwd username  #可修改其他用户密码
```
`chpasswd`命令能从标准输入自动读取登录名和密码对(由冒号分割)列表，给密码加密，然后为用户账户设置。
也可以用重定向命令来将含有userid:passwd对的文件重定向给该命令。
```bash
$ chpasswd < user.txt
```
`chsh`、`chfn`和`chage`命令
`chsh`、`chfn`和`chage`工具专门用来修改特定的账户信息。`chsh`命令用来快速修改默认的用户登录shell。使用时必须用shell的全路径名作为参数，不能只用shell名。
```bash
 $ chsh -s /bin/csh test
```
`chfn`命令提供了在`/etc/passwd`文件的备注字段中存储信息的标准方法。
`chage`命令用来帮助管理用户账户的有效期。
|参数|描述|
|----|----|
|-d|设置上次修改密码到现在的天数|
|-E|设置密码过期的日期|
|-I|设置密码过期到锁定账户的天数|
|-m|设置修改密码之间最少要多少天|
|-W|设置密码过期前多久开始出现提醒信息|

**使用 Linux 组**
`/etc/group` 文件
```bash
$cat /etc/group #查看组
root:x:0:root
#...
```
`/etc/group`文件有4个字段:
>- 组名
>- 组密码
>- GID
>- 属于该组的用户列表

- 创建新组
```bash
$ groupadd groupname
```
在创建新组时，默认没有用户被分配到该组。`groupadd`命令没有提供将用户添加到组中的选项，但可以用`usermod`命令来弥补这一点。
```bash
$ usermod -G groupname username 
```
- 修改组
`groupmod`命令可以修改已有组的GID(加-g选项)或组名(加-n选项)。
```bash
$ groupmod -n newgroupname oldgroupname
```
**理解文件权限**

使用文件权限符
```bash
-rw-rw-r-- 1 rich rich 50 2010-09-13 07:49 file1.gz
```
这个字段的第一个字符代表了对象的类型:
>- -代表文件
>- d代表目录
>- l代表链接
>- c代表字符型设备
>- b代表块设备
>- n代表网络设备
之后有3组三字符的编码。每一组定义了3种访问权限:
>- r代表对象是可读的
>- w代表对象是可写的
>- x代表对象是可执行的
若没有某种权限,在该权限位会出现单破折线。这3组权限分别对应对象的3个安全级别:
>- 对象的属主
>- 对象的属组
>- 系统其他用户

默认文件权限
命令`umask`可以查看和设置默认文件属性
```bash
$ umask       #显示默认权限
0002  
$ umask 0001  #设置默认权限
```
第一位代表了一项特别的安全特性，叫作[粘着位](#粘着位)(sticky bit)。
后面的3位表示文件或目录对应的`umask`八进制值。
**八进制模式的安全性设置**先获取这3个rwx权限的值，然后将其转换成3位二进制值，用一个八进制值来表示。
Linux文件权限码:
|权限|二进制值|八进制值|描述|
|:----:|:----:|:----:|:----|
|---|000|0|没有任何权限| 
|--x|001|1|只有执行权限| 
|-w-|010|2|只有写入权限|
|-wx|011|3|只有写入和执行权限| 
|r--|100|4|只有读取权限| 
|r-x|101|5|只有读取和执行权限|
|rw-|110|6|只有读取和写入权限|
|rwx|111|7|有全部权限|

`umask`值只是个掩码。它会屏蔽掉不想授予该安全级别的权限。
要把`umask`值从对象的全权限值中减掉。对文件来说，全权限的值是666(所有用户都有读和写的权限);而对目录来说，则是777(所有用户都有读、写、执行权限)。
对于文件，对应的`umask`值为(666-文件权限对应的3位8进制值)，对于目录，对应的`umask`值为(777-目录对应的3位8进制值)。
改变安全性设置
改变权限
`chmod`命令用来改变文件和目录的安全性设置，该命令的格式如下:
`chmod options mode file`
与通常用到的3组三字符权限字符不同，`chmod`命令采用了另一种方法。下面是在符号模式下指定权限的格式。
`[ugoa...][[+-=][rwxXstugo...]`
第一组字符定义了权限作用的对象:
>- u代表用户
>- g代表组
>- o代表其他
>- a代表上述所有

后面跟着的符号表示你是想在现有权限基础上增加权限(+)，还是在现有权限基础上移除权限(-)，或是将权限设置成后面的值(=)。

第三个符号代表作用到设置上的权限:
>- x:如果对象是目录或者它已有执行权限,赋予执行权限。
>- s:运行时重新设置UID或GID。
>- t:保留文件或目录。
>- u:将权限设置为跟属主一样。
>- g:将权限设置为跟属组一样。
>- o:将权限设置为跟其他用户一样。
```bash
chmod o+r newfile #为其他用户添加读的权限
```
options为`chmod`命令提供了另外一些功能。`-R`选项可以让权限的改变递归地作用到文件和子目录。
改变所属关系。`-h`选项可以改变该文件的所有符号链接文件的所属关系。
`chown`命令用来改变文件的属主。
```bash
chown options owner[.group] file
#可用登录名或UID来指定文件的新属主。
chown dan newfile
#chown命令也支持同时改变文件的属主和属组
chown dan.shared newfile
```
`chgrp`命令用来改变文件或目录的默认属组。
```bash
chgrp shared newfile
```
<a id="粘着位"></a>
共享文件
Linux系统上共享文件的方法是创建组。
Linux还为每个文件和目录存储了3个额外的信息位:
>- 设置用户ID(SUID):当文件被用户使用时，程序会以文件属主的权限运行。
>- 设置组ID(SGID):对文件来说，程序会以文件属组的权限运行;对目录来说，目录中创建的新文件会以目录的默认属组作为默认属组。
>- 粘着位:进程结束后文件还驻留(粘着)在内存中。

SGID可通过`chmod`命令设置。它会加到标准3位八进制值之前(组成4位八进制值)，或者在符号模式下用符号`s`。
八进制模式， chmod SUID、SGID和粘着位的八进制值:
|二进制值|八进制值|描述|
|:----:|:----:|:----:|
|000|0|所有位都清零|
|001|1|粘着位置位|
|010|2|SGID位置位|
|011|3|SGID位和粘着位都置位|
|100|4|SUID位置位|
|101|5|SUID位和粘着位都置位|
|110|6|SUID位和SGID位都置位|
|111|7|所有位都置位|

创建一个共享目录，使目录里的新文件都能沿用目录的属组，只需将该目录的SGID位置位。
```bash
 mkdir testdir
ls -l
drwxrwxr-x 2 rich rich
chgrp testgroup testdir
ls -l
drwxrwxr-x 2 rich testgroup
chmod g+s testdir
ls -l 
drwxrwsr-x 2 rich testgroup
umask 002
cd testdir
touch testfile
ls -l
-rw-rw-r-- 1 rich testgroup
```
### 管理文件系统
创建分区
```bash
fdisk /dev/sdb  #进入分区磁盘，交互提示分区设置
```
`fdisk`命令提示符下的可用命令:
|命令|描述|
|:---:|:---|
|a|设置活动分区标志|
|b|编辑BSD Unix系统用的磁盘标签|
|c|设置DOS兼容标志|
|d|删除分区|
|l|显示可用的分区类型|
|m|显示命令选项|
|n|添加一个新分区|
|o|创建DOS分区表|
|p|显示当前分区表|
|q|退出,不保存更改|
|s|为Sun Unix系统创建一个新磁盘标签|
|t|修改分区的系统ID|
|u|改变使用的存储单位|
|v|验证分区表|
|w|将分区表写入磁盘|
|x|高级功能|

常用命令：
```bash
#用p命令将一个存储设备的详细信息显示出来
Command (m for help): p 
#以使用n命令在该存储设备上创建新的分区
Command (m for help): n
Command action
e extended  #扩展分区
p primary partition (1-4) #主分区
p
Partition number (1-4): 1
First cylinder (1-652, default 1): 1
Last cylinder, +cylinders or +size{K,M,G} (1-652, default 652): +2G #分配大小
#用w命令将更改保存到存储设备上
Command (m for help): w
```
主分区可以被文件系统直接格式化，而扩展分区则只能容纳其他逻辑分区。
扩展分区出现的原因是每个存储设备上只能有4个分区。可以通过创建多个扩展分区,然后在扩展分区内创建逻辑分区进行扩展。
创建文件系统
创建文件系统的命令行程序列表如下:
|工具|用途|
|---|---|
|mkefs|创建一个ext文件系统|
|mke2fs|创建一个ext2文件系统|
|mkfs.ext3|创建一个ext3文件系统|
|mkfs.ext4|创建一个ext4文件系统|
|mkreiserfs|创建一个ReiserFS文件系统|
|jfs_mkfs|创建一个JFS文件系统|
|mkfs.xfs|创建一个XFS文件系统|
|mkfs.zfs|创建一个ZFS文件系统|
|mkfs.btrfs|创建一个Btrfs文件系统|
```bash
type mkefs #查看工具是否可用
```
所有的文件系统命令都允许通过不带选项的简单命令来创建一个默认的文件系统。
```bash
sudo mkfs.ext4 /dev/sdb1
```
创建过程中有一步是创建新的日志。
将创建好的文件系统挂载到虚拟目录下的某个挂载点，这样就可以将数据存储在新文件系统中了。
```bash
$ ls /mnt
$ sudo mkdir /mnt/my_partition
$ ls -al /mnt/my_partition/
$ ls -dF /mnt/my_partition
/mnt/my_partition/
$ sudo mount -t ext4 /dev/sdb1 #将文件系统挂载
 /mnt/my_partition
$ ls -al /mnt/my_partition/
```
文件系统的检查与修复
`fsck`命令能够检查和修复大部分类型的Linux文件系统。
```bash
#用法格式：
fsck options filesystem
``` 
 fsck的命令行选项
|选项|描述|
|---|---|
|-a|如果检测到错误,自动修复文件系统|
|-A|检查/etc/fstab文件中列出的所有文件系统|
|-C|给支持进度条功能的文件系统显示一个进度条(只有ext2和ext3)|
|-N|不进行检查,只显示哪些检查会执行|
|-r|出现错误时提示|
|-R|使用-A选项时跳过根文件系统|
|-s|检查多个文件系统时,依次进行检查|
|-t|指定要检查的文件系统类型|
|-T|启动时不显示头部信息|
|-V|在检查时产生详细输出|
|-y|检测到错误时自动修复文件系统|

***只能在未挂载的文件系统上运行`fsck`命令**
以上讲的是关于物理存储设备中的文件系统。
逻辑卷管理
Linux逻辑卷管理器(logical volume manager,LVM)软件包。
逻辑卷管理的核心在于如何处理安装在系统上的硬盘分区。在逻辑卷管理的世界里，硬盘称作物理卷(physical volume,PV)。每个物理卷都会映射到硬盘上特定的物理分区。
多个物理卷集中在一起可以形成一个卷组(volume group,VG)。
使用 Linux LVM
1. 定义物理卷
```bash
[...]
#。在创建了基本的Linux分区之后,你需要通过t命令改变分区类型。
Command (m for help): t
Selected partition 1
#分区类型8e表示这个分区将会被用作Linux LVM系统的一部分
Hex code (type L to list codes): 8e
Changed system type of partition 1 to 8e (Linux LVM)
Command (m for help): p
[...]
Command (m for help): w
```
用分区来创建实际的物理卷，通过`pvcreate`命令定义用于物理卷的物理分区。它只是简单地将分区标记成Linux LVM系统中的分区而已。
```bash
sudo prcreate /dev/sdb1
```
想查看创建进度，用`pvdisplay`命令来显示已创建的物理卷列表。
```bash
sudo pvdisplay /dev/sdb1
```
2. 创建卷组
从物理卷中创建一个或多个卷组。
使用`vgcreate`命令来创建卷组。`vgcreate`命令需要一些命令行参数来定义卷组名以及你用来创建卷组的物理卷名。
```bash
 sudo vgcreate Vol1 /dev/sdb1
```
用`vgdisplay`命令新创建的卷组的细节。
```bash
sudo vgdisplay Vol1
```
3. 创建逻辑卷
Linux系统使用逻辑卷来模拟物理分区，并在其中保存文件系统。Linux系统会像处理物理分区一样处理逻辑卷，允许你定义逻辑卷中的文件系统，然后将文件系统挂载到虚拟目录上。
要创建逻辑卷,使用lvcreate命令。
`lvcreate`的选项

|选项|长选项名|描述|
|---|---|---|
|-c|--chunksize|指定快照逻辑卷的单位大小|
|-C|--contiguous|设置或重置连续分配策略|
|-i|--stripes|指定条带数|
|-I|--stripesize|指定每个条带的大小|
|-l|--extents|指定分配给新逻辑卷的逻辑区段数,或者要用的逻辑区段的百分比|
|-L|--size|指定分配给新逻辑卷的硬盘大小|
||--minor|指定设备的次设备号|
|-m|--mirrors|创建逻辑卷镜像|
|-M|--persistent|让次设备号一直有效|
|-n|--name|指定新逻辑卷的名称|
|-p|--permission|为逻辑卷设置读/写权限|
|-r|--readahead|设置预读扇区数|
|-R|--regionsize|指定将镜像分成多大的区|
|-s|snapshot|创建快照逻辑卷|
|-Z|--zero|将新逻辑卷的前1 KB数据设置为零|

常用选项:
```bash
sudo lvcreate -l 100%FREE -n lvtest Vol1
Logical volume "lvtest" created
```
用`lvdisplay`命令查看创建的逻辑卷的详细情况。
```bash
sudo lvdisplay Vol1
```
4. 创建文件系统
```bash
 sudo mkfs.ext4 /dev/Vol1/lvtest
```
在创建了新的文件系统之后,可以用标准Linux mount命令将这个卷挂载到虚拟目录中，就跟它是物理分区一样。唯一的不同是你需要用特殊的路径来标识逻辑卷。
```bash
sudo mount /dev/Vol1/lvtest /mnt/my_partition
mount
cd /mnt/my_partition
ls -al
```
`mkfs.ext4`和`mount`命令中用到的路径都有点奇怪。路径中使用了卷组名和逻辑卷名，而不是物理分区路径。
5. 修改LVM
Linux LVM的好处在于能够动态修改文件系统。
Linux LVM包中的常见命令:
|命令|功能|
|---|---|
|vgchange|激活和禁用卷组|
|vgremove|删除卷组|
|vgextend|将物理卷加到卷组中|
|vgreduce|从卷组中删除物理卷|
|lvextend|增加逻辑卷的大小|
|lvreduce|减小逻辑卷的大小|

运行`lsblk`命令，查看磁盘的分区。
>lsblk 用于查看磁盘和分区结构，df 用于查看文件系统的空间使用情。

逻辑卷中的文件系统需要手动修整来处理大小上的改变。大多数文件系统都包含了能够重新格式化文件系统的命令行程序，比如用于ext2、ext3和ext4文件系统的`resize2fs`程序。
### 安装软件程序
Linux中广泛使用的两种主要的PMS基础工具是`dpkg`和`rpm`。
基于Debian的发行版(如Ubuntu和Linux Mint)使用的是`dpkg`命令。
基于Red Hat的发行版(如Fedora、 openSUSE及Mandriva)使用的是`rpm`命令。
 基于 Debian 的系统
`dpkg`工具常用选项:
列出特定软件包所安装的全部文件
```bash
dpkg -L package_name
 dpkg -L vim-common
```
查找某个特定文件属于哪个软件包
```bash
dpkg --search absolute_file_name
#在使用的时候必须用绝对文件路径
dpkg --search /usr/bin/xxd
```
```bash
#使用下面的结构来指定仓库源。
deb (or deb-src) address distribution_name package_type_list
cat /etc/apt/sources.list #查看软件源
deb http://mirrors.aliyun.com/ubuntu/ focal multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse #Added by software-properties
```
deb或deb-src的值表明了软件包的类型。 deb值说明这是一个已编译程序源，而deb-src值则说明这是一个源代码的源。
distribution_name条目是这个特定软件仓库的发行版版本的名称。
package_type_list条目可能并不止一个词，它还表明仓库里面有什么类型的包。你可以看到诸如main、restricted、universe和partner这样的值。
 基于 Red Hat 的系统
 `yum`:在Red Hat和Fedora中使用。
 `urpm`:在Mandriva中使用。
 `zypper`:在openSUSE中使用。
列出系统上已安装的包
```bash
yum list installed                      #通常会一闪而过，查看所有已安装的包
yum list installed > installed_software #将这个记录重定向至文件方便查看
yum list xterm                          #查看特定应用包是否安装
yum provides file_name                  #找出系统上的某个特定文件属于哪个软件包
```
用`yum`安装软件
```bash
yum install package_name
```
手动下载`rpm`安装文件并用`yum`安装,这叫作本地安装。基本的命令是:
```bash
yum localinstall package_name.rpm
```
用`yum`更新软件
```bash
yum list updates        #列出所有可用更新
yum update package_name #更新特定安装包
yum update              #更新所有安装包
```
用 yum 卸载软件
```bash
yum remove package_name   #只删除软件包而保留配置文件和数据文件
yum erase package_name    #删除软件和它所有的文件
```
处理损坏的包依赖关系
有时在安装多个软件包时，某个包的软件依赖关系可能会被另一个包的安装覆盖掉。这叫作损坏的包依赖关系(broken dependency) 。
```bash
yum clean all
yum updates
```
显示所有包的库依赖关系以及什么软件可以提供这些库依赖关系。
```bash
yum deplist package_name
```
如果这样仍未解决问题，还有最后一招:
```bash
yum update --skip-broken    #--skip-broken选项允许你忽略依赖关系损坏的那个包,继续去更新其他软件包
```
 yum 软件仓库
要想知道你现在正从哪些仓库中获取软件，输入如下命令:
```bash
yum repolist
```
如果仓库中没有需要的软件，你可以编辑一下配置文件。`yum`的仓库定义文件位于`/etc/yum.repos.d`。你需要添加正确的URL，并获得必要的加密密钥。
从源码安装

```bash
tar -zxvf sysstat-11.1.1.tar.gz   #解压文件放入sysstat-11.1.1的目录中
cd sysstat
./configure                       #检查你的Linux系统,确保它拥有合适的编译器能够编译源代码,另外还要具备正确的库依赖关系
make                              #make命令会编译源码,然后链接器会为这个包创建最终的可执行文件
sudo make install                 #想将它安装到Linux系统中常用的位置上
```
## shell 脚本编程基础
### 构建基本脚本
创建 shell 脚本文件
在创建shell脚本文件时，必须在文件的第一行指定要使用的shell。其格式为:
```bash
#!/bin/bash
```
shell并不会处理shell脚本中的注释行。然而，shell脚本文件的第一行是个例外，# 后面的惊叹号会告诉shell用哪个shell来运行脚本(是的，你可以使用bash shell，同时还可以使用另一个shell来运行你的脚本)。
 显示消息
```bash
echo string
```
`echo`命令可用单引号或双引号来划定文本字符串。如果在字符串中用到了它们，你需要在文本中使用其中一种引号，而用另外一种来将字符串划定起来。

```bash
echo "it's hello wrold" 
```
把文本字符串和命令输出显示在同一行中，在字符串的两侧使用引号，保证要显示的字符串尾部有一个空格。
```bash
echo -n "hello world: " #后引号前有一个空格
```
使用变量

环境变量

```bash
echo $HOME
echo "The cost of the item is \$15" #加斜杠转义为符号而不是变量
```
用户变量

用户变量可以是任何由字母、数字或下划线组成的文本字符串，长度不超过20个。
**用户变量区分大小写**。
使用**等号**将值赋给用户变量。在变量、等号和值之间***不能出现空格**。
```bash
#!/bin/bash
date=10
guest="Peter"
echo "$guest checked in $day days ago"
```
命令替换

有两种方法可以将命令输出赋给变量:
>- 反引号字符( ` )
>- $( )格式

```bash
test=`date`
test1=$(date)
#一段常见用法,在脚本中通过命令替换获得当前日期并用它来生成唯一文件名
today=$(date +%y%m%d) #%y年份后两位
ls /usr/bin -al > log.$today
```
命令替换会**创建一个子shell**来运行对应的命令。子shell (subshell)是由运行该脚本的shell所创建出来的一个独立的子shell(child shell)。正因如此，由该子shell所执行命令是无法使用脚本中所创建的变量的。
在命令行提示符下使用路径 ./ 运行命令的话，也会创建出子shell;要是运行命令的时候不加入路径，就不会创建子shell。如果你使用的是内建的shell命令，并不会涉及子shell。在命令行提示符下运行脚本时一定要留心!

 重定向输入和输出
 
 输出重定向

最基本的重定向将命令`>`的输出发送到一个文件中，如果输出文件已经存在了，重定向操作符会用新的文件数据覆盖已有文件。
```bash
command > outputfile
date > test
```
追加数据`>>`
```bash
date >> test
```
 输入重定向
 输入重定向符号是小于号(<)。
`wc`命令可以对对数据中的文本进行计数:
>- -l(--lines)文本的行数
>- -w(--words)文本的词数
>- -c(--bytes)文本的字节数
>- -m(--chars)字符统计

将文本文件重定向到`wc`命令:
```bash
wc < test   #默认输出3个值，行数，词数，字节数
```
另外一种输入重定向的方法，称为内联输入重定向(inline input redirection)。
内联输入重定向符号是远小于号(<<)，你必须指定一个文本标记来划分输入数据的开始和结尾。任何字符串都可作为文本标记，但在数据的开始和结尾文本标记必须一致。
```bash
command << marker
date
marker
wc << EOF
>string 1
>string 2
>string 3
>EOF
#次提示符(>)
```
管道
管道符号`|`。
```bash
command | command
```
Linux系统实际上会**同时运行**由管道(`|`)连接的命令，在系统**内部**将它们**连接**起来。数据传输不会用到任何中间文件或缓冲区。
```bash
cat /etc/passwd | grep "n" | sort | more
cat /etc/passwd | grep "n" | sort > sorted_file
```
执行数学运算
`expr` 命令
```bash
$ expr 1 + 5  #+和数字之间有空格
6
```
许多`expr`命令操作符在shell中另有含义(比如星号)，在它们传入expr命令之前,需要使用shell的转义字符(反斜线)将其标出来。
```bash
$ expr 5 \* 2
10
```
**`expr`缺点较多，shell编程时不推荐使用**。
使用方括号[ ]
```bash
var1=$[ 1 + 2 ]
var2=$[ $var1 * 5 ]
```
**bash shell**数学运算符`[ ]`只支持**整数运算**。
z shell(zsh)提供了完整的浮点数算术操作。
浮点解决方案
最常见的方案是用内建的bash计算器，叫作`bc`。

1. `bc`的基本用法

bash计算器能够识别:
>- 数字(整数和浮点数)
>- 变量(简单变量和数组)
>- 注释(以#或C语言中的/* */开始的行)
>- 表达式
>- 编程语句(例如if-then语句)
>- 函数

浮点运算是由内建变量scale控制的，必须将这个值设置为你希望在计算结果中保留的小数位数，否则无法得到期望的结果。
`scale`变量的默认值是0。`-q`命令行选项可以不显示bash计算器冗长的欢迎信息。

```bash
bc -q   #进入bash计算器
var1=10
var1*3
var2=var1/5
scale-4   #保留4位小数
print var2
quit    #退出bc计算器
```
变量一旦被定义，你就可以在整个bash计算器会话中使用该变量了。print语句允许你打印变量和数字。
2. 在脚本中使用`bc`
基本格式如下:
```bash
variable=$(echo "options; expression" | bc)
```
```bash
#! /bin/bash
var1=$(echo "scale=4; 3.44 / 5" | bc)
echo The answer is $var1
```
如果需要进行大量运算，在一个命令行中列出多个表达式就会有点麻烦。
`bc`命令能识别输入重定向，允许你将一个文件重定向到`bc`命令来处理。
最好的办法是使用内联输入重定向，它允许你直接在命令行中重定向数据。
```bash
variable=$(bc << EOF
>options
>statements
>expressions
>EOF
)
```
在bash计算器中创建的变量只在bash计算器中有效，不能在shell脚本中使用。

退出脚本

shell中运行的每个命令都使用退出状态码(exit status)告诉shell它已经运行完毕。退出状态码是一个0~255的整数值，在命令结束运行时由命令传给shell。可以捕获这个值并在脚本中使用。
查看退出状态码
Linux提供了一个专门的变量`$?`来保存上个已执行命令的退出状态码。
```bash
date
echo $?
```
Linux退出状态码
|状态码|描述|
|---|---|
|0|命令成功结束|
|1|一般性未知错误|
|2|不适合的shell命令|
|126|命令不可执行|
|127|没找到命令|
|128|无效的退出参数|
|128+x|与Linux信号x相关的严重错误|
|130|通过Ctrl+C终止的命令|
|255|正常范围之外的退出状态码|
 
`exit` 命令
`exit`命令允许你在脚本结束时指定一个退出状态码。
```bash
#!/bin/bash
# testing the exit status
var1=10
var2=30
var3=$[$var1 + $var2]
echo The answer is $var3
exit 5
#也可以在exit命令的参数中使用变量。
exit $var1
```
**退出状态码最大只能是255**。
### 使用结构化命令
使用 if-then 语句
if-then语句格式:
```bash
if command
then
commands
fi
```
