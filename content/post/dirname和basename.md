+++
title = "dirname和basename"
date = 2021-07-10T19:42:58+08:00
description = ""
tags = ['linux', 'shell']
aplayer = false
showToc = true
math = true
showLicense = false
+++

从路径获取目录部分或文件名部分可以分别使用`dirname`和`basename`命令。

<!--more-->

## dirname
`dirname` 命令主要作用是去除路径中的非目录部分，删除最后一个`/`后面的路径，显示父目录。

如果所给路径中没有包含`/`，会输出`.`表示当前目录。

所给的路径名参数不要求是真实存在的路径，`dirname` 只是对所给的路径名字符串进行处理。

选项：
- `-z, --zero` 不换行打印

示例：
```bash
# 去除最后一个 `/` 的路径
$ dirname /usr/local/src/
/usr/local

$ dirname dir1/str dir2/str
dir1
dir2

# 不换行
$ dirname -z dir1/str dir2/str
dir1dir2

# 路径中没有 `/` ，返回 .
$ dirname stdio.h
.
```

## basename
`basename`命令用于打印目录或者文件的基本名称，显示**最后的**目录名或文件名。如果指定后缀，则可以删除相应的文件后缀。

选项：
- `-a, --multiple`
指定多个路径或文件
- `-s, --suffix=SUFFIX`
指定用于删除的后缀
- `-z, --zero`
不换行打印

示例：
```bash
# 打印最后的目录名
$ basename /usr/bin/sort
sort
$ basename /usr/bin/sort/
sort

$ basename include/stdio.h
stdio.h
$ basename stdio.h
stdio.h

# 两种方式去掉后缀
$ basename include/stdio.h .h
stdio
$ basename -s .h include/stdio.h
stdio

# 指定多个路径或文件
$ basename -a any/str1 any/str2
str1
str2

# 不换行打印
$ basename -az any/str1 any/str2
str1str2

# 混合使用，只能去一个后缀
$ basename -a -s .h any/str1.js any/str2.h
str1.js
str2
```

## 其它
以上操作其实可以使用bash的字符串截取操作。详见 Linux Shell Bash 的字符串操作。
