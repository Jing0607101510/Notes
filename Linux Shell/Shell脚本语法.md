# 条件测试

***\*命令test\****或 [ 可以测试一个条件是否成立，如果测试结果为真，则该命令的Exit Status为0，如果测试结果为假，则命令的Exit Status为1（注意与C语言的逻辑表示正好相反）。例如测试两个数的大小关系：

虽然看起来很奇怪，但左方括号 [ 确实是一个命令的名字，传给命令的各参数之间应该用空格隔开，比如：$VAR、-gt、3、] 是 [ 命令的四个参数，它们之间必须用空格隔开。命令test或 [ 的参数形式是相同的，只不过test命令不需要 ] 参数。以 [ 命令为例，常见的测试命令如下表所示：

```
[ -d DIR ] 如果DIR存在并且是一个目录则为真
[ -f FILE ] 如果FILE存在且是一个普通文件则为真
[ -z STRING ] 如果STRING的长度为零则为真
[ -n STRING ] 如果STRING的长度非零则为真
[ STRING1 = STRING2 ] 如果两个字符串相同则为真
[ STRING1 != STRING2 ] 如果字符串不相同则为真
[ ARG1 OP ARG2 ] ARG1和ARG2应该是***\*整数或者取值为整数的变量\****，OP是-eq（等于）-ne（不等于）-lt（小于）-le（小于等于）-gt（大于）-ge（大于等于）之中的一个
```

和C语言类似，测试条件之间还可以做与、或、非逻辑运算：

```
[ ! EXPR ] EXPR可以是上表中的任意一种测试条件，!表示“逻辑反(非)”
[ EXPR1 -a EXPR2 ] EXPR1和EXPR2可以是上表中的任意一种测试条件，-a表示“逻辑与”
[ EXPR1 -o EXPR2 ] EXPR1和EXPR2可以是上表中的任意一种测试条件，-o表示“逻辑或”
```

注意，如果上例中的$VAR变量事先没有定义，则被Shell展开为空字符串，会造成测试条件的语法错误（展开为[ -d Desktop -a = 'abc'  ]），**作为一种好的Shell编程习惯**，**应该总是把变量取值放在双引号之中**（展开为[ -d Desktop -a  "" = 'abc'  ]）：







# 分支

## if

和C语言类似，在Shell中用if、then、elif、else、fi这几条命令实现分支控制。这种流程控制语句本质上也是由若干条Shell命令组成的，例如先前讲过的

```Shell
if [ -f  ~/.bashrc ]; then
 	. ~/.bashrc
fi
```

其实是三条命令，if [ -f  ~/.bashrc ]是第一条，then . ~/.bashrc是第二条，fi是第三条。如果两条命令写在同一行则需要用;号隔开，一行只写一条命令就不需要写;号了，另外，then后面有换行，但这条命令没写完，Shell会自动续行，把下一行接在then后面当作一条命令处理。和[命令一样，要注意命令和各参数之间必须用空格隔开。if命令的参数组成一条子命令，如果该子命令的Exit Status为0（表示真），则执行then后面的子命令，如果Exit Status非0（表示假），则执行elif、else或者fi后面的子命令。if后面的子命令通常是测试命令，但也可以是其它命令。Shell脚本没有{}括号，所以用fi表示if语句块的结束。见下例：

```Shell
#! /bin/sh

if [ -f /bin/bash ]
then 
	echo "/bin/bash is a file"
else 
	echo "/bin/bash is NOT a file"
fi
if :; then echo "always true"; fi
```

“:”是一个特殊的命令，称为空命令，该命令不做任何事，但Exit Status总是真。此外，也可以执行/bin/true或/bin/false得到真或假的Exit Status。再看一个例子：

```Shell
#! /bin/sh

echo "Is it morning? Please answer yes or no."
read YES_OR_NO
if [ "$YES_OR_NO" = "yes" ]; then
	echo "Good morning!"
elif [ "$YES_OR_NO" = "no" ]; then
	echo "Good afternoon!"
else
	echo "Sorry, $YES_OR_NO not recognized. Enter yes or no."
	exit 1
fi
exit 0
```

此外，Shell还提供了&&和||语法，和C语言类似，具有Short-circuit特性，很多Shell脚本喜欢写成这样

``test "$(whoami)" != 'root' && (echo you are using a non-privileged account; exit 1)``

&&相当于“if…then…”，而||相当于“if not…then…”。&&和||用于连接两个命令，而上面讲的-a和-o仅用于在测试表达式中连接两个测试条件，要注意它们的区别，例如：

``test "$VAR" -gt 1 -a "$VAR" -lt 3``

和以下写法是等价的

``test "$VAR" -gt 1 && test "$VAR" -lt 3``

&& 和 || 用于连接命令。-a，-o用于连接测试条件



## case

case命令可类比C语言的switch/case语句，esac表示case语句块的结束。C语言的case只能匹配整型或字符型常量表达式，而Shell脚本的case可以匹配字符串和Wildcard，***\*每个匹配分支可以有若干条命令，末尾必须以;;结束\****，执行时找到第一个匹配的分支并执行相应的命令，然后直接跳到esac之后，不需要像C语言一样用break跳出。

```Shell
#! /bin/sh

echo "Is it morning? Please answer yes or no."
read YES_OR_NO
case "$YES_OR_NO" in
yes|y|Yes|YES)
	echo "Good Morning!";;
[nN]*)
	echo "Good Afternoon!";;
*)
	echo "Sorry, $YES_OR_NO not recognized. Enter yes or no."
	exit 1;;
esac
exit 0
```



# 循环

## for/do/done

Shell脚本的for循环结构和C语言很不一样，它类似于某些编程语言的foreach循环。例如：

```Shell
#! /bin/sh

for FRUIT in apple banana pear; do
	echo "I like $FRUIT"
done
```

FRUIT是一个循环变量，第一次循环$FRUIT的取值是apple，第二次取值是banana，第三次取值是pear。再比如，要将当前目录下的chap0、chap1、chap2等文件名改为chap0~、chap1~、chap2~等（按惯例，末尾有~字符的文件名表示临时文件），这个命令可以这样写：

```Shell
$ for FILENAME in chap?; do mv $FILENAME $FILENAME~; done
```

也可以这样写：

```Shell
$ for FILENAME in `ls chap?`; do mv $FILENAME $FILENAME~; done
```



## while/do/done

while的用法和C语言类似。比如一个验证密码的脚本：

```Shell
#! /bin/sh

echo "Enter password:"
read TRY
while [ "$TRY" != "secret" ]; do
	echo "Sorry, try again"
	read TRY
done
```

下面的例子通过算术运算控制循环的次数：

```Shell
#! /bin/sh

COUNTER=1
while [ "$COUNTER" -lt 10 ]; do
	echo "Here we go again"
	COUNTER=$(($COUNTER+1))
done
```

另，Shell还有until循环，类似C语言的do…while。如有兴趣可在课后自行扩展学习。



## break/continue

break[n]可以指定跳出几层循环；continue跳过本次循环，但不会跳出循环。

即break跳出，continue跳过。





# 位置参数和特殊变量

有很多特殊变量是被Shell自动赋值的，我们已经遇到了\$?和\$1。其他常用的位置参数和特殊变量在这里总结一下：

```
$0 			相当于C语言main函数的argv[0]
$1、$2...	这些称为位置参数（Positional Parameter），相当于C语言main函数的argv[1]、argv[2]...
$# 			相当于C语言main函数的argc - 1，注意这里的#后面不表示注释
$@ 			表示参数列表"$1" "$2" ...，例如可以用在for循环中的in后面。
$* 			表示参数列表"$1" "$2" ...，同上
$? 			上一条命令的Exit Status
$$ 			当前进程号
```

位置参数可以用shift命令左移。比如shift 3表示原来的\$4现在变成\$1，原来的$5现在变成$2等等，原来的\$1、\$2、\$3丢弃，**$0不移动**。不带参数的shift命令相当于shift 1。例如：





# 输入输出

## echo

显示文本行或变量，或者把字符串输入到文件。

echo [option] string

-e 解析转义字符

-n 不回车换行。默认情况echo回显的内容后面跟一个回车换行。

```Shell
echo "hello\n\n"
echo -e "hello\n\n"
echo "hello"
echo -n "hello"
```





## 管道

可以通过 | 把一个**命令的输出传递给另一个命令做输入**。

```Shell
cat myfile | more
ls -l | grep "myfile"
df -k | awk '{print $1}' | grep -v "文件系统"
df -k 查看磁盘空间，找到第一列，去除“文件系统”，并输出
```



## tee

tee命令把结果输出到标准输出，另一个副本输出到相应文件。

```Shell
df -k | awk '{print $1}' | grep -v "文件系统" | tee a.txt
```

tee -a a.txt表示追加操作。

```Shell
df -k | awk '{print $1}' | grep -v "文件系统" | tee -a a.txt
```



## 文件重定向

```Shell
cmd > file 				把标准输出重定向到新文件中
cmd >> file 			追加
cmd > file 2>&1 		标准出错也重定向到1所指向的file里
cmd >> file 2>&1
cmd < file1 > file2 	输入输出都定向到文件里
cmd < &fd 				把文件描述符fd作为标准输入
cmd > &fd 				把文件描述符fd作为标准输出
cmd < &- 				关闭标准输入
```





# 函数

和C语言类似，Shell中也有函数的概念，但是函数定义中没有返回值也没有参数列表（但是可以传入参数）。例如：

```Shell
#! /bin/sh

foo(){ echo "Function foo is called";}
echo "-=start=-"
foo
echo "-=end=-"
```

注意函数体的左花括号 { 和后面的命令之间必须有空格或换行，如果将最后一条命令和右花括号 } 写在同一行，命令末尾必须有分号;。但，不建议将函数定义写至一行上，不利于脚本阅读。

在定义foo()函数时并不执行函数体中的命令，就像定义变量一样，只是给foo这个名一个定义，到后面调用foo函数的时候（注意Shell中的**函数调用不写括号**）才执行函数体中的命令。Shell脚本中的函数必须**先定义后调用**，一般把函数定义语句写在脚本的前面，把函数调用和其它命令写在脚本的最后（类似C语言中的main函数，这才是整个脚本实际开始执行命令的地方）。

Shell函数没有参数列表并不表示不能传参数，事实上，函数就像是迷你脚本，调用函数时可以传任意个参数，在函数内同样是用\$0、\$1、​\$2等变量来提取参数，函数中的位置参数相当于函数的局部变量，改变这些变量并不会影响函数外面的\$0、​\$1、​\$2等变量。函数中可以用return命令返回，如果return后面跟一个数字则表示函数的Exit Status。

```Shell
#! /bin/sh

is_directory()
{
	DIR_NAME=$1
	if [ ! -d $DIR_NAME ]; then
		return 1
	else
		return 0 # 可以有返回值
	fi
}

for DIR in "$@"; do
	if is_directory "$DIR"
	then :
	else
		echo "$DIR doesn't exist. Creating it now..."
		mkdir $DIR > /dev/null 2>&1
		if [ $? -ne 0 ]; then
			echo "Cannot create directory $DIR"
			exit 1
		fi
	fi
done
```





# Shell脚本调试方法

Shell提供了一些用于调试脚本的选项，如：

- -n		读一遍脚本中的命令但不执行，用于检查脚本中的语法错误。

- -v		一边执行脚本，一边将执行过的脚本命令打印到标准错误输出。

- -x		提供跟踪执行信息，将执行的每一条命令和结果依次打印出来。

使用方法：

1. 在命令行提供参数。

```Shell
$ sh -x ./script.sh
```

2. 在脚本开头提供参数

```Shell
#! /bin/sh -x
```

3. 在脚本中使用set命令启用或者禁用参数

```Shell
#! /bin/sh

if [ -z "$1" ]; then
	set -x
	echo "ERROR: Insufficient Args."
	exit 1
	set +x
fi
```

set -x和set +x分别表示启用和禁用-x参数，这样可以只对脚本中的某一段进行跟踪调试。