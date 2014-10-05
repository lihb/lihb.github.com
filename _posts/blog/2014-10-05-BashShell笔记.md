---
layout: post
title: BaseShell笔记
description: shell编程的简单笔记
category: blog
---

## 1.普通变量

```bash
age= “34”
name= "david"
echo "my name is $name and I'm $age years old!"

```
输出结果为：

```
my name is david and I'm 34 years old! 
```
**注意**：变量名和”=“号之间不能有**空格**。

## 2.输入参数相关

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



  
