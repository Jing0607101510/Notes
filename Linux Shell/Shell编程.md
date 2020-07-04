# Shell编程

Shell的作用是解释执行用户的命令，用户输入一条命令，Shell就解释执行一条，这种方式称为交互式（Interactive），Shell还有一种执行命令的方式称为批处理（Batch），用户事先写一个Shell脚本（Script），其中有很多条命令，让Shell一次把这些命令执行完，而不必一条一条地敲命令。Shell脚本和编程语言很相似，也有变量和流程控制语句，但Shell脚本是解释执行的，不需要编译，Shell程序从脚本中一行一行读取并执行这些命令，相当于一个用户把脚本中的命令一行一行敲到Shell提示符下执行。由于历史原因，UNIX系统上有很多种Shell：

- sh（Bourne Shell）：由Steve Bourne开发，各种UNIX系统都配有sh。
- csh（C Shell）：由Bill Joy开发，随BSD UNIX发布，它的流程控制语句很像C语言，支持很多Bourne Shell所不支持的功能：作业控制，命令历史，命令行编辑。
- ksh（Korn Shell）：由David Korn开发，向后兼容sh的功能，并且添加了csh引入的新功能，是目前很多UNIX系统标准配置的Shell，在这些系统上/bin/sh往往是指向/bin/ksh的符号链接。
- tcsh（TENEX C Shell）：是csh的增强版本，引入了命令补全等功能，在FreeBSD、MacOS X等系统上替代了csh
- bash（Bourne Again Shell）：由GNU开发的Shell，主要目标是与POSIX标准保持一致，同时兼顾对sh的兼容，bash从csh和ksh借鉴了很多功能，是各种Linux发行版标准配置的Shell，在Linux系统上/bin/sh往往是指向/bin/bash的符号链接。虽然如此，bash和sh还是有很多不同的，一方面，**bash扩展了一些命令和参数，另一方面，bash并不完全和sh兼容**，有些行为并不一致，所以bash需要模拟sh的行为：当我们通过sh这个程序名启动bash时，bash可以假装自己是sh，不认扩展的命令，并且行为与sh保持一致。

```Shell
itcast$ vim /etc/passwd
	其中最后一列显示了用户对应的shell类型
	root:x:0:0:root:/root:/bin/bash
	nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
	syslog:x:101:103::/home/syslog:/bin/false
	itcast:x:1000:1000:itcast,,,:/home/itcast:/bin/bash
	ftp:x:115:125:ftp daemon,,,:/srv/ftp:/bin/false
```

用户在命令行输入命令后，一般情况下Shell会fork并exec该命令，但是Shell的内建命令例外，执行内建命令相当于调用Shell进程中的一个函数，并不创建新的进程。以前学过的cd、alias、umask、exit等命令即是内建命令，凡是用which命令查不到程序文件所在位置的命令都是内建命令，内建命令没有单独的man手册，要在man手册中查看***\*内建命令\****，应该执行

``man bash-builtins``

如export、shift、if、eval、[、for、while等等。内建命令虽然不创建新的进程，但也会有Exit Status，通常也用0表示成功非零表示失败，虽然内建命令不创建新的进程，但执行结束后也会有一个状态码，也可以用特殊变量$?读出。





# 执行脚本

Shell脚本中用#表示注释，相当于C语言的//注释。但如果#位于第一行开头，并且是#!（称为Shebang）则例外，它表示该脚本使用后面指定的解释器/bin/sh解释执行。如果把这个脚本文件加上可执行权限然后执行：

```Shell
# test.sh
#! /bin/bash
cd ..
ls
# 

chmod a+x test.sh
./test.sh
```

**Shell会fork一个子进程并调用exec执行./test.sh这个程序**，exec系统调用应该把子进程的代码段替换成./test.sh程序的代码段，并从它的_start开始执行。然而test.sh是个文本文件，根本没有代码段和_start函数，怎么办呢？其实exec还有另外一种机制，如果要执行的是一个文本文件，并且第一行用Shebang指定了解释器，则用解释器程序的代码段替换当前进程，并且从解释器的_start开始执行，而这个文本文件被当作命令行参数传给解释器。因此，执行上述脚本相当于执行程序

``./test.sh 相当于 /bin/bash ./test.sh``, 而且``/bin/bash ./test.sh``的方式不需要test.sh文件具有可执行权限

如果将命令行下输入的命令用()括号括起来，那么也会fork出一个子Shell执行小括号中的命令，一行中可以输入由分号;隔开的多个命令，比如：

```Shell
itcast$ (cd ..;ls -l)
```

和上面两种方法执行Shell脚本的效果是相同的，**cd ..命令改变的是子Shell的PWD**，而不会影响到交互式Shell。然而命令， 而``./test.sh 相当于 /bin/bash ./test.sh``的cd..命令改变的是子进程的pwd，不影响shell进程的pwd。

```Shell
itcast$ cd ..;ls -l
```

则有不同的效果，cd ..命令是直接在交互式Shell下执行的，改变交互式Shell的PWD，然而这种方式相当于这样执行Shell脚本：

```Shell
itcast$ source ./test.sh

或者

itcast$ . ./test.sh
```

source或者.命令是Shell的内建命令，这种方式也不会创建子Shell，而是直接在交互式Shell下逐行执行脚本中的命令。







# 基本语法

## 变量

按照惯例，Shell变量通常由字母加下划线开头，由任意长度的字母、数字、下划线组成。有两种类型的Shell变量：1.环境变量。2.本地变量

### 环境变量

环境变量可以从父进程传给子进程，因此Shell进程的环境变量可以从当前Shell进程传给fork出来的子进程。用printenv命令可以显示当前Shell进程的环境变量。（fork子进程时，会复制环境变量，而exec后复制过来的本地变量抹去了，不可访问）

### 本地变量

只存在于**当前Shell进程**，用**set**命令可以显示当前Shell进程中定义的所有变量（包括本地变量和环境变量）和函数。使用``./test.sh或/bin/bash ./test.sh``是新创建了子进程执行脚本，而不是在当前Shell进程中执行，所以test.sh定义的局部变量和环境变量在当前Shell进程都是不可见的。（环境变量保存在用户空间，但是父子进程的环境变量是拷贝，改变父进程的环境变量不会影响子进程的环境变量）

一个**变量定义后仅存在于当前Shell进程**，它是***\*本地变量\****，用export命令可以把本地变量导出为环境变量，定义和导出环境变量通常可以一步完成：

```Shell
itcast$ export VARNAME=value
```

也可以分两步完成：

```Shell
itcast$ VARNAME=value

itcast$ export VARNAME
```

用***\*unset\****命令可以***\*删除\****已定义的环境变量或本地变量。

```Shell
itcast$ unset VARNAME
```

如果一个变量叫的值为VARNAME，用 ' VARNAME ' 可以表示它的值，在不引起歧义的情况下也可以用VARNAME表示它的值。

注意，在定义变量时不用“'”取变量值时要用。和C语言不同的是，Shell变量不需要明确定义类型，事实上**Shell变量的值都是字符串**，比如我们定义VAR=45，其实VAR的值是字符串45而非整数。**Shell变量不需要先定义后使用，如果对一个没有定义的变量取值，则值为空字符串。**





## 通配符代换（文件名代换Globbing）

这些用于匹配的字符称为通配符（Wildcard），如：* ? [ ] 具体如下：

```Shell
	* 匹配0个或多个任意字符
	? 匹配一个任意字符
	[若干字符] 匹配方括号中任意一个字符的一次出现

itcast$ ls /dev/ttyS*
itcast$ ls ch0?.doc
itcast$ ls ch0[0-2].doc
itcast$ ls ch[012] [0-9].doc
```

注意，**Globbing所匹配的文件名是由Shell展开**（使用""可以跳过Shell对通配符进行展开）的，也就是说在参数还没传给程序之前已经展开了，比如上述ls ch0[012].doc命令，如果当前目录下有ch00.doc和ch02.doc，则传给ls命令的参数实际上是这两个文件名，而不是一个匹配字符串。





## 命令代换

由“`”反引号括起来的也是一条命令，**Shell先执行该命令，然后将输出结果立刻代换到当前命令行中**。例如定义一个变量存放date命令的输出：

```Shell
itcast$ DATE=`date`
```

命令代换也可以用$()表示：

```Shell
DATE=$(date)
```





## 算术代换

使用\$(())，用于算术计算，(())中的Shell变量取值将转换成整数，同样含义的\$[ ]等价例如：

```Shell
itcast$ VAR=45

itcast$ echo $(($VAR+3))	等价于	echo $[VAR+3]或 $[$VAR+3] 或 $((VAR+3))
```

$(())中只能用+-*/和()运算符，并且只能做**整数运算**。

$[base#n]，其中base表示进制，n按照base进制解释，后面再有运算数，按十进制解释。

```Shell
echo $[2#10+11]
echo $[8#10+11]
echo $[16#10+11]
```





## 转义字符

和C语言类似，\在Shell中被用作转义字符，用于去除紧跟其后的单个字符的特殊意义（回车除外），换句话说，紧跟其后的字符取字面值。例如：

```Shell
itcast$ echo $SHELL
/bin/bash
itcast$ echo \$SHELL
$SHELL
itcast$ echo \\
\
```

还有一个字符虽然不具有特殊含义，但是要用它做文件名也很麻烦，就是-号。如果要创建一个文件名以-号开头的文件，这样是不正确的：

```Shell
itcast$ touch -hello

touch: invalid option -- h

Try `touch --help' for more information.
```

即使加上\转义也还是报错：

```Shell
itcast$ touch \-hello

touch: invalid option -- h

Try `touch --help' for more information.
```

因为各种UNIX命令都把-号开头的命令行参数当作命令的选项，而不会当作文件名。如果非要处理以-号开头的文件名，可以有两种办法：

```Shell
itcast$ touch ./-hello

或者

itcast$ touch -- -hello
```

\还有一种用法，在\后敲回车表示续行，Shell并不会立刻执行命令，而是把光标移到下一行，给出一个续行提示符>，等待用户继续输入，最后把所有的**续行**接到一起当作一个命令执行。例如：（相当于对回车的转义，）

```Shell
itcast$ ls \

\> -l

（ls -l命令的输出）
```







## 单引号

和C语言同，Shell脚本中的单引号和双引号一样都是字符串的界定符（双引号下一节介绍），而不是字符的界定符。**单引号用于保持引号内所有字符的字面值**，即使引号内的\和回车也不例外，但是字符串中不能出现单引号。如果引号没有配对就输入回车，Shell会给出续行提示符，要求用户把引号配上对。例如：

```Shell
itcast$ echo '$SHELL'

$SHELL

itcast$ echo 'ABC\（回车）

> DE'（再按一次回车结束命令）

ABC\

DE
```





## 双引号

被双引号用括住的内容，将被视为单一字串。**它防止通配符扩展**，但**允许变量扩展**。这点与单引号的处理方式不同

```Shell
itcast$ DATE=$(date)

itcast$ echo "$DATE"

itcast$ echo '$DATE'
```

再比如：

```Shell
itcast$ VAR=200

itcast$ echo $VAR	

200

itcast$ echo '$VAR'

$VAR

itcast$ echo "$VAR"

200
```

