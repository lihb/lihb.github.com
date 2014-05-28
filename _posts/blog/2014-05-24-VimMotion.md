---
layout: post
title: VimMotion
description: 实用的VimMotion。
category: blog
---

主要介绍vim编辑器中`motion`个人认为比较高级，且在编写代码中会用到的一些用法,一般都是和操作命令如（c,d等）配合使用。

words
======
**aw**: 一个单词，包含单词两端的空格

**iw**: 单词内部，不包含两端的空格

示例：

```vim
This is a sh*ort sentence!
```
    ps: 光标位置在'*'后的第一个字符上。

在光标处输入命令：`daw`

```vim
This is a sentence!
```
sentences
=========
**as**：一条语句

**is**：语句内部

```
Text objects beginning with a include the surrounding white space in the text object.those starting with i do not. This convention is fo*llowed by all text objects.
```

在光标处输入命令：`cis`

```
Text objects beginning with a include the surrounding white space in the text object.those starting with i do not. 
```

结果为：删除光标所在行，并自动进入`插入模式`。

Paragraphs
==========

**ap**：一个段落

**ip**：段落内部

```
Lorem ips*um dolor sit amet, consectetur adipisicing elit, sed do
eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis 
nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.

Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla
pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt
mollit anim id est laborum.

```

在光标处输入命令：`dap`---删除一整个段落

```
Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla
pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt
mollit anim id est laborum.

```
其他一些特性
============

**di[", ', ], ), }]**-----分别删除" ", ' ', [ ], ( ), { }的内容

**da[", ', ], ), }]**-----分别删除" ", ' ', [ ], ( ), { }的内容,带括号符号一起删除。

**ci[", ', ], ), }]**-----分别删除" ", ' ', [ ], ( ), { }的内容,进入`插入模式`，光标在符号内部。

**ca[", ', ], ), }]**-----分别删除" ", ' ', [ ], ( ), { }的内容,带括号符号一起删除，进入`插入模式`。

    ps:此时光标须在符号之间：(*sdfdrg)或者(hhkk*hzsf)或者(sfdstr*)都行。

示例:

<table>
    <tr>
        <td>源字符</td>
        <td>命令</td>
        <td>处理后的字符</td>
        <td>备注</td>
    </tr>
    <tr>
        <td>bsfesr(dfest)tert</td>
        <td>di)或di(</td>
        <td> bsfesr(*)tert</td>
        <td>   </td>
    </tr>
</table>

```

源字符               命令          处理后的字符        备注

bsfesr(dfest)tert   di)或di(      bsfesr(*)tert

hello(lhb)world     da(或da)      hello*world


bsfesr(dfest)tert   ci)或ci(      bsfesr(*)tert     进入插入模式

hello(lhb)world     ca(或ca)      hello*world       进入插入模式

```

html标签
-------

同上类似，括号符号用`t`(tag)代替

示例：

```
           源字符                命令          处理后           备注
    <h2>Sample Title</h2>        dit         <h2>*</h2>    只删除标签里的内容 
    <h2>Sample Title</h2>        dat           *              整个删除

    <h2>Sample Title</h2>        cit         <h2>*</h2>      进入插入模式
    <h2>Sample Title</h2>        cat            *            进入插入模式

```






