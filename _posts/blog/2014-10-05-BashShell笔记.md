---
layout: post
title: BaseShell笔记
description: shell编程的简单笔记
category: blog
---

简单记录本人学习shell编程的知识点。

#### 1.普通变量

```bash
age= "34"
name= "david"
echo "my name is $name and I'm $age years old!"

```
输出结果为：

```
my name is david and I'm 34 years old! 
```
**注意**：变量名和”=“号之间不能有**空格**。

#### 2.输入参数相关

* $@：表示所有参数
* $#：表示参数总的数目
* $0：表示当前文件名称
* $1~n：表示第几个参数

```
if [ $# == 0 ]
then
    echo "$0 paras error!!"
else
   echo "the first var is $1"
   for var in $@
     do
    ┊   echo "current var is $var"
    done
fi
```
输出结果为：

```
lhb@lhb-PC ~ $ ./ShellDemo.sh adc de seewr
the first var is adc
current var is adc
current var is de
current var is seewr

```
#### 3.if语句

在shell中,0表示真，1表示假。我们可以通过**echo $?**命令查看，上次命令的返回值是真或假。

```
lhb@lhb-PC /media/lhb/e/testDir $ ls
a.txt  b.txt  c.txt
lhb@lhb-PC /media/lhb/e/testDir $ echo $?
0
lhb@lhb-PC /media/lhb/e/testDir $ [ 5 -lt 6 ]  # 判断5<6
lhb@lhb-PC /media/lhb/e/testDir $ echo $?
0
lhb@lhb-PC /media/lhb/e/testDir $ [ 5 -gt 6 ]  # 判断5>6
lhb@lhb-PC /media/lhb/e/testDir $ echo $?
1

```

**if语句的语法格式：**

```
if condition1 ;then
    xxxx
elif condition2;then
    xxxx
else
    xxxx
fi

```

示例：

```
if [ $1 -gt $2 ];then
   echo $1
else
   echo $2
fi
```
结果：

```
lhb@lhb-PC ~ $ ./ShellDemo.sh 12 46
46
lhb@lhb-PC ~ $ ./ShellDemo.sh 34 23
34
```
#### 4.shell中的比较运算

##### 字符串的比较
 
比较字符串的运算符主要有：

* x=y ——用来检查x与y是否相等

* x!=y ——用来检查x和y是否不等

* -n x ——用来检查字符串x的长度是否大于0，x不是空字符串时为真

* -z x ——用来检查字符串x长度是否为0，x是空字符串时为真

ps:  如果两个字符串长度不同时，系统会先为较短的字符串后添加空格使得两者长度相同，而后完成比较。

##### 数字的比较

* x -eq y ——检查是否x和y相等

* x -ne y ——检查是否x和y不等

* x -gt y ——检查是否x大于y

* x -lt y ——检查是否x小于y

* x -ge y ——检查是否x大于等于y

* x -le y ——检查是否x小于等于y

##### 针对文件的运算符

* -e file ——文件file已经存在

* -f file ——文件file是普通文件

* -s file ——文件file大小不为0

* -d file ——文件file是一个目录

* -r file ——文件file对当前用户可以读取

* -w file ——文件file对当前用户可以写入

* -x file ——文件file对当前用户可以执行

* -g file ——文件file的GID标志被设置

* -u file ——文件file的UID标志被设置

* -O file ——文件file是属于当前用户的

* -G file ——文件file的组ID和当前用户相同

* file1 -nt file2 ——文件file1比file2更新

* file1 -ot file2 ——文件file1比file2更老

##### 逻辑运算符

* ! 逻辑取非运算符，对一个逻辑表达式操作

* -a 逻辑与运算符，对两个逻辑表达式操作

* -o 逻辑或运算符，对两个逻辑表达式操作

#### 5.shell中将命令执行的结果作为输入

语法：** \`命令\` **

示例:

```
for file in `ls`
do
if [ -x $file ];then
    echo "$file is a execution file."
elif [ -f $file ];then
    echo "$file is a normal file."
else
    echo "other..."
fi
done

```
结果：

```
lhb@lhb-PC ~/testDir $ ls
a.txt  b.txt  c.txt  ShellDemo.sh
lhb@lhb-PC ~/testDir $ ./ShellDemo.sh
a.txt is a normal file.
b.txt is a normal file.
c.txt is a normal file.
ShellDemo.sh is a execution file.

```

#### read读取控制台数据给参数赋值

示例

```
lhb@lhb-PC ~ $ read pi
3.1415926  #这是从控制台输入的。
lhb@lhb-PC ~ $ echo "the pi is $pi"
the pi is 3.1415926
```

#### 其他语法知识点

* while 循环

* util语句

* switch-case语句

* break-continue

* 函数 



