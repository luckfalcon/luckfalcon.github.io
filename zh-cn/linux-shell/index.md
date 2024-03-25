# linux shell攻略

linux shell脚本攻略
<!--more-->
不能出现在文件名中的字符只有斜线(/)和空操作符(null)两个。
>- 斜线用来分割构成路径名的各个文件
>- 空操作符则用来终止一个路径名

`/`表示根目录，以`/`开头的路径为绝对路径，否则为相对路径。相对路径从工作目录开始解释

# linux+shell脚本攻略
[参考源码](http://www.packtput.com "教程源码")
## 预备知识
### 显示输出
`username@hostname$`或`root@hostname #`:`$` 表示普通用户， `#` 表示管理员用户root。
shell 脚本通常以 shebang(也作 hashbang) 起始:`#!/bin/bash`
shebang 是一个文本行，其中 `#!` 位于解释器路径之前。 `/bin/bash`是`Bash`的解释器命令路径。 `bash`
将以 `#` 符号开头的行视为注释。脚本中只有第一行可以使用shebang来定义解释该脚本所使用的解
释器。
- 脚本的执行方式有两种
(1) 将脚本名作为命令行参数：
`bash myScript.sh`
(2) 授予脚本执行权限，将其变为可执行文件:
```bash
chmod 755 myScript.sh
#chmod a+x myScript.sh #所有用户获得执行权限
./myScript.sh
```
```bash
$ ./sample.sh
#./表示当前目录
$ /home/path/sample.sh
#使用脚本的完整路径
```
`~` 表示主目录，它通常是 `/home/user` ，其中 `user` 是用户名，如果是`root`用户，则为 `/root` 。
shell 使用分号或换行符来分隔单个命令或命令序列。
```bash
$ cmd1;cmd2
```
等同于
```bash
$ cmd1
$ cmd2
```
`echo` 是用于终端打印的最基本命令。
默认情况下， `echo` 在每次调用后会添加一个换行符。
```bash
$ echo "Welcome to Bash"
#$ echo Welcome to Bash 同样可以输出
Welcome to Bash
$ echo 'text in quotes'#单引号也可以实现同样的效果
#双引号允许shell解释字符串中出现的特殊字符。单引号不会对其做任何解释。
```
如果需要打印像`!`这样的特殊字符，那就不要将其放入双引号中，而是使用单引号，或是在特殊字符之前加上一个反斜线（\）：
```bash
$ echo Hello world !
$ echo 'Hello world !'
$ echo "Hello world \!" #将转义字符放在前面
#输出结果
Hello world !
```
另一个可用于终端打印的命令是 `printf` 。该命令使用的参数和C语言中的 `printf` 函数一样。
```bash
$ printf "Hello world"
#输出结果
Hello world $
```
使用 `echo` 和 `printf` 的命令选项时，要确保选项出现在命令中的所有字符串之前，否则 Bash 会将其视为另外一个字符串。
### 使用变量与环境变量
**变量名**由一系列字母、数字和下划线组成，其中不包含空白字符。常用的惯例是在脚本中使用大写字母命名环境变量，使用驼峰命名法或小写字母命名其他变量。
所有的应用程序和脚本都可以访问环境变量。可以使用 `env` 或 `printenv` 命令查看当前shell
中所定义的全部环境变量。
```bash
$ env
#要查看其他进程的环境变量，可以使用如下命令
$ cat /proc/$PID/environ
#PID 是相关进程的进程ID（ PID 是一个整数）
```
我们可以使用 `pgrep` 命令获得`gedit`的进程ID:
```bash
#查看运行程序gedit的ID数值
$ pgrep gedit
12501
```
特殊文件`/proc/PID/environ`是一个包含环境变量以及对应变量值的列表。每一个变量以 `name=value` 的形式来描述，彼此之间由null字符（ \0 ）分隔。
生成一份易读的报表，可以将 `cat` 命令的输出通过管道传给 `tr` ，将其中的 `\0` 替换成 `\n`：
```bash
$ cat /proc/12501/environ | tr '\0' '\n'
```
varName 是变量名， value 是赋给变量的值。如果 value 不包含任何空白字符（例如空格），那么就不需要将其放入引号中，则必须使用单引号或双引号。
```bash
varName=value
```
**注意**：`var = value` 不同于 `var=value` 。把 `var=value` 写成 `var = value`是一个常见的错误。两边没有空格的等号是赋值操作符，加上空格的等号表示的是等量关系测试。
在变量名之前加上美元符号（$）就可以访问变量的内容。
```bash
var="value" #将"value"赋给变量var
echo $var   #显示var中的值
#也可以 echo ${var}
```
可以在 `printf` 、 `echo` 或其他命令的双引号中引用变量值
```bash
#!/bin/bash
#文件名:variables.sh
fruit=apple
count=5
echo "We have $count ${fruit}(s)"
We have 5 apple(s) #输出结果
```
shell使用空白字符来分隔单词，所以我们需要加上一对花括号来告诉shell这里的变量名是 fruit ，而不是 fruit(s)。
在 `PATH` 中添加一条新路径，可以使用如下命令:
```bash
#将/home/user/bin添加到了 PATH 中
 export PATH="$PATH:/home/user/bin"
#或者
PATH="$PATH:/home/user/bin"
export PATH
```
`export` 命令声明了将由子进程所继承的一个或多个变量。可以将一个变量设置为环境变量，使得该变量在当前 Shell 进程中可被其他子进程访问到。
**注意**：使用单引号时，变量不会被扩展（expand），仍依照原样显示。这意味着 `$ echo '$var'` 会显示 `$var` 。
- 获得字符串的长度：
```bash
var=123456789
length=${#var}
```
- 识别当前所使用的shell：
```bash
echo $SHELL
#或者
echo $0
```
- 检查是否为超级用户：
```bash
#!/bin/bash
#环境变量 UID 中保存的是用户ID
#它可以用于检查当前脚本是以root用户还是以普通用户的身份运行的
#root用户的 UID 是 0
# [ 实际上是一个命令，必须将其与剩余的字符串用空格隔开
If [ $UID -ne 0 ]; then
echo Non root user. Please run as root.
else
echo Root user
fi
```
等价于
```bash
If test $UID -ne 0:1
then
echo Non root user. Please run as root.
else
echo Root user
fi
```
- 修改Bash的提示字符串（ username@hostname:~$ ）
```bash
#利用 PS1 环境变量来定义主提示字符串
#默认的提示字符串是在文件 ~/.bashrc 中的某一行设置的
#查看设置变量 PS1 的那一行
$ cat ~/.bashrc | grep PS1
PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
#修改提示字符串，可以输入
$ PS1="PROMPT>" #提示字符串已经改变
PROMPT> Type commands here.
```
还有一些特殊的字符可以扩展成系统参数。例如： `\u` 可以扩展为用户名， `\h` 可以扩展为主机名，而 `\w` 可以扩展为当前工作目录。
### 使用函数添加环境变量
环境变量通常保存了可用于搜索可执行文件、库文件等的路径列表。
```bash
export PATH=/opt/myapp/bin:$PATH
export LD_LIBRARY_PATH=/opt/myapp/lib; $LD_LIBRARY_PATH
#结果
PATH=/opt/myapp/bin:/usr/bin:/bin # : 路径分隔符
LD_LIBRARY_PATH=/opt/myapp/lib:/usr/lib; /lib
```
我们可以在.bashrc文件中定义如下函数，简化路径添加操作：
```bash
#定义函数
prepend() { [ -d "$2" ] && eval $1=\"$2':'\$$1\" && export $1; }
#用法
prepend PATH /opt/myapp/bin
prepend LD_LIBRARY_PATH /opt/myapp/lib
```
函数 prepend() 首先确认该函数第二个参数所指定的目录是否存在。如果存在， eval 表达式将第一个参数所指定的变量值设置成第二个参数的值加上 : （路径分隔符），随后再跟上第一个参数的原始值。
在进行添加时，如果变量为空，则会在末尾留下一个 : 。要解决这个问题，可以对该函数再做一些修改：
```bash
prepend() { [ -d "$2" ] && eval $1=\"$2"\$\$$1\" && export $1; }
```
在这个函数中，我们引入了一种shell参数扩展的形式：
```bash
${parameter:+expression}
```
如果 parameter 有值且不为空，则使用 expression 的值。
通过这次修改，在向环境变量中添加新路径时，当且仅当旧值存在，才会增加 : 。
### 使用 shell 进行数学运算
Bash shell使用 let 、 (( )) 和 [ ] 执行基本的算术操作。工具 `expr` 和 `bc` 可以用来执行高级操作。
- `let`命令可以直接执行基本的算术操作。当使用 `let` 时，变量名之前不需要再添加 `$ `。
```bash
no1=4;
no2=5;
let result=no1+no2;
echo $result
```
`let` 命令的其他用法如下：
自加操作
```bash
$ let no1++
```
自减操作
```bash
$ let no1--
```
简写形式
```bash
let no+=6
let no-=6
#它们分别等同于 let no=no+6 和 let no=no-6
```
- 操作符 [ ] 的使用方法和 let 命令一样：
```bash
result=$[ no1 + no2 ] #注意空格[]前后位置都要有空格
result=$[ $no1 + 5 ]
```
- 也可以使用操作符 (()) 。出现在 (()) 中的变量名之前需要加上 `$` ：
```bash
result=$(( no1 + 50 ))
```
- expr 同样可以用于基本算术操作：
```bash
result=`expr 3 + 4`
result=$(expr $no1 + 5)
```
**注意：以上这些方法不支持浮点数，只能用于整数运算。**
- `bc` 是一个用于数学运算的高级实用工具，这个精密的计算器包含了大量的选项。我们可以借助它执行浮点数运算并使用一些高级函数：
```bash
echo "4 * 0.56" | bc
2.24
no=54;
#result=`echo "$no * 1.5" | bc` #反引号旧语法，推荐使用下面的$()
result=$(echo "$no * 1.5" | bc)
echo $result
81.0
```
`bc` 可以接受操作控制前缀。这些前缀之间使用分号分隔。
设定小数精度：
```bash
#参数 scale=2 将小数位个数设置为 2
echo "scale=2;22/7" | bc
3.14
```
进制转换：
```bash
#!/bin/bash
#数字转换
no=100
echo "obase=2;$no" | bc #二进制,默认ibase为十进制
1100100
no=1100100
echo "obase=10;ibase=2;$no" | bc #obase：输出进制，ibase:输入进制
100
```
计算平方以及平方根：
```bash
echo "sqrt(100)" | bc #平方根
echo "10^10" | bc #平方
```
### 文件描述符与重定向
文件描述符是与输入和输出流相关联的**整数**。常见的文件描述符有`stdin`(标准输入)、`stdout`(标准输出)和`stderr`(标准错误)。我们可以将某个文件描述符的内容重定向到另一个文件描述符中。
脚本可以使用大于号(>)将输出重定向到文件中,命令产生的文本可能是正常输出，也可能是错误信息。默认情况下，正常输出（ stdout ）和错误信息（ stderr ）都会显示在屏幕上。
文件描述符是与某个打开的文件或数据流相关联的整数。文件描述符 0 、 1 以及 2 是系统预留的:
> 0 —— stdin （标准输入）
> 1 —— stdout （标准输出）
> 2 —— stderr （标准错误）

使用大于号(>)将文本保存到文件中(若文件存在，则先清空文件内的文本，再写入文本)：
```bash
$ echo "This is a sample text 1" > temp.txt
#该命令会将输出的文本保存在temp.txt中。
#如果temp.txt已经存在，大于号会清空该文件中
先前的内容。
```
使用双大于号(>>)将文本追加到文件中(不会清空文件中的内容)：
```bash
echo "This is sample text 2" >> temp.txt
```
使用 `cat` 查看文件内容：
```bash
$ cat temp.txt
This is sample text 1
This is sample text 2
```
重定向 `stderr`:
```bash
$ ls +
ls: cannot access +: No such file or directory
#这里， + 是一个非法参数，因此会返回错误信息。
$ ls + > out.txt #因为 stdout 并没有输出，所以out.txt的内容为空
$ ls + 2> out.txt #重定向到out.txt
```
将 `stderr` 和 `stdout` 分别重定向到不同的文件中：
```bash
$ cmd 2>stderr.txt 1>stdout.txt
```
将`stderr` 和 `stdout` 都重定向到同一个文件中：
```bash
$ cmd 2>&1 alloutput.txt
#或者这样
$ cmd &> output.txt
```
如果你不想看到或保存错误信息，那么可以将 `stderr` 的输出重定向到`/dev/null`(一个特殊的设备文件，它会丢弃接收到的任何数据)。
我们在处理一些命令输出的同时还想将其保存下来，以备后用。 `stdout` 作为**单数据流（single stream），可以被重定向到文件或是通过管道传入其他程序，但是无法两者兼得**。
有一种方法**既可以将数据重定向到文件，还可以提供一份重定向数据的副本作为管道中后续命令**的 `stdin` 。 `tee` 命令从 `stdin` 中读取，然后将输入数据重定向到 `stdout` 以及一个或多个文件中。
```bash
command | tee FILE1 FILE2 | otherCommand
```
命令 `cat -n` 为从 `stdin` 中接收到的每一行数据前加上行号并将其写入 `stdout` ：
```bash
$ cat a* | tee out.txt | cat -n
cat: a1: Permission denied
1 A2
2 A3
```
使用 cat 查看out.txt的内容：
```bash
$ cat out.txt
A2
A3
```
**注意**：`cat: a1: Permission denied` 并没有在文件内容中出现，因为这些信息被发送到了 `stderr` ，而 `tee` 只能从 `stdin` 中读取。
默认情况下， `tee` 命令会将文件覆盖，但它提供了一个 `-a` 选项，可用于追加内容。
```bash
$ cat a* | tee -a out.txt | cat -n
```
带有参数的命令可以写成： `command FILE1 FILE2 ...` ，或者就简单地使用 `command FILE` 。
要发送输入内容的两份副本给 `stdout` ，使用 `-` 作为命令的文件名参数即可：
```bash
#$ cmd1 | cmd2 | cmd -
$ echo who is this | tee -
who is this
who is this
```
默认情况下，**重定向操作**针对的是**标准输出**。
- 从 `stdin` 读取输入的命令能以多种方式接收数据。可以用 `cat` 和管道来指定我们自己的文件描述符。
- 将文件重定向到命令：
```bash
#借助小于号（ < ），我们可以像使用 stdin 那样从文件中读取数据
$ cmd < file
```
- 重定向脚本内部的文本块
```bash
#!/bin/bash
#可以将脚本中的文本重定向到文件。
#要想将一条警告信息添加到自动生成的文件顶部，可以使用下面的代码:
cat<<EOF>log.txt #重定义文件结束符 EOF
This is a generated file. Do not edit. Changes will be overwritten.
EOF
#出现在 cat <<EOF>log.txt 与下一个 EOF 行之间的所有文本行都会被当作 stdin 数据。log.txt文件的内容显示如下:
$ cat log.txt
This is a generated file. Do not edit. Changes will be overwritten.
```
- 自定义文件描述符
文件描述符是一种用于访问文件的抽象指示器存取文件离不开被称为"文件描述符"的特殊数字。 0 、 1 和 2 分别是`stdin` 、 `stdout` 和 `stderr` 预留的描述符编号。
`exec` 命令创建全新的文件描述符。常用的打开模式有3种：
(1)只读模式。<
(2)追加写入模式。>>
(3)截断写入模式。>

创建一个用于读取文件的文件描述符：
```bash
$ # exec 3<input.txt #使用文件描述符3打开并读取文件
$ echo this is a test line > input.txt
$ exec 3<input.
```
`<&`：将输入重定向到指定的文件描述符。
在命令中使用文件描述符 3：
```bash 
$ cat<&3
this is a test line
```
如果要再次读取，我们就不能继续使用文件描述符 3 了，而是需要用 `exec` 重新创建一个新的文件描述符（可以是 4 ）来从另一个文件中读取或是重新读取上一个文件。
创建一个用于写入（截断模式）的文件描述符：
```bash
$ #exec 4>output.txt #打开文件进行写入
$ exec 4>output.txt
$ echo newline >&4
$ cat output.txt
newline
```
创建一个用于写入（追加模式）的文件描述符：
```bash
#$ exec 5>>input.txt
$ exec 5>>input.txt
$ echo appended line >&5
$ cat input.txt
newline
appended line
```
### 数组与关联数组
Bash从4.0版本才开始支持关联数组
数组允许脚本利用索引将数据集合保存为独立的条目。Bash支持普通数组和关联数组，前者使用整数作为数组索引，后者使用字符串作为数组索引。当数据以数字顺序组织的时候，应该使用普通数组，例如一组连续的迭代。当数据以字符串组织的时候，关联数组就派上用场了，例如主机名称。
- 定义数组的方法
```bash
#这些值将会存储在以0为起始索引的连续位置上
array_var=(test1 test2 test3 test4)
#将数组定义成一组“索引-值”
array_var[0]="test1"
array_var[1]="test2"
array_var[2]="test3"
array_var[3]="test4"
```
- 打印出特定索引的数组元素内容
```bash
echo ${array_var[0]}
test1
index=3
echo ${array_var[$index]}
test4
```
- 以列表形式打印出数组中的所有值
```bash
$ echo ${array_var[*]}
test1 test2 test3 test4
#或者
$ echo ${array_var[@]}
test1 test2 test3 test4
```
- 打印数组长度（即数组中元素的个数）
```bash
$ echo ${#array_var[*]}
4
```
- 定义关联数组
```bash
#使用声明语句将一个变量定义为关联数组
$ declare -A ass_array
```
可以用下列两种方法将元素添加到关联数组中
- 使用行内"索引 - 值"列表
```bash
$ ass_array=([index1]=val1 [index2]=val2)
```
- 使用独立的"索引 - 值"进行赋值：
```bash
$ ass_array[index1]=val1
$ ass_array[index2]=val2
```
- 列出数组索引(普通数组和关联数组均可使用)
```bash
$ echo ${!array_var[*]}
#或者
$ echo ${!array_var[@]}
```
### 别名
使用 `alias` 命令创建别名。
- 创建别名
```bash
#为 apt-get install 创建了一个别名
$ alias install='sudo apt-get install'
```
`alias` 命令的效果只是暂时的。一旦关闭当前终端，所有设置过的别名就失效了。为了使别名在所有的shell中都可用，可以将其定义放入`~/.bashrc`文件中。每当一个新的交互式shell进程生成时，都会执行 `~/.bashrc`中的命令。
```bash
#添加别名
$ echo 'alias cmd="command seq"' >> ~/.bashrc
```
- 删除别名
只需将其对应的定义（如果有的话）从`~/.bashrc`中删除，或者使用unalias 命令。也可以使用 `alias example=` ，这会取消别名 example。

**注意**：创建别名时，如果已经有同名的别名存在，那么原有的别名设置将被新的设置取代。
- 创建别名并为原文件保留一个副本
```bash
#创建一个别名 rm ，它能够删除原始文件，同时在backup目录中保留副本
alias rm='cp $@ ~/backup && rm $@'
```
>- `$@` 的含义：`$@` 表示所有的位置参数，每个参数都是一个独立的字符串。
>- 在使用 `$@` 时，最好将其放在双引号中以避免参数的空格和特殊字符被解释为多个参数。这样可以确保每个参数都作为独立的字符串进行处理。
>- `$@` 和 `$*` 的区别：`$@`和 `$*` 都表示所有的位置参数，但在引用时有所不同。`$@` 会将每个参数作为独立的引用，并保留参数中的空白和特殊字符；而 `$*` 会将所有参数作为单个字符串引用，并在参数之间插入第一个字符处定义的特殊字符（通常是空格）。

- 对别名进行转义
创建一个和原生命令同名的别名很容易，你不应该以特权用户的身份运行别名化的命令。我们可以转义要使用的命令，忽略当前定义的别名。
```bash
$ \command
```
字符 \ 可以转义命令，从而执行原本的命令。在不可信环境下执行特权命令时，在命令前加上 \ 来忽略可能存在的别名总是一种良好的安全实践。这是因为攻击者可能已经将一些别有用心的命令利用别名伪装成了特权命令，借此来盗取用户输入的重要信息。
- 列举别名
```bash
alias
alias ll='ls -l'
alias lt='ls -t'
alias la='ls -a'
...
```
### 采集终端信息
`tput` 和 `stty` 是两款终端处理工具。
- 获取终端的行数和列数：
```bash
tput cols
tput lines
```
- 打印出当前的终端名：
```bash
tput longname
```
- 将光标移动到坐标(100,100)处：
```bash
tput cup 100 100
```
- 设置终端背景色：
```bash
tput setb n
#n 可以在0到7之间取值
```
- 设置终端前景色：
```bash
tput setf n
#n 可以在0到7之间取值
```
**注意**：包括常用的 `color ls` 在内的一些命令可能会重置前景色和背景色。
- 设置文本样式为粗体：
```bash
tput bold
```
- 设置下划线的起止：
```bash
tput smul
tput rmul
```
- 删除从当前光标位置到行尾的所有内容：
```bash
tput ed
```
- 输入密码时，脚本不应该显示输入内容。在下面的例子中，我们将看到如何使用 `stty` 来实现这一需求：
```bash
#!/bin/sh
#Filename: password.sh
echo -e "Enter password: "
# 在读取密码前禁止回显
stty -echo
read password
# 重新允许回显
stty echo
echo $password #显示输入的密码
```
`stty` 命令的选项 `-echo` 禁止将输出发送到终端，而选项 `echo` 则允许发送输出。
### 获取并设置日期及延时
在系统内部，日期被存储成一个整数，其取值为自1970年1月1日0时0分0秒 起所流逝的秒数。这种计时方式称为纪元时或Unix时间。
- 读取日期：
```bash
$ date
Thu May 20 23:09:04 IST 2010
2024年 03月 21日 星期四 22:52:24 CST
```
- 打印纪元时：
```bash
$ date +%s
1711032681
```
- 将日期转换成纪元时：
```bash
$ date --date "Wed mar 15 08:09:16 EDT 2017" +%s
$ date -d "Wed mar 15 08:09:16 EDT 2017" +%s
#这条命令无法转带中文格式日期
1489579718
```
- 用带有前缀 `+` 的格式化字符串作为 `date` 命令的参数，可以按照你的选择打印出相应格式的日期：
```bash
$ date "+%d %B %Y"
20 May 2010
```
- 设置日期和时间：
```bash
## date -s "格式化的日期字符串"
date -s "21 June 2009 11:01:22"
#如果系统已经联网，可以使用 ntpdate 来设置日期和时间：
/usr/sbin/ntpdate -s time-b.nist.gov
```
- date 命令可以用于计算一组命令所花费的执行时间:
```bash
#!/bin/bash
#文件名: time_take.sh
start=$(date +%s) #date 命令的最小精度是秒,对命令计时的另一种更好的方式是使用 time 命令
commands;
statements;
end=$(date +%s)
difference=$(( end - start))
echo Time taken to execute commands is $difference seconds.
```
- 在脚本中生成延时：
```bash
#!/bin/bash
#文件名: sleep.sh
echo Count:
tput sc
# 循环40秒
for count in `seq 0 40`
do
tput rc
tput ed
echo -n $count #-n,不输出尾部换行符
sleep 1
done
```
在上面的例子中，变量依次使用了由 `seq` 命令生成的一系列数字。我们用 `tput sc` 存储光标位置。在每次循环中，通过 `tput rc` 恢复之前存储的光标位置，在终端中打印出新的 count 值，然后使用 `tputs ed` 清除从当前光标位置到行尾之间的所有内容。行被清空之后，脚本就可以显示出新的值。 `sleep` 可以使脚本在每次循环迭代之间延迟1秒钟。
### 调试脚本
可以利用Bash内建的调试工具或者按照易于调试的方式编写脚本。
- 使用选项 `-x`，启用shell脚本的跟踪调试功能：
```bash
$ bash -x script.sh
#或
sh -x script
```
运行带有 `-x` 选项的脚本可以打印出所执行的每一行命令以及当前状态。
- 使用 `set -x` 和 `set +x` 对脚本进行部分调试。
```bash
#!/bin/bash
#文件名: debug.sh
for i in {1..6};
do
set -x #-x与+x包裹部分是被限制的调试区域
echo $i
set +x
done
echo "Script executed"
```
该脚本并没有使用上例中的 `seq` 命令，而是用 `{start..end}` 来迭代从 `start` 到 `end` 之间的值。这个语言构件（construct）在执行速度上要比 `seq` 命令略快。
- 定义 `_DEBUG` 环境变量来启用或禁止调试及生成特定形式的信息。
```bash
#!/bin/bash
#名字: debug.sh 
function DEBUG()
{
[ "$_DEBUG" == "on" ] && $@ || : #[]为真执行$@(位置参数)，[]为假执行 ||后面的:,shell空命令(这里当作占位符)
}
for i in {1..10}
do
DEBUG echo "I is $i"
done
```
将调试功能设置为 `on` 来运行上面的脚本：
```bash
_DEBUG=on ./debug.sh #需有x权限
```
我们在每一条需要打印调试信息的语句前加上 DEBUG 。如果没有把 `_DEBUG=on` 传递给脚本，那么调试信息就不会打印出来。在Bash中，命令 `:` 告诉shell不要进行任何操作。
- `set -x` ：在执行时显示参数和命令。
- `set +x` ：禁止调试。
- `set -v` ：当命令进行读取时显示输入。
- `set +v` ：禁止打印输入。
- shebang的妙用
把shebang从 `#!/bin/bash` 改成 `#!/bin/bash -xv` ，这样一来，不用任何其他选项就可以启用调试功能了。
- 以将环境变量PS4设置为 `'$LINENO: '` ，显示出每行的行号：
```bash
PS4='$LINENO: '
```
> ``（反引号）用于执行命令替换，将命令的输出结果作为替换结果。
> ''（单引号）用于创建字符串字面值，保持所有字符的原样性，不进行变量替换或命令执行。

使用了 `-x`或 `set -x` ，调试输出会被发送到 `stderr`，可以使用下面的命令将其重定向到文件中：
```bash
sh -x testScript.sh 2> debugout.txt
```
Bash 4.0以及后续版本支持对调试输出使用自定义文件描述符：
```bash
exec 6> /tmp/debugout.txt
BASH_XTRACEFD=6
```
> `BASH_XTRACEFD=6` 是一个环境变量设置，它指示Bash Shell将跟踪（trace）输出重定向到文件描述符6。通常，Bash Shell使用文件描述符2（标准错误）来输出跟踪信息，但通过将`BASH_XTRACEFD`设置为6，你可以将跟踪输出重定向到文件描述符6，从而将跟踪信息写入`/tmp/debugout.txt` 文件。
- 函数和参数
函数和别名乍一看很相似，不过两者在行为上还是略有不同。最大的差异在于函数参数可以在函数体中任意位置上使用，而别名只能将参数放在命令尾部。
函数的定义包括function命令、函数名、开/闭括号以及包含在一对花括号中的函数体。
- 函数可以这样定义：
```bash
function fname()
{
statements;
}
#或者
fname()
{
statements;
}
#或者
fname() { statement; }
```
- 只需使用函数名就可以调用函数
```bash
$ fname ; #执行函数
```
- 函数参数可以按位置访问， `$1` 是第一个参数， `$2` 是第二个参数，以此类推：
```bash
fname arg1 arg2 ; #传递参数
```
以下是函数 `fname` 的定义。在函数 `fname` 中，包含了各种访问函数参数的方法。
```bash
fname()
{
echo $1, $2; #访问参数1和参数2
echo "$@"; #以列表的方式一次性打印所有参数
echo "$*"; #类似于$@，但是所有参数被视为单个实体
return 0; #返回值
}
```
传入脚本的参数可以通过下列形式访问。
>- `$0` 是脚本名称。
>- `$1` 是第一个参数。
>- `$2` 是第二个参数。
>- `$n` 是第n个参数。
>- `"$@"` 被扩展成 `"$1" "$2" "$3"` 等。
>- `"$*"` 被扩展成 `"$1c$2c$3"` ，其中 `c` 是IFS的第一个字符。
>- `$@`要比`$*`用得多。由于`$*`将所有的参数当作单个字符串,因此它很少被使用。
- grep命令找到的是字符串`eth0`,而不是IP地址。如果我们使用函数来实现的话,可以将设备名作为参数传入`ifconfig`,不再交给 `grep` :
```bash
$> function getIP() { /sbin/ifconfig $1 | grep 'inet '; }
$> getIP eth0
inet addr:192.168.1.2 Bcast:192.168.255.255 Mask:255.255.0.0
#直接方式也可以获得
$> ifconfig eth0 | grep "inet"
inet addr:192.168.1.2 Bcast:192.168.255.255 Mask:255.255.0.0
```
Bash函数的技巧
在Bash中,函数同样支持递归调用(可以调用自身的函数),如`F() { echo $1; F hello;sleep 1; }`
- 递归函数
[Fork炸弹](https://en.wikipedia.org/wiki/Fork_bomb "Fork bomb")
```bash
:(){ :|:& };:
#这个函数会一直地生成新的进程,最终形成拒绝服务攻击。
#函数调用前的 &将子进程放入后台。
#这段危险的代码能够不停地衍生出进程,因而被称为Fork炸弹。
```
可以通过修改配置文件`/etc/security/limits.conf`中的 `nproc` 来限制可生成的最大进程数,进而阻止这种攻击。
```bash
#将所有用户可生成的进程数限制为100
hard nproc 100
```
-  导出函数
函数也能像环境变量一样用export导出,如此一来,函数的作用域就可以扩展到子进程中:
```bash
export -f fname
$> function getIP() { /sbin/ifconfig $1 | grep 'inet '; }
$> echo "getIP eth0" >test.sh
$> sh test.sh #sh 无法使用函数
sh: getIP: No such file or directory
$> export -f getIP
$> sh test.sh
test.sh: 2: getip: not found 
$> bash test.sh #sh 不支持export命令
inet addr: 192.168.1.2 Bcast: 192.168.255.255 Mask:255.255.0.0
```
-  读取命令返回值(状态)
```bash
cmd
echo $?
#返回值被称为退出状态。它可用于确定命令执行成功与否。
#如果命令成功退出,那么退出状态为0,否则为非0。
```
下面的脚本可以报告命令是否成功结束:
```bash
#!/bin/bash
#文件名: success_test.sh
#对命令行参数求值,比如success_test.sh ‘ls | grep txt’
eval $@ #eval $@ 是一个在Shell脚本中常见的用法，它用于执行传递给脚本的命令行参数。当使用 eval $@ 时，eval 命令会对 $@ 中的参数进行解析和执行。
if [ $? -eq 0 ];
then
echo "$CMD executed successfully"
else
echo "$CMD terminated unsuccessfully"
fi
```
-  向命令传递参数
`$#` 是一个特殊变量，用于获取传递给脚本的命令行参数的数量。
`shift`命令可以将参数依次向左移动一个位置,让脚本能够使用$1 来访问到每一个参数。下面的代码显示出了所有的命令行参数:
```bash
 $ cat showArgs.sh
for i in `seq 1 $#`
do
echo $i is $1
shift
done
$ sh showArgs.sh a b c
1 is a
2 is b
3 is c
```
### 将一个命令的输出发送给另一个命令
使用管道 `|` 来连接多条命令
```bash
$ cmd1 | cmd2 | cmd3
```
- 组合两个命令：
```bash
$ ls | cat -n > out.txt # -n，加上行号
```
- 将命令序列的输出赋给变量:
```bash
#这种方法叫作子shell法
cmd_output=$(ls | cat -n)
echo $cmd_output
#或者，反引号
cmd_output=`COMMANDS`
```
- 利用子shell生成一个独立的进程
子shell本身就是独立的进程。可以使用()操作符来定义一个子shell。
```bash
$> pwd
/
$> (cd /bin; ls)
awk bash cat...
$> pwd
/
```
当命令在子shell中执行时,不会对当前shell造成任何影响;所有的改变仅限于该子shell内。例如,当用`cd`命令改变子shell的当前目录时,这种变化不会反映到主shell环境中。
- 通过引用子shell的方式保留空格和换行符
```bash
$ cat text.txt
1
2
3
$ out=$(cat text.txt)
$ echo $out
1 2 3
 # 丢失了1、2、3中的\n
$ out="$(cat text.txt)"
$ echo $out
1
2
3
```
### 在不按下回车键的情况下读入 n 个字符
`read`命令提供了一种不需要按回车键就能够搞定这个任务的方法。
- 句从输入中读取n个字符并存入变量variable_name:
```bash
read -n number_of_chars variable_name
$ read -n 2 var #读入最多2个字符终止输入
$ echo $var
```
- 用无回显的方式读取密码:
```bash
read -s var
```
- 使用read显示提示信息:
```bash
read -p "Enter input:"
var
```
- 在给定时限内读取输入:
```bash
#read -t timeout var
$ read -t 2 var
#在2秒内将键入的字符串读入变量var
```
- 用特定的定界符作为输入行的结束:
```bash
#read -d delim_char var
$ read -d ":" var
hello:
 #var被设置为hello
```
### 持续运行命令直至执行成功
```bash
repeat()
{
while true
do
$@ && return
done
}
#函数repeat()中包含了一个无限while循环,
#该循环执行以函数参数形式(通过$@访问)传入的命令。
#如果命令执行成功,则返回,进而退出循环。
```
- 一种更快的做法
`true`是作为`/bin`中的一个二进制文件来实现的。这就意味着每执行一次之前提到的`while`循环,shell就不得不生成一个进程。为了避免这种情况,可以使用shell的内建命令`:`,该命令的退出状态总是为0:
```bash
repeat() { while :; do $@ && return; done }
#尽管可读性不高,但是肯定比第一种方法快
```
- 加入延时
```bash
repeat() { while :; do $@ && return; sleep 30; done }
#sleep 30秒
```
### 字段分隔符与迭代器
内部字段分隔符(Internal Field Separator,**IFS**)是shell脚本编程中的一个重要概念。在处理文本数据时,它的作用可不小。
它是一个环境变量,其中保存了用于分隔的字符。
```bash
#CSV数据data
data="name, gender,rollno,location"
oldIFS=$IFS
IFS=, #IFS现在被设置为,
for item in $data;
do
echo Item: $item
done
IFS=$oldIFS
```
**注意**：`IFS`的默认值为空白字符(换行符、制表符或者空格)。
以`/etc/passwd`为例,看看IFS的另一种用法。在文件`/etc/passwd`中,每一行包含了由冒号分隔的多个条目。该文件中的每行都对应着某个用户的相关属性。每行的最后一项指定了用户的默认shell。
- 利用IFS打印出用户以及他们默认的shell：
```bash
#!/bin/bash
#用途: 演示IFS的用法
line="root:x:0:0:root:/root:/bin/bash"
oldIFS=$IFS;
IFS=":"
count=0
for item in $line;
do
[ $count -eq 0 ] && user=$item;
[ $count -eq 6 ] && shell=$item;
let count++
done;
IFS=$oldIFS
#echo $user's shell is $shell;
# unexpected EOF while looking for matching `''
#syntax error: unexpected end of file
echo "$user's shell is $shell"
```
- 面向列表的for循环：
```bash
for var in list; 
do
commands;
 #使用变量$var
done
```
list可以是一个字符串,也可以是一个值序列。
```bash
echo {1..50};  #生成一个从1~50的数字序列
echo {a..z} {A..Z};  #生成大小写字母序列
```
- 迭代指定范围的数字：
```bash
for((i=0;i<10;i++)) #双层小括号()
{
commands;
 #使用变量$i
}
```
- 循环到条件满足为止：
```bash
while condition
do
commands;
done
```
- `until`循环：
在Bash中还可以使用一个特殊的循环`until`。它会一直循环，直到给定的条件为真。
```bash
x=0;
until [ $x -eq 9 ];
 #条件是[$x -eq 9 ]
do
let x++; echo $x;
done
```
###  比较与测试
用`if、if else` 以及逻辑运算符来测试。还有一个 `test`命令也可以用于测试。
- `if`条件：
```bash
if condition;
then
commands;
fi
```
- `else if`和`else`：
```bash
if condition;
then
commands;
else if condition; then
commands;
else
commands;
fi
```
- `if`的条件判断部分可能会变得很长，但可以用
逻辑运算符将它变得简洁一些：
```bash
[ condition ] && action;  # 如果condition为真,则执行action
[ condition ] || action;  # 如果condition为假,则执行action
```
- 算术比较
```bash
[$var -eq 0 ] or [ $var -eq 0] #注意在[或]与操作数之间有一个空格
[ $var -eq 0 ]  #当$var等于0时,返回真
[ $var -ne 0 ]  #当$var不为0时,返回真
#-gt:大于
#-lt:小于
#-ge:大于或等于
#-le:小于或等于
[ $var1 -ne 0 -a $var2 -gt 2 ]  #使用逻辑与-a
[ $var1 -ne 0 -o $var2 -gt 2 ]  #逻辑或-o
```
- 文件系统相关测试
>-  `[ -f $file_var ]`：如果给定的变量包含正常的文件路径或文件名，则返回真。
>- `[ -x $var ]`：如果给定的变量包含的文件可执行，则返回真。
>- `[ -d $var ]`：如果给定的变量包含的是目录，则返回真。
>- `[ -e $var ]`：如果给定的变量包含的文件存在，则返回真。
>- `[ -c $var ]`：如果给定的变量包含的是一个字符设备文件的路径，则返回真。
>- `[ -b $var ]`：如果给定的变量包含的是一个块设备文件的路径，则返回真。
>- `[ -w $var ]`：如果给定的变量包含的文件可写，则返回真。
>- `[ -r $var ]`：如果给定的变量包含的文件可读，则返回真。
>- `[ -L $var ]`：如果给定的变量包含的是一个符号链接，则返回真。

```bash
fpath="/etc/passwd"
if [ -e $fpath ]; then
echo File exists;
else
echo Does not exist;
fi
```
- 字符串比较
进行字符串比较时,最好用双中括号，因为有时候采用单个中括号会产生错误。
**注意**：双中括号是Bash的一个扩展特性。如果出于性能考虑，使用`ash`或`dash`来运行脚本,那么将无法使用该特性。
- 常用字符比较:
```bash
#测试两个字符串是否相同
[[ $str1 = $str2 ]] #当 str1等于str2时,返回真。也就是说,str1和 str2包含的文本是一模一样的。
[[ $str1 == $str2 ]] #这是检查字符串是否相同的另一种写法。
#测试两个字符串是否不同
[[ $str1 != $str2 ]] #如果str1和str2不相同,则返回真
#找出在字母表中靠后的字符串
 [[ $str1 > $str2 ]] #如果str1的字母序比str2大,则返回真。
[[ $str1 < $str2 ]] #如果str1的字母序比str2小,则返回真。
#测试空串
[[ -z $str1 ]] #如果str1为空串,则返回真
[[ -n $str1 ]] #如果str1不为空串,则返回真
```
- 使用逻辑运算符 `&&` 和 `||`组合多个条件：
```bash
if [[ -n $str1 ]] && [[ -z $str2 ]] ;
then
commands;
fi
```
- `test`命令可以用来测试条件，用`test`可以避免使用过多的括号，增强代码的可读性：
```bash
if [ $var -eq 0 ]; then echo "True"; fi
#用test写法
if test $var -eq 0 ; then echo "True"; fi
```
**注意**：`test`是一个外部程序,需要衍生出对应的进程，而 `[` 是Bash的一个内部函数，因此后者的执行效率更高。`test`兼容于Bourne shell、ash、dash等。
###  使用配置文件定制 bash
Linux和Unix中能够放置定制脚本的文件不止一个。这些配置文件分为3类:登录时执行的、启动交互式shell时执行的以及调用shell处理脚本文件时执行的。
当用户登录shell时,会执行下列文件:
```bash
/etc/profile, 
$HOME/.profile,
 $HOME/.bash_login,
  $HOME/.bash_profile /
```
通过图形化登录管理器登入的不会执行`/etc/profile、
$HOME/.profile和$HOME/.bash_profile`这3个文件的。
因为图形化窗口管理器并不会启动shell。当你打开终端窗口时才会创建shell，但这个shell也不是登录shell。
如果`.bash_profile`或`.bash_login`文件存在,则不会去读取`.profile`文件。
## 命令之乐
实用的命令`grep、 awk 、 sed和find`。
### 用 `cat` 进行拼接
`cat`命令是一个经常会用到的简单命令，它本身表示conCATenate(拼接)。
- `cat`读取文件内容的一般语法是：
```bash
cat file1 file2 file3 ...
```
该命令将作为命令行参数的文件内容拼接在一起并将结果发送到`stdout`。
```bash
#打印单个文件的内容
$ cat file.txt
#打印多个文件的内容
$ cat one.txt two.txt
```
`cat`命令不仅可以读取文件、拼接数据，还能够从标准输入中读取。
- 管道操作符可以将数据作为`cat`命令的标准输入:
```bash
# OUTPUT_FROM_SOME COMMANDS | cat
echo "hello" | cat
```
`cat`也可以将文件内容与终端输入拼接在一起。
- 将`stdin`和另一个文件中的数据组合在一起：
```bash
$ echo 'Text through stdin' | cat - file.txt
# - 被作为stdin文本的文件名
```
- 去掉多余的空白行：
```bash
$ cat -s file.txt
#将file中连续的空白行压缩为单行空白
#可以用 tr 删除所有的空白行
```
- 将制表符显示为`^I`。
对于Python而言，制表符和空格是区别对待的。
`cat`有一个特性，可以将制表符识别出来。
用`cat`命令的`-T`选项能够将制表符标记成 `^I`。
```bash
$ cat file.py
def function():
var = 5
next = 6
third = 7
$ cat -T file.py
def function():
^Ivar = 5
^I^Inext = 6
^Ithird = 7^I
```
