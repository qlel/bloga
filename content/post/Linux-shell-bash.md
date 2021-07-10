+++
title = "Linux Shell Bash"
description = ""
tags = ['linux', 'shell', 'bash']
date =  "2021-04-08T17:12:50+08:00"
lastmod = "2021-04-08T17:12:50+08:00"
+++
shell bash 脚本相关
<!--more-->

## bash简介
Bash 是 Unix 系统和 Linux 系统的一种 Shell（命令行环境），是目前绝大多数 Linux 发行版的默认 Shell。

进入命令行环境以后，一般就已经打开 Bash 了。如果你的 Shell 不是 Bash，可以输入bash命令启动 Bash。
```bash
# 进入
$ bash

# 退出
$ exit

# 版本
$ bash --version
GNU bash, version 5.0.3(1)-release (x86_64-pc-linux-gnu)
Copyright (C) 2019 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>

This is free software; you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

# 或者
$ echo $BASH_VERSION
5.0.3(1)-release
```
## bash基本语法
主要介绍语法和一些常用命令。

命令语法：
```bash
command [ arg1 ... [ argN ]]

# 换行
# 命令可以用反斜杠 \ 换行
command arg1 \
arg2 arg3 \
arg4...

# 空格
# 数之间有多个空格，Bash 会自动忽略多余的空格
$ echo this is a     test
this is a test

# 分号(;)，结束符
# 使得一行可以放置多个命令
$ clear; ls
```
### `&&`和`||`
命令的组合符`&&`和`||`:
```bash
# 如果Command1命令运行成功，则继续运行Command2命令。
Command1 && Command2

# 如果Command1命令运行失败，则继续运行Command2命令。
Command1 || Command2
```
### 快捷键
- `Ctrl + L`：清除屏幕并将当前行移到页面顶部。
- `Ctrl + C`：中止当前正在执行的命令。
- `Shift + PageUp`：向上滚动。
- `Shift + PageDown`：向下滚动。
- `Ctrl + U`：从光标位置删除到行首。
- `Ctrl + K`：从光标位置删除到行尾。
- `Ctrl + D`：关闭 Shell 会话。
- `↑`，`↓`：浏览已执行命令的历史记录。

### type
Bash 本身内置了很多命令，同时也可以执行外部程序。怎么知道一个命令是内置命令，还是外部程序呢？

type命令用来判断命令的来源。
```bash
# 内建程序
$ type echo
echo is a shell builtin

# # 外部程序
$ type ls
ls is hashed (/bin/ls)
```
`-a`参数查看一个命令的所有定义：
```bash
# echo命令即是内置命令，也有对应的外部程序。
$ type -a echo
echo is shell builtin
echo is /usr/bin/echo
echo is /bin/echo
```
`-t`参数返回一个命令的类型：别名（alias），关键词（keyword），函数（function），内置命令（builtin）和文件（file）。
```bash
$ type -t bash
file
$ type -t if
keyword
```

### echo
echo命令的作用是在屏幕输出一行文本。
```bash
$ echo hello world
hello world
```
如果想要输出的是多行文本，即包括换行符。这时需要把多行文本放在引号里面。
```bash
$ echo "<HTML>
    <HEAD>
          <TITLE>Page Title</TITLE>
    </HEAD>
    <BODY>
          Page body.
    </BODY>
</HTML>"
```
参数：

`-n`：
输出不换行

`-e`：
启用反斜杠转义的解释。如果不使用`-e`参数，引号会让特殊字符变成普通字符，echo不解释它们，原样输出。例如：
```bash
$ echo "Hello\nWorld"
Hello\nWorld

# 双引号的情况
$ echo -e "Hello\nWorld"
Hello
World

# 单引号的情况
$ echo -e 'Hello\nWorld'
Hello
World
```

## 模式扩展
模式扩展类似正则表达式，但早于正则表达式出现。它的功能没有正则那么强大灵活，但是优点是简单和方便。

Bash 一共提供八种扩展：
- 波浪线扩展
- `?` 字符扩展
- `*` 字符扩展
- 方括号扩展
- 大括号扩展
- 变量扩展
- 子命令扩展
- 算术扩展

Bash 允许用户关闭模式扩展：
```bash
$ set -o noglob
# 或者
$ set -f
```
重新打开模式扩展：
```bash
$ set +o noglob
# 或者
$ set +f
```
### 波浪线扩展
波浪线`~`会自动扩展成当前用户的主目录。
```bash
$ echo ~
/home/me
```
`~user`表示扩展成用户`user`的主目录:
```bash
$ echo ~foo
/home/foo

$ echo ~root
/root
```
`~+`会扩展成当前所在的目录，等同于`pwd`命令:
```bash
$ cd ~/foo
$ echo ~+
/home/me/foo
```
### `?` 字符扩展 
`?`字符代表文件路径里面的任意**单个**字符，不包括空字符。比如，`Data???`匹配所有`Data`后面跟着三个字符的文件名。
```bash
# 存在文件 a.txt 和 b.txt
$ ls ?.txt
a.txt b.txt
```
`?` 字符扩展属于文件名扩展，只有文件确实存在的前提下，才会发生扩展。如果文件不存在，扩展就不会发生。
```bash
# 当前目录有 a.txt 文件
$ echo ?.txt
a.txt

# 当前目录为空目录
$ echo ?.txt
?.txt
```
### `*` 字符扩展
`*`字符代表文件路径里面的任意数量的任意字符，包括零个字符。
```bash
# 存在文件 a.txt、b.txt 和 ab.txt
$ ls *.txt
a.txt b.txt ab.txt
```
注意，`*`不会匹配隐藏文件（以`.`开头的文件），即`ls *`不会输出隐藏文件。

如果要匹配隐藏文件，需要写成`.*`。
```bash
# 显示所有隐藏文件
$ echo .*
```
如果要匹配隐藏文件，同时要排除`.`和`..`这两个特殊的隐藏文件，可以与方括号扩展结合使用，写成`.[!.]*`。
```bash
$ echo .[!.]*
```
注意，`*`字符扩展属于文件名扩展，只有文件确实存在的前提下才会扩展。如果文件不存在，就会原样输出。
```bash
# 当前目录不存在 c 开头的文件
$ echo c*.txt
c*.txt
```
### 方括号扩展
方括号扩展的形式是`[...]`，只有文件确实存在的前提下才会扩展。如果文件不存在，就会原样输出。方括号里面各值类似逻辑或关系。

括号之中的任意一个字符，比如，`[aeiou]`可以匹配五个元音字母中的任意一个。
```bash
# 存在文件 a.txt 和 b.txt
$ ls [ab].txt
a.txt b.txt

# 只存在文件 a.txt
$ ls [ab].txt
a.txt
```
方括号扩展还有两种变体：`[^...]`和`[!...]`。它们表示匹配不在方括号里面的字符，这两种写法是等价的。比如，`[^abc]`或`[!abc]`表示匹配除了`a、b、c`以外的字符。
```bash
# 存在 aaa、bbb、aba 三个文件
$ ls ?[!a]?
aba bbb
```
>注意，如果需要匹配`[`字符，可以放在方括号内，比如`[[aeiou]`。如果需要匹配连字号`-`，只能放在方括号内部的开头或结尾，比如`[-aeiou]`或`[aeiou-]`。

方括号扩展有一个简写形式`[start-end]`，表示匹配一个连续的范围。比如，`[a-c]`等同于`[abc]`，`[0-9]`匹配`[0123456789]`。
```bash
# 存在文件 a.txt、b.txt 和 c.txt
$ ls [a-c].txt
a.txt
b.txt
c.txt

# 存在文件 report1.txt、report2.txt 和 report3.txt
$ ls report[0-9].txt
report1.txt
report2.txt
report3.txt
...
```
下面是一些常用简写的例子:
- `[a-z]`：所有小写字母。
- `[a-zA-Z]`：所有小写字母与大写字母。
- `[a-zA-Z0-9]`：所有小写字母、大写字母与数字。
- `[abc]*`：所有以`a、b、c`字符之一开头的文件名。
- `program.[co]`：文件`program.c`与文件`program.o`。
- `BACKUP.[0-9][0-9][0-9]`：所有以`BACKUP.`开头，后面是三个数字的文件名。

### 大括号扩展
大括号扩展`{...}`表示分别扩展成大括号里面的所有值，各个值之间使用逗号分隔。比如，`{1,2,3}`扩展成`1 2 3`。大括号里面各值类似逻辑与关系。
```bash
$ echo {1,2,3}
1 2 3

$ echo d{a,e,i,u,o}g
dag deg dig dug dog

$ echo Front-{A,B,C}-Back
Front-A-Back Front-B-Back Front-C-Back
```
>注意，大括号扩展不是文件名扩展。它会扩展成所有给定的值，而不管是否有对应的文件存在。

```bash
$ ls {a,b,c}.txt
ls: 无法访问'a.txt': 没有那个文件或目录
ls: 无法访问'b.txt': 没有那个文件或目录
ls: 无法访问'c.txt': 没有那个文件或目录
```
>另一个需要注意的地方是，大括号内部的逗号前后不能有空格。否则，大括号扩展会失效。

```bash
$ echo {1 , 2}
{1 , 2}
```
号前面可以没有值，表示扩展的第一项为空。
```bash
$ cp a.log{,.bak}

# 等同于
# cp a.log a.log.bak
```
大括号可以嵌套。
```bash
$ echo {j{p,pe}g,png}
jpg jpeg png

$ echo a{A{1,2},B{3,4}}b
aA1b aA2b aB3b aB4b
```
大括号也可以与其他模式联用，并且总是先于其他模式进行扩展。
```bash
$ echo /bin/{cat,b*}
/bin/cat /bin/b2sum /bin/base32 /bin/base64 ... ...

# 基本等同于
$ echo /bin/cat;echo /bin/b*
```
由于大括号扩展`{...}`不是文件名扩展，所以它总是会扩展的。这与方括号扩展`[...]`完全不同，如果匹配的文件不存在，方括号就不会扩展。这一点要注意区分。
```bash
# 不存在 a.txt 和 b.txt
$ echo [ab].txt
[ab].txt

$ echo {a,b}.txt
a.txt b.txt
```
大括号扩展有一个简写形式`{start..end}`，表示扩展成一个连续序列。比如，`{a..z}`可以扩展成26个小写英文字母。
```bash
$ echo {a..c}
a b c

$ echo d{a..d}g
dag dbg dcg ddg

$ echo {1..4}
1 2 3 4

$ echo Number_{1..5}
Number_1 Number_2 Number_3 Number_4 Number_5
```
这种简写形式支持逆序。
```bash
$ echo {c..a}
c b a

$ echo {5..1}
5 4 3 2 1
```
注意，如果遇到无法理解的简写，大括号模式就会原样输出，不会扩展。
```bash
$ echo {a1..3c}
{a1..3c}
```
支持`for...in`循环：
```bash
for i in {1..4}
do
  echo $i
done
```
如果整数前面有前导`0`，扩展输出的每一项都有前导`0`。
```bash
$ echo {01..5}
01 02 03 04 05

$ echo {001..5}
001 002 003 004 005
```
这种简写形式还可以使用第二个双点号`{start..end..step}`，用来指定扩展的步长。
```bash
$ echo {0..8..2}
0 2 4 6 8
```
多个简写形式连用，会有循环处理的效果。
```bash
$ echo {a..c}{1..3}
a1 a2 a3 b1 b2 b3 c1 c2 c3
```
### 变量扩展
Bash 将美元符号`$`开头的词元视为变量，将其扩展成变量值。
```bash
$ echo $SHELL
/bin/bash
```
变量名除了放在美元符号后面，也可以放在`${}`里面:
```bash
$ echo ${SHELL}
/bin/bash
```
`${!string*}`或`${!string@}`返回所有匹配给定字符串`string`的变量名。
```bash
$ echo ${!SH*}
SHELL SHELLOPTS SHLVL
```
### 子命令扩展
`$(...)`可以扩展成另一个命令的运行结果，该命令的所有输出都会作为返回值。
```bash
$ echo $(date)
Tue Jan 28 00:01:13 CST 2020
```
还有另一种较老的语法，子命令放在反引号之中，也可以扩展成命令的运行结果。
```bash
$ echo `date`
Tue Jan 28 00:01:13 CST 2020
```
`$(...)`可以嵌套，比如`$(ls $(pwd))`。

### 算术扩展
`$((...))`可以扩展成整数运算的结果。
```bash
$ echo $((2 + 2))
4
```
### 量词语法
量词语法用来控制模式匹配的次数。它只有在 Bash 的`extglob`参数打开的情况下才能使用，不过一般是默认打开的。

下面的命令可以查询:
```bash
$ shopt extglob
extglob        	on
```
如果`extglob`参数是关闭的，可以用下面的命令打开:
```bash
$ shopt -s extglob
```
量词语法有下面几个:
- `?(pattern-list)`：匹配零个或一个模式。
- `*(pattern-list)`：匹配零个或多个模式。
- `+(pattern-list)`：匹配一个或多个模式。
- `@(pattern-list)`：只匹配一个模式。
- `!(pattern-list)`：匹配给定模式以外的任何内容。

```bash
$ ls abc?(.)txt
abctxt abc.txt

# 匹配零个或一个def
$ ls abc?(def)
abc abcdef

$ ls abc+(.txt)
abc.txt abc.txt.txt
```
量词语法也属于文件名扩展，如果不存在可匹配的文件，就会原样输出:
```bash
# 没有 abc 开头的文件名
$ ls abc?(def)
ls: 无法访问'abc?(def)': 没有那个文件或目录
```

## 引号与转义
Bash 只有一种数据类型，就是**字符串**。不管用户输入什么数据，Bash 都视为字符串。因此，字符串相关的引号和转义，对 Bash 来说就非常重要。

### 转义
转义分为 特殊字符转为普通字符 和 普通字符转为特殊字符。
```dot
digraph G{
    rankdir="LR"
    node[shape=box,fontname="微软雅黑"]
    edge[arrowhead=none,fontname="微软雅黑"]

    a00[label="特殊字符" color="red"]
    a01[label="普通字符" color="blue"]

    a00->a01[label="转义\\" arrowhead="lvee" color="red"]
    a01->a00[arrowhead="lvee" color="blue"]
}
```
特殊字符转为普通字符：如`$`、`&`、`*`、`\`等特殊字符转为普通字符：`\$`、`\&`、`\*`、`\\`。

普通字符转为特殊字符：
- `\a`：响铃
- `\b`：退格
- `\n`：换行
- `\r`：回车
- `\t`：制表符

在bash中，换行符是一个特殊字符，表示命令的结束，Bash 收到这个字符以后，就会对输入的命令进行解释执行。换行符前面加上反斜杠转义，就使得换行符变成一个普通字符，Bash 会将其当作空格处理，从而可以将一行命令写成多行。
```bash
$ mv \
/path/to/foo \
/path/to/bar

# 等同于
$ mv /path/to/foo /path/to/bar
```
### 单引号与双引号
Bash 允许字符串放在单引号或双引号之中，加以引用。

单引号用于**保留字符的字面含义**，各种特殊字符在单引号里面，都会变为普通字符，比如星号（`*`）、美元符号（`$`）、反斜杠（`\`）等。
```bash
$ echo '*'
*

$ echo '$USER'
$USER

$ echo '$((2+2))'
$((2+2))

$ echo '$(echo foo)'
$(echo foo)
```
由于反斜杠在单引号里面变成了普通字符，所以如果单引号之中，还要使用单引号，不能使用转义，需要在外层的单引号前面加上一个美元符号（`$`），然后再对里层的单引号转义。
```bash
# 不正确
$ echo it's

# 不正确
$ echo 'it\'s'

# 正确
$ echo $'it\'s'
```
更合理的方法是改在双引号之中使用单引号。
```bash
$ echo "it's"
it's
```
双引号与单引号的不同在于，双引号会解析三个特殊字符：美元符号（$）、反引号（`）和反斜杠（\）。

```bash
$ echo "$SHELL"
/bin/bash

$ echo "`date`"
Mon Jan 27 13:33:18 CST 2020

$ echo "\\"
\
```
双引号与单引号都可以使换行符不在解析为结束符，可以在命令行输入多行文本。
```bash
$ echo 'hello
> woooo'
hello
woooo

$ echo "hello
> world"
hello
world
```
双引号还有一个作用，就是保存原始命令的输出格式：
```bash
# 原始输出
$ cal
     March 2021
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30 31

# 单行输出
$ echo $(cal)
March 2021 Su Mo Tu We Th Fr Sa 1 ... 30 31

# 原始格式输出
$ echo "$(cal)"
     March 2021
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30 31
```
### Here 文档
Here 文档（here document）是一种输入多行字符串的方法，格式如下：
```bash
<< token
text
token
```
组成部分：
1. 开始标记：`<< 标记`。`标记`可以随意取名，后面必须换行。
2. 内容
3. 结束标记：`标记`。必须在单独一行。

Here 文档内部会发生变量替换，同时支持反斜杠转义，但是不支持通配符扩展，双引号和单引号也失去语法作用，变成了普通字符。
```bash
$ foo='hello world'
$ cat << _example_
$foo
"$foo"
'$foo'
_example_

hello world
"hello world"
'hello world'
```
如果不希望发生变量替换，可以把 Here 文档的开始标记放在单引号之中。
```bash
$ foo='hello world'
$ cat << '_example_'
$foo
"$foo"
'$foo'
_example_

$foo
"$foo"
'$foo'
```
Here 文档的本质是**重定向**，它将字符串重定向输出给某个命令，相当于包含了`echo`命令。
```bash
$ command << token
  string
token

# 等同于

$ echo string | command
```
Here 文档还有一个变体，叫做 Here 字符串（Here string），使用三个小于号（`<<<`）表示。作用是将字符串通过标准输入，传递给命令。
```bash
$ cat <<< 'hi there'
# 等同于
$ echo 'hi there' | cat
```

## 变量
Bash 变量分成**环境变量**和**自定义变量**两类。

### 环境变量
环境变量是 Bash 环境自带的变量，进入 Shell 时已经定义好了，可以直接使用。它们通常是系统定义好的，也可以由用户从父 Shell 传入子 Shell。

显示所有环境变量：
```bash
$ env
# 或者
$ printenv

# 显示所有变量（包括环境变量和自定义变量），以及所有的 Bash 函数
$ set
```
查看单个环境变量的值
```bash
$ printenv PATH
# 或者
$ echo $PATH
```

常见的环境变量:
环境变量|说明
-|-
`BASHPID`|Bash 进程的进程 ID。
`BASHOPTS`|当前 Shell 的参数，可以用shopt命令修改。
`DISPLAY`|图形环境的显示器名字，通常是:0，表示 X Server 的第一个显示器。
`EDITOR`|默认的文本编辑器。
`HOME`|用户的主目录。
`HOST`|当前主机的名称。
`IFS`|词与词之间的分隔符，默认为空格。
`LANG`|字符集以及语言编码，比如zh_CN.UTF-8。
`PATH`|由冒号分开的目录列表，当输入可执行程序名后，会搜索这个目录列表。
`PS1`|Shell 提示符。
`PS2`| 输入多行命令时，次要的 Shell 提示符。
`PWD`|当前工作目录。
`RANDOM`|返回一个0到32767之间的随机数。
`SHELL`|Shell 的名字。
`SHELLOPTS`|启动当前 Shell 的set命令的参数
`TERM`|终端类型名，即终端仿真器所用的协议。
`UID`|当前用户的 ID 编号。
`USER`|当前用户的用户名。

很多环境变量很少发生变化，而且是只读的，可以视为常量。

>注意：Bash 变量名区分大小写

### 定义变量
用户创建变量的时候，变量名必须遵守下面的规则：
- 字母、数字和下划线字符组成。
- 第一个字符必须是一个字母或一个下划线，不能是数字。
- 不允许出现空格和标点符号。

变量声明：
```bash
variable=value
```
>注意，等号两边不能有空格。

如果变量的值包含空格，则必须将值放在引号中:
```bash
myvar="hello world"
```
Bash 没有数据类型的概念，所有的变量值都是字符串。

变量可以重复赋值，后面的赋值会覆盖前面的赋值:
```bash
$ foo=1
$ foo=2
$ echo $foo
2
```
如果同一行定义多个变量，必须使用分号（`;`）分隔:
```bash
$ foo=1;bar=2
```
### 读取变量
读取变量的时候，直接在变量名前加上`$`就可以了:
```bash
$ foo=bar
$ echo $foo
bar
```
如果变量不存在，Bash 不会报错，而会输出空字符。

读取变量的时候，变量名也可以使用花括号`{}`包围，比如`$a`也可以写成`${a}`。这种写法可以用于变量名与其他字符连用的情况。
```bash
$ a=foo
$ echo $a_file

$ echo ${a}_file
foo_file
```
如果变量的值本身也是变量，可以使用`${!varname}`的语法，读取最终的值:
```bash
$ myvar=USER
$ echo ${!myvar}
qlel
```
如果变量值包含连续空格（或制表符和换行符），最好放在双引号里面读取:
```bash
$ a="1 2  3"
$ echo $a
1 2 3
$ echo "$a"
1 2  3
```
### 删除变量
`unset`命令用来删除一个变量:
```bash
unset NAME
```
这个命令不是很有用。因为不存在的 Bash 变量一律等于空字符串，所以即使`unset`命令删除了变量，还是可以读取这个变量，值为空字符串。

所以，删除一个变量，也可以将这个变量设成空字符串。

### export 命令
用户创建的变量仅可用于当前 Shell，子 Shell 默认读取不到父 Shell 定义的变量。为了把变量传递给子 Shell，需要使用`export`命令。这样输出的变量，对于子 Shell 来说就是环境变量。

`export`命令用来向子 Shell 输出变量。
```bash
NAME=foo

# 输出
export NAME

# 赋值和输出一起
export NAME=value
```
子 Shell 如果修改继承的变量，不会影响父 Shell：
```bash
# 输出变量 $foo
$ export foo=bar

# 新建子 Shell
$ bash

# 读取 $foo
$ echo $foo
bar

# 修改继承的变量
$ foo=baz

# 退出子 Shell
$ exit

# 读取 $foo
$ echo $foo
bar
```
### 特殊变量
Bash 提供一些特殊变量。这些变量的值由 Shell 提供，用户不能进行赋值。

> 01 `$?`

`$?`为上一个命令的退出码，用来判断上一个命令是否执行成功。
- 返回值是0，表示上一个命令执行成功
- 如果是非零，上一个命令执行失败。

```bash
$ ls abc
ls: cannot access 'abc': No such file or directory

$ echo $?
2
```
> 02 `$$`

`$$`为当前 Shell 的进程 ID。
```bash
$ echo $$
671
```
> 03 `$_`

`$_`为上一个命令的最后一个参数。
```bash
$ ls -a
$ $ echo $_
-a

$ find ~/test -name 'read*'
/home/qlel/test/read_0.sh

$ echo $_
read_0.sh
```
> 04 `$!`

`$!`为最近一个后台执行的异步命令的进程 ID。
```bash
$ firefox &
[1] 11064

$ echo $!
11064
```

> 05 `$0`

`$0`为当前 Shell 的名称（在命令行直接执行时）或者脚本名（在脚本中执行时）。
```bash
$ echo $0
bash
```
> 06 `$-`

`$-`为当前 Shell 的启动参数。
```bash
$ echo $-
himBHs
```
> 07 `$@`和`$#`

`$@`和`$#`表示脚本的参数数量。

### 变量的默认值
Bash 提供四个特殊语法，跟变量的默认值有关，目的是保证变量不为空。
- `${varname:-word}`
    - 如果变量`varname`存在且不为空，则返回它的值，否则返回`word`。
    - 它的目的是返回一个默认值，不设置`varname`变量的值。
- `${varname:=word}`
    - 如果变量`varname`存在且不为空，则返回它的值，否则将它设为`word`，并且返回`word`。
    - 它的目的是设置变量的默认值，并将默认值返回。
- `${varname:+word}`
    - 如果变量`varname`存在且不为空，则返回`word`，否则返回空值。
    - 它的目的是测试变量是否存在
- `${varname:?message}`
    - 如果变量`varname`存在且不为空，则返回它的值，否则打印出`varname: message`，并中断脚本的执行。
    - 如果省略了`message`，则输出默认的信息“parameter null or not set.”。
    - 它的目的是防止变量未定义

上面四种语法如果用在脚本中，变量名的部分可以用数字1到9，表示脚本的参数。
```bash
filename=${1:?"filename missing."}
```
`1`表示脚本的第一个参数。如果该参数不存在，就退出脚本并报错。

### declare 命令
`declare`命令可以声明一些特殊类型的变量，为变量设置一些限制，比如声明只读类型的变量和整数类型的变量。

语法：
```bash
declare OPTION VARIABLE=value
```
选项：
选项|说明
-|-
`-a`|声明数组变量。
`-f`|输出所有函数定义。
`-F`|输出所有函数名。
`-i`|声明整数变量。
`-l`|声明变量为小写字母。
`-p`|查看变量信息。
`-r`|声明只读变量。
`-u`|声明变量为大写字母。
`-x`|该变量输出为环境变量。

> 注意，`declare`命令如果用在函数中，声明的变量只在函数内部有效，等同于`local`命令。

(1) `-i`
`-i`参数声明整数变量以后，可以直接进行数学运算。
```bash
$ declare -i val1=12 val2=5
$ declare -i result
$ result=val1*val2
$ echo $result
60
```
注意，一个变量声明为整数以后，依然可以被改写为字符串。
```bash
$ declare -i var=12
$ var=foo
$ echo $var
0
```
(2) `-x`
`-x`参数等同于`export`命令，可以输出一个变量为子 Shell 的环境变量。
```bash
$ declare -x foo
# 等同于
$ export foo
```
(3) `-r`
`-r`参数可以声明只读变量，无法改变变量值，也不能`unset`变量。
```bash
$ declare -r bar=1

$ bar=2
bash: bar：只读变量
$ echo $?
1

$ unset bar
bash: bar：只读变量
$ echo $?
1
```
### readonly 命令
`readonly`命令等同于`declare -r`，用来声明只读变量，不能改变变量值，也不能`unset`变量。
```bash
$ readonly foo=1
$ foo=2
bash: foo：只读变量
$ echo $?
1
```
选项|说明
-|-
`-f`|声明的变量为函数名。
`-p`|打印出所有的只读变量。
`-a`|声明的变量为数组。

### let 命令
`let`命令声明变量时，可以直接执行算术表达式。
```bash
$ let foo=1+2
$ echo $foo
3
```
let命令的参数表达式如果包含空格，就需要使用引号:
```bash
$ let "foo = 1 + 2"
```
可以同时对多个变量赋值，赋值表达式之间使用空格分隔:
```bash
$ let "v1 = 1" "v2 = v1++"
$ echo $v1,$v2
2,1
```

## 字符串操作
### 字符串长度
获取字符串长度的语法：
```bash
${#varname}

# 示例
$ myPath=/home/cam/book/long.file.name
$ echo ${#myPath}
29
```
### 子字符串
字符串提取子串的语法：
```bash
${varname:offset:length}
```
从`offset`(从`0`开始计算)位置开始，返回长度为`length`的变量`varname`的子串，原变量`varname`不变。
```bash
$ aa=helloworld

$ echo ${aa:2:5}
llowo

# 省略length
$ echo ${aa:3}
loworld

$ echo $aa
helloworld
```
如果`offset`为负值，表示从字符串的末尾开始算起。注意，负数前面必须有一个空格， 以防止与`${variable:-word}`的变量的设置默认值语法混淆。这时还可以指定`length`，`length`可以是正值，也可以是负值（负值不能超过`offset`的长度）。
```bash
$ aa=helloworld

$ echo ${aa: -3}
rld
$ echo ${aa: -3:2}
rl
$ echo ${aa: -3:-2}
r
$ echo ${aa: -3:-3}

$ echo ${aa: -3:-5}
-bash: -5: substring expression < 0
```
### 搜索和替换
(1) 字符串头部的模式匹配 

从字符串头部开始匹配，如果匹配成功，就删除匹配的部分，返回剩下的部分。如果匹配不成功，则返回原始字符串。原始变量不会发生变化。
```bash
# 如果 pattern 匹配变量 variable 的开头，
# 删除最短匹配（非贪婪匹配）的部分，返回剩余部分
${variable#pattern}

# 如果 pattern 匹配变量 variable 的开头，
# 删除最长匹配（贪婪匹配）的部分，返回剩余部分
${variable##pattern}
```
匹配模式`pattern`可以使用`*`、`?`、`[]`等通配符。
```bash
$ phone="555-456-1414"
$ echo ${phone#*-}
456-1414
$ echo ${phone##*-}
1414
```
替换匹配内容：
```bash
${variable/#pattern/string}

# 示例
$ foo=JPG.JPG
$ echo ${foo/#JPG/jpg}
jpg.JPG

# 只有贪婪匹配替换
$ phone="555-456-1414"
$ echo ${phone/#*-/2222}
22221414
```

(2) 字符串尾部的模式匹配 

从字符串尾部开始匹配，如果匹配成功，就删除匹配的部分，返回剩下的部分。如果匹配不成功，则返回原始字符串。原始变量不会发生变化。
```bash
# 如果 pattern 匹配变量 variable 的结尾，
# 删除最短匹配（非贪婪匹配）的部分，返回剩余部分
${variable%pattern}

# 如果 pattern 匹配变量 variable 的结尾，
# 删除最长匹配（贪婪匹配）的部分，返回剩余部分
${variable%%pattern}
```
示例：
```bash
$ phone="555-456-1414"

$ echo ${phone%-*}
555-456
$ echo ${phone%%-*}
555
```
替换匹配内容：
```bash
${variable/%pattern/string}

# 示例
$ foo=JPG.JPG
$ echo ${foo/%JPG/jpg}
JPG.jpg

# 只有贪婪匹配替换
$ phone="555-456-1414"
$ echo ${phone/%-*/2222}
5552222
```

(3) 任意位置的模式匹配 

从字符串任意位置开始匹配，如果匹配成功，就删除匹配的部分，返回剩下的部分。如果匹配不成功，则返回原始字符串。原始变量不会发生变化。
```bash
# 如果 pattern 匹配变量 variable 的一部分，
# 最长匹配（贪婪匹配）的那部分被 string 替换，但仅替换第一个匹配
${variable/pattern/string}

# 如果 pattern 匹配变量 variable 的一部分，
# 最长匹配（贪婪匹配）的那部分被 string 替换，所有匹配都替换
${variable//pattern/string}
```
示例：
```bash
$ phone="555-456-1414"
$ echo ${phone/*-}
1414
$ echo ${phone/*-/abcd}
abcd1414
$ echo ${phone//*-}
1414
$ echo ${phone//*-/aaa}
aaa1414

# 将分隔符从:换成换行符
$ echo -e ${PATH//:/'\n'}
/usr/local/bin
/usr/bin
/bin
...
```

### 改变大小写
```bash
# 转为大写
${varname^^}

# 转为小写
${varname,,}
```
示例：
```bash
$ foo=heLLo
$ echo ${foo^^}
HELLO
$ echo ${foo,,}
hello
```

## 算术运算
### 算术表达式
`((...))`语法可以进行**整数**的算术运算，还会自动忽略内部的空格：
```bash
$ ((foo = 5 + 5))
$ echo $foo
10
```
此语法不返回值，只要算术结果不是`0`，命令就算执行成功。如果算术结果为0，命令就算执行失败。
```bash
$ ((2-5))
$ echo $?
0

$ ((2-2))
$ echo $?
1
```
如果要**读取**算术运算的结果，需要在`((...))`前面加上美元符号：`$((...))`，使其变成算术表达式，返回算术运算的值。
```bash
$ echo $((2 + 2))
4
```
支持的算术运算符:
运算符|说明
-|-
`+`|加法
`-`|减法
`*`|乘法
`/`|除法（整除）
`%`|余数
`**`|指数
`++`|自增运算（前缀或后缀）
`--`|自减运算（前缀或后缀）

注意，除法运算符的返回结果总是整数:
```bash
$ echo $((5 / 2))
2
```
- `++`或`--`
    - 作为**前缀**是先运算后返回值
    - 作为**后缀**是先返回值后运算

```bash
$ i=0
$ echo $i
0
$ echo $((i++))
0
$ echo $i
1
$ echo $((++i))
2
$ echo $i
2
```
`$((...))`内部可以用圆括号改变运算顺序:
```bash
$ echo $(( (2 + 3) * 4 ))
20
```

### 数值的进制
Bash 的数值默认都是十进制，但是在算术表达式中，也可以使用其他进制:
语法|说明
-|-
`number`|没有任何特殊表示法的数字是十进制数（以10为底）。
`0number`|八进制数。
`0xnumber`|十六进制数。
`base#number`|base进制的数。

```bash
# $(())会返回10进制的数值，16进制转10进制
$ echo $((0xff))
255

# 2进制转10进制
$ echo $((2#11111111))
255
```
### 位运算
`$((...))`支持以下的二进制位运算符：
运算符|说明
-|-
`<<`|位左移运算，把一个数字的所有位向左移动指定的位。
`>>`|位右移运算，把一个数字的所有位向右移动指定的位。
`&`|位的“与”运算，对两个数字的所有位执行一个`AND`操作。
`|`|位的“或”运算，对两个数字的所有位执行一个`OR`操作。
`~`|位的“否”运算，对一个数字的所有位取反。
`^`|位的异或运算，对两个数字的所有位执行一个异或操作。

示例：
```bash
# 右移，16的二进制 0001 0000 -> 0000 0100 -> 4
$ echo $((16>>2))
4

# 左移，16的二进制 0001 0000 -> 0100 0000 -> 64
$ echo $((16<<2))
64
```
### 逻辑运算
`$((...))`支持以下逻辑运算符：
运算符|说明
-|-
`<`|小于
`>`|大于
`<=`|小于或相等
`>=`|大于或相等
`==`|相等
`!=`|不相等
`&&`|逻辑与
`||`|逻辑或
`!`|逻辑否
`expr1?expr2:expr3`|三元条件运算符。<br>若表达式`expr1`的计算结果为非零值（算术真），则执行表达式`expr2`，否则执行表达式`expr3`。

如果逻辑表达式为真，返回`1`，否则返回`0`。
```bash
$ echo $((3 > 2))
1
$ echo $(( (3 > 2) || (4 <= 1) ))
1

# 三元条件运算符
$ a=0
$ echo $((a<1 ? 1 : 0))
1
$ echo $((a>1 ? 1 : 0))
0
```
### 赋值运算
`$((...))`支持以下赋值运算符：
运算符|说明
-|-
`var = value`|简单赋值。
`var += value`|等价于`var = var + value`。
`var -= value`|等价于`var = var – value`。
`var *= value`|等价于`var = var * value`。
`var /= value`|等价于`var = var / value`。
`var %= value`|等价于`var = var % value`。
`var <<= value`|等价于`var = var << value`。
`var >>= value`|等价于`var = var >> value`。
`var &= value`|等价于`var = var & value`。
`var |= value`|等价于`var = var | value`。
`var ^= value`|等价于`var = var ^ value`。

示例：
```bash
$ foo=5
$ echo $((foo*=2))
10
```
### 求值运算
逗号`,`在`$((...))`内部是求值运算符，执行前后两个表达式，并返回后一个表达式的值。
```bash
$ echo $((foo = 1 + 2, 3 * 4))
12
$ echo $foo
3
```
### expr 命令
`expr`命令支持算术运算，可以不使用`((...))`语法，但也不支持非整数运算。
```bash
$ expr 3 + 2
5

$ a=3
$ expr $a - 7
-4

# 不支持非整数
$ expr 3.5 + 2
expr: non-integer argument
```
> 注意，`expr`使用运算符两边要有空格

## read 命令
`read`命令用于从标准输入读取数值。

有时，脚本需要在执行过程中，由用户提供一部分数据，这时可以使用`read`命令。它将用户的输入存入一个变量，方便后面的代码使用。用户按下回车键，就表示输入结束。

格式：
```bash
read [-options] [variable...]
```
- `options`是参数选项
- `variable`是用来保存输入数值的一个或多个变量名
- 如果没有提供变量名，环境变量`REPLY`会包含用户输入的一整行数据

示例：`demo.sh`
```bash
#!/bin/bash

echo -n "输入一些文本 > "
read text
echo "你的输入：$text"
```
执行：
```bash
$ bash demo.sh
输入一些文本 > 你好，世界
你的输入：你好，世界
```
`read`可以接受用户输入的多个值，多个值之间用空格分开，分别读入到相应的多个变量中，多余值都会读入到最后一个变量中：`read_0.sh`
```bash
#!/usr/bin/env bash

echo -n "输入一些文本 > "
read text1 text2
echo "1: $text1"
echo "2: $text2"
```
执行：
```bash
$ ./read_0.sh
输入一些文本 > a b 1 3
1: a
2: b 1 3
```
还可以读取文件：
```bash
#!/bin/bash

filename=$(echo ~/py_msg.txt)

while read myline;do
        echo "$myline"
done < $filename
```
### 选项
选项|说明
-|-
`-a array`|以空格分隔顺序读取字符到数组array
`-d delim`|设置分隔符为delim
`-e`|开启tab自动补全
`-i text`|设置输入字符默认值为text
`-n nchars`|设置输入字符长度为nchars，<br>如果提前遇到分隔符则返回到分隔符前的字符
`-N nchars`|设置输入字符长度为nchars，忽略分隔符
`-p prompt`|设置输入前的提示信息prompt
`-r`|禁止反斜杠转义任何字符
`-s`|在输入字符时不再屏幕上显示，对密码有用
`-t timeout`|设置输入字符的等待时间<br>超过时间脚本将放弃等待，继续向下执行
`-u fd`|使用文件描述符fd作为输入

## 条件判断
### if
语法结构：
```bash
if commands; then
  commands
[elif commands; then
  commands...]
[else
  commands]
fi
```
示例：
```bash
# 可以写成两行，这时不需要分号
if true
then
  echo 'hello world'
fi
```
### test 命令
`if`结构的判断条件，一般使用`test`命令，有三种形式:
```bash
# 写法一
test expression

# 写法二
[ expression ]

# 写法三
[[ expression ]]
```
上面三种形式是等价的，但是第三种形式还支持正则判断，前两种不支持。

`expression`是一个表达式，如果表达式为真，`test`命令执行成功（返回值为`0`）；表达式为假，`test`命令执行失败（返回值为`1`）。

>注意，第二种和第三种写法，`[`和`]`与内部的表达式之间必须有空格。

示例：判断文件是否存在
```bash
$ test -f /etc/hosts
$ echo $?
0

# 或者
$ [ -f /etc/hosts ]
$  echo $?
0
```
### 文件判断
表达式|说明
-|-
`[ -b file ]`|如果 file 存在并且是一个块（设备）文件，则为true。
`[ -c file ]`|如果 file 存在并且是一个字符（设备）文件，则为true。
`[ -d file ]`|如果 file 存在并且是一个目录，则为true。
`[ -e file ]`|如果 file 存在，则为true。
`[ -f file ]`|如果 file 存在并且是一个普通文件，则为true。
`[ -g file ]`|如果 file 存在并且设置了组 ID，则为true。
`[ -G file ]`|如果 file 存在并且属于有效的组 ID，则为true。
`[ -h file ]`|如果 file 存在并且是符号链接，则为true。
`[ -k file ]`|如果 file 存在并且设置了它的“sticky bit”，则为true。
`[ -L file ]`|如果 file 存在并且是一个符号链接，则为true。
`[ -N file ]`|如果 file 存在并且自上次读取后已被修改，则为true。
`[ -O file ]`|如果 file 存在并且属于有效的用户 ID，则为true。
`[ -p file ]`|如果 file 存在并且是一个命名管道，则为true。
`[ -r file ]`|如果 file 存在并且可读（当前用户有可读权限），则为true。
`[ -s file ]`|如果 file 存在且其长度大于零，则为true。
`[ -S file ]`|如果 file 存在且是一个网络 socket，则为true。
`[ -t fd ]`|如果 fd 是一个文件描述符，并且重定向到终端，则为true。 <br>这可以用来判断是否重定向了标准输入／输出／错误。
`[ -u file ]`|如果 file 存在并且设置了 setuid 位，则为true。
`[ -w file ]`|如果 file 存在并且可写（当前用户拥有可写权限），则为true。
`[ -x file ]`|如果 file 存在并且可执行（有效用户有执行／搜索权限），则为true。
`[ file1 -nt file2 ]`|如果 FILE1 比 FILE2 的更新时间最近，或者 FILE1 存在而 FILE2 不存在，则为true。
`[ file1 -ot file2 ]`|如果 FILE1 比 FILE2 的更新时间更旧，或者 FILE2 存在而 FILE1 不存在，则为true。
`[ FILE1 -ef FILE2 ]`|如果 FILE1 和 FILE2 引用相同的设备和 inode 编号，则为true。

### 字符串判断

- `[ string ]`：如果string不为空（长度大于0），则判断为真。
- `[ -n string ]`：如果字符串string的长度大于零，则判断为真。
- `[ -z string ]`：如果字符串string的长度为零，则判断为真。
- `[ string1 = string2 ]`：如果string1和string2相同，则判断为真。
- `[ string1 == string2 ]`等同于`[ string1 = string2 ]`。
- `[ string1 != string2 ]`：如果string1和string2不相同，则判断为真。
- `[ string1 '>' string2 ]`：如果按照字典顺序string1排列在string2之后，则判断为真。
- `[ string1 '<' string2 ]`：如果按照字典顺序string1排列在string2之前，则判断为真。

>注意，`test`命令内部的`>`和`<`，必须用引号引起来（或者是用反斜杠转义）。否则，它们会被 shell 解释为重定向操作符。

### 整数判断
- `[ int1 -eq int2 ]`：如果int1等于int2，则为true。
- `[ int1 -ne int2 ]`：如果int1不等于int2，则为true。
- `[ int1 -le int2 ]`：如果int1小于或等于int2，则为true。
- `[ int1 -lt int2 ]`：如果int1小于int2，则为true。
- `[ int1 -ge int2 ]`：如果int1大于或等于int2，则为true。
- `[ int1 -gt int2 ]`：如果int1大于int2，则为true。

### 正则判断
`[[ expression ]]`这种判断形式，支持正则表达式。
```bash
[[ string1 =~ regex ]]
```
`regex`是一个正则表示式，`=~`是正则比较运算符。

示例：
```bash
#!/bin/bash

INT=-5

if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
  echo "INT is an integer."
  exit 0
else
  echo "INT is not an integer." >&2
  exit 1
fi
```
### test 判断的逻辑运算
通过逻辑运算，可以把多个`test`判断表达式结合起来，创造更复杂的判断。

三种逻辑运算AND，OR，和NOT，都有自己的专用符号。
- `AND`运算：符号`&&`，也可使用参数`-a`。
- `OR`运算：符号`||`，也可使用参数`-o`。
- `NOT`运算：符号`!`。

示例：
```bash
#!/bin/bash

MIN_VAL=1
MAX_VAL=100

INT=50

if [ ! \( $INT -ge $MIN_VAL -a $INT -le $MAX_VAL \) ]; then
    echo "$INT is outside $MIN_VAL to $MAX_VAL."
else
    echo "$INT is in range."
fi
```
使用否定操作符`!`时，最好用圆括号确定转义的范围。`test`命令内部使用的圆括号，必须使用引号或者转义，否则会被 Bash 解释。

### 算术判断
Bash 还提供了`((...))`作为算术条件，进行算术运算的判断。
```bash
if ((3 > 2)); then
  echo "true"
fi
```
如果算术计算的结果是非零值，则表示判断成立。这一点跟命令的返回值正好相反，需要小心。
```bash
$ if ((1)); then echo "It is true."; fi
It is true.
$ if ((0)); then echo "It is true."; else echo "it is false."; fi
It is false.
```
### case
`case`结构用于多值判断，可以为每个值指定对应的命令，跟包含多个`elif`的`if`结构等价，但是语义更好。它的语法如下:
```bash
case expression in
  pattern )
    commands ;;
  pattern )
    commands ;;
  ...
esac
```
`expression`是一个表达式，`pattern`是表达式的值或者一个模式，可以有多条，用来匹配多个值，每条以两个分号（`;`）结尾。
```bash
#!/bin/bash

echo -n "输入一个1到3之间的数字（包含两端）> "
read character
case $character in
  1 ) echo 1
    ;;
  2 ) echo 2
    ;;
  3 ) echo 3
    ;;
  * ) echo 输入不符合要求
esac
```
- `case`的匹配模式可以使用各种通配符:
- `a)`：匹配a。
- `a|b)`：匹配a或b。
- `[[:alpha:]])`：匹配单个字母。
- `???)`：匹配3个字符的单词。
- `*.txt)`：匹配`.txt`结尾。
- `*)`：匹配任意输入，通过作为`case`结构的最后一个模式。

```bash
echo -n "输入一个字母或数字 > "
read character
case $character in
  [[:lower:]] | [[:upper:]] ) echo "输入了字母 $character"
                              ;;
  [0-9] )                     echo "输入了数字 $character"
                              ;;
  * )                         echo "输入不符合要求"
esac
```
Bash 4.0之前，`case`结构只能匹配一个条件，然后就会退出`case`结构。

Bash 4.0之后，允许匹配多个条件，这时可以用`;;&`终止每个条件块。
```bash
read -n 1 -p "Type a character > "
echo
case $REPLY in
  [[:upper:]])    echo "'$REPLY' is upper case." ;;&
  [[:lower:]])    echo "'$REPLY' is lower case." ;;&
  [[:alpha:]])    echo "'$REPLY' is alphabetic." ;;&
  [[:digit:]])    echo "'$REPLY' is a digit." ;;&
  [[:graph:]])    echo "'$REPLY' is a visible character." ;;&
  [[:punct:]])    echo "'$REPLY' is a punctuation symbol." ;;&
  [[:space:]])    echo "'$REPLY' is a whitespace character." ;;&
  [[:xdigit:]])   echo "'$REPLY' is a hexadecimal digit." ;;&
esac
```

## 循环
### while
`while`循环有一个判断条件，只要符合条件，就不断循环执行指定的语句。
```bash
while condition; do
  commands
done
```
循环条件`condition`可以使用`test`命令，跟`if`结构的判断条件写法一致。
```bash
number=0
while [ "$number" -lt 10 ]; do
  echo "Number = $number"
  number=$((number + 1))
done
```
### until
`until`循环与`while`循环恰好相反，只要不符合判断条件（判断条件失败），就不断循环执行指定的语句。一旦符合判断条件，就退出循环。
```bash
until condition; do
  commands
done
```
`until`的条件部分也可以是一个命令，表示在这个命令执行成功之前，不断重复尝试。
```bash
until cp $1 $2; do
  echo 'Attempt to copy failed. waiting...'
  sleep 5
done
```
### for...in
`for...in`循环用于遍历列表的每一项。
```bash
for variable in list; do
  commands
done
```
`for`循环会依次从`list`列表中取出一项，作为变量`variable`，然后在循环体中进行处理。
```bash
#!/bin/bash

for i in word1 word2 word3; do
  echo $i
done
```
列表可以由通配符产生:
```bash
for i in *.png; do
  ls -l $i
done
```
`in list`的部分可以省略，这时`list`默认等于脚本的所有参数`$@`。但是，为了可读性，最好还是不要省略，参考下面的例子:
```bash
for filename; do
  echo "$filename"
done

# 等同于

for filename in "$@" ; do
  echo "$filename"
done
```
在函数体中也是一样的，`for...in`循环省略`in list`的部分，则`list`默认等于函数的所有参数。

### for
`for`循环还支持 C 语言的循环语法。
```bash
for (( expression1; expression2; expression3 )); do
  commands
done
```
`expression1`用来初始化循环条件，`expression2`用来决定循环结束的条件，`expression3`在每次循环迭代的末尾执行，用于更新值。

注意，循环条件放在双重圆括号之中。另外，圆括号之中使用变量，不必加上美元符号`$`。
```bash
for (( i=0; i<5; i=i+1 )); do
  echo $i
done
```
`for`条件部分的三个语句，都可以省略:
```bash
for ((;;))
do
  read var
  if [ "$var" = "." ]; then
    break
  fi
done
```
上面脚本会反复读取命令行输入，直到用户输入了一个点（`.`）为止，才会跳出循环。

### break，continue
`break`命令立即终止循环，程序继续执行循环块之后的语句，即不再执行剩下的循环。
```bash
#!/bin/bash

for number in 1 2 3 4 5 6
do
  echo "number is $number"
  if [ "$number" = "3" ]; then
    break
  fi
done
```
上面例子只会打印3行结果。一旦变量`$number`等于3，就会跳出循环，不再继续执行。

`continue`命令立即终止本轮循环，开始执行下一轮循环。

### select
`select`结构主要用来生成简单的菜单。它的语法与`for...in`循环基本一致。
```bash
select name
[in list]
do
  commands
done
```
Bash 会对`select`依次进行下面的处理:
1. `select`生成一个菜单，内容是列表list的每一项，并且每一项前面还有一个数字编号。
2. Bash 提示用户选择一项，输入它的编号。
3. 用户输入以后，Bash 会将该项的内容存在变量`name`，该项的编号存入环境变量`REPLY`。如果用户没有输入，就按回车键，Bash 会重新输出菜单，让用户选择。
4. 执行命令体`commands`。
5. 执行结束后，回到第一步，重复这个过程。

实力：
```bash
#!/bin/bash
# select.sh

select brand in Samsung Sony iphone symphony Walton
do
  echo "You have chosen $brand"
done
```
执行：
```bash
$ ./select.sh
1) Samsung
2) Sony
3) iphone
4) symphony
5) Walton
#?
```
如果用户没有输入编号，直接按回车键。Bash 就会重新输出一遍这个菜单，直到用户按下`Ctrl + c`，退出执行。

`select`可以与`case`结合，针对不同项，执行不同的命令。

## 函数
函数定义的语法有两种：
```bash
# 第一种
fn() {
  # codes
}

# 第二种
function fn() {
  # codes
}
```
`fn`是自定义的函数名，函数代码就写在大括号之中。这两种写法是等价的。

示例：
```bash
#!/bin/bash

# 函数体里面的$1表示函数调用时的第一个参数。
hello(){
        echo "Hello $1"
}

hello "this is hello func"
```
执行：
```bash
$ ./func_0.sh
Hello this is hello func
```
删除一个函数，可以使用`unset`命令:
```bash
unset -f functionName
```
### 参数变量
函数体内可以使用参数变量，获取函数参数。函数的参数变量，与脚本参数变量是一致的。
- `$1~$9`：函数的第一个到第9个的参数。
- 如果函数的参数多于9个，那么第10个参数可以用`${10}`的形式引用，以此类推。
- `$0`：函数所在的脚本名。
- `$#`：函数的参数总数。
- `$@`：函数的全部参数，参数之间使用空格分隔。
- `$*`：函数的全部参数，参数之间使用变量`$IFS`值的第一个字符分隔，默认为空格，但是可以自定义。

### return 命令
`return`命令用于从函数返回一个值。函数执行到这条命令，就不再往下执行了，直接返回了。
```bash
fn() {
  return 10
}
```
函数将返回值返回给调用者。如果命令行直接执行函数，下一个命令可以用`$?`拿到返回值。

`return`后面不跟参数，只用于返回也是可以的:
```bash
fn() {
  commond
  ...
  return
}
```
### 全局和局部变量，local 命令
Bash 函数体内直接声明的变量，属于全局变量，整个脚本都可以读取。也可以直接修改全局变量。
```bash
#!/bin/bash

# func_1.sh
fn(){
        foo=1
        echo "fn: foo = $foo"
}

fn

echo "global: foo = $foo"
```
执行：
```bash
$ ./func_1.sh
fn: foo = 1
global: foo = 1
```
上面例子中，变量`$foo`是在函数`fn`内部声明的，函数体外也可以读取。

函数里面可以用`local`命令声明局部变量。
```bash
#!/bin/bash

# func_2.sh
fn(){
        local foo
        foo=1
        echo "fn: foo = $foo"
}

fn

echo "global: foo = $foo"
```
执行：
```bash
$ ./func_2.sh
fn: foo = 1
global: foo =
```
上面例子中，`local`命令声明的`$foo`变量，只在函数体内有效，函数体外没有定义。

## 数组
数组（array）是一个包含多个值的变量。成员的编号从`0`开始，数量没有上限，也没有要求成员被连续索引。

### 创建数组
```bash
arr=(value1 value2 ... valueN)

# 等同于

arr=(
  value1
  value2
  ...
  valueN
)

# 等同于

arr[0]=value1
arr[1]=value2
...
arr[N-1]=valueN
```
`arr`是数组的名字，可以是任意合法的变量名。索引从`0`开始，是一个大于或等于零的整数，也可以是算术表达式。

可以指定索引创建数组：
```bash
$ array=(a b c)
$ array=([2]=c [0]=a [1]=b)
```
可以使用通配符:
```bash
$ mp3s=( *.mp3 )
```
`declare -a`命令声明一个数组或`read -a`命令则是将用户的命令行输入读入一个数组。
```bash
$ declare -a arr1

$ read -a arr2
```
### 读取数组
```bash
# 读取单个元素
$ arr1=(a b c)
$ echo ${arr1[1]}
b

# 如果没有指定索引，默认读取索引 0 位置的值
$ echo ${arr1}
a

# 读取所有成员
$ echo ${arr1[@]}
a b c

# 或
$ echo ${arr1[*]}
a b c
```
### 读取数组索引
```bash
$ arr=([5]=a [9]=b [23]=c)
$ echo ${!arr[@]}
5 9 23
$ echo ${!arr[*]}
5 9 23
```
利用这个语法，也可以通过`for`循环遍历数组:
```bash
arr=(a b c d)

for i in ${!arr[@]};do
  echo ${arr[i]}
done
```

### 遍历数组
使用`for...in`遍历数组。

遍历读取数组时，放不放在双引号之中，是有差别的。

使用`*`或`@`时，不放在双引号之中，结果都会解析双引号中的空格：
```bash
#!/bin/bash

arr1=(a b "cc 33" d "ee 55" f)

for i in ${arr1[@]}; do
        echo $i
done

# 执行结果：
$ ./arr_for_0.sh
a
b
cc
33
d
ee
55
f
```
放在双引号之中：
```bash
#!/bin/bash

arr1=(a b "cc 33" d "ee 55" f)

for i in "${arr1[@]}"; do
        echo $i
done

# 执行结果：
$ ./arr_for_0.sh
a
b
cc 33
d
ee 55
f
```
`*`: `${arr1[*]}`放在双引号之中，所有元素就会变成单个字符串返回。
```bash
#!/bin/bash

arr1=(a b "cc 33" d "ee 55" f)

for i in "${arr1[*]}"; do
        echo $i
done

# 执行结果：
$ ./arr_for_0.sh
a b cc 33 d ee 55 f
```

### 数组的长度
```bash
${#array[*]}
${#array[@]}
```
示例：
```bash
$ a[100]=foo

$ echo ${#a[*]}
1

$ echo ${#a[@]}
1
```
### 提取数组成员
`${array[@]:position:length}`
```bash
$ arr1=(a b c d e f)
$ echo ${arr1[@]:1:1}
b
$ echo ${arr1[@]:1:3}
b c d

# 省略length，则返回从指定位置开始到最后的所有成员
$ echo ${arr1[@]:3}
d e f
```
### 追加数组成员
数组末尾追加成员，可以使用`+=`赋值运算符。它能够自动地把值追加到数组末尾。
```bash
$ arr1=(a b c)
$ arr1+=(d e)
$ echo ${arr1[@]}
a b c d e
```
### 删除数组成员
删除数组成员，使用`unset`命令。
```bash
$ arr1=(a b c)
$ unset arr1[1]
$ echo ${arr1[@]}
a c
```
将某个成员设为空值，可以从返回值中“隐藏”这个成员。“隐藏”不是删除，成员还存在，数组长度也没变，只是变为空值。
```bash
$ arr1=(a b c)
$ arr1[1]=
$ echo ${arr1[@]}
a c
$ echo ${#arr1[@]}
3
```
`unset ArrayName`可以清空整个数组。
```bash
$ arr1=(a b c)
$ echo ${arr1[@]}
a b c
$ unset arr1
$ echo ${arr1[@]}

$ echo ${#arr1[@]}
0
```
### 关联数组
Bash 的新版本支持关联数组。关联数组**使用字符串而不是整数作为数组索引**。

`declare -A`可以声明关联数组。
```bash
$ declare -A colors
$ colors["red"]="#ff0000"
$ colors["green"]="#00ff00"
$ colors["blue"]="#0000ff"

$ echo ${colors[@]}
#ff0000 #00ff00 #0000ff
$ echo ${#colors[@]}
3
$ echo ${colors['blue']}
#0000ff
```

## set 命令
`set`命令用来修改子 Shell 环境的运行参数，即定制环境。

如果命令行下不带任何参数，直接运行`set`，会显示所有的环境变量和 Shell 函数。
```bash
$ set
```
### -u
`set -u`：在脚本运行中遇到不存在的变量时报错并退出脚本运行。

执行脚本时，如果遇到不存在的变量，Bash 默认忽略它：
```bash
#!/usr/bin/env bash

echo $a
echo bar
```
上面代码中，$a是一个不存在的变量。执行结果如下：
```bash
$ bash script.sh

bar
```
可以看到，`echo $a`输出了一个空行，Bash 忽略了不存在的`$a`，然后继续执行`echo bar`。大多数情况下，这不是开发者想要的行为，遇到变量不存在，脚本应该报错，而不是一声不响地往下执行。

`set -u`就用来改变这种行为。脚本在头部加上它，遇到不存在的变量就会报错，并停止执行。
```bash
#!/usr/bin/env bash
set -u

echo $a
echo bar
```
运行结果如下:
```bash
$ bash set_0.sh
set_0.sh: line 5: a: unbound variable
```
可以看到，脚本报错了，并且不再执行后面的语句。

`-u`还有另一种写法`-o nounset`，两者是等价的。
```bash
set -o nounset
```
### -x
默认情况下，脚本执行后，只输出运行结果，没有其他内容。如果多个命令连续执行，它们的运行结果就会连续输出。有时会分不清，某一段内容是什么命令产生的。

`set -x`用来在运行结果之前，先输出执行的那一行命令。
```bash
#!/usr/bin/env bash
set -x

echo bar
```
执行上面的脚本，结果如下:
```bash
$ bash script.sh
+ echo bar
bar
```
这对于调试复杂的脚本是很有用的。

`-x`还有另一种写法`-o xtrace`：
```bash
set -o xtrace
```
脚本当中如果要关闭命令输出，可以使用`set +x`。
```bash
#!/bin/bash

number=1

set -x
if [ $number = "1" ]; then
  echo "Number equals 1"
else
  echo "Number does not equal 1"
fi
set +x
```
上面的例子中，只对**特定的代码段**打开命令输出。

### -e
如果脚本里面有运行失败的命令（返回值非`0`），Bash 默认会继续执行后面的命令。

`set -e`使得脚本只要发生错误，就终止执行。

`-e`打开；`+e`关闭。可以只对特定命令段使用，某些命令的非零返回值可能不表示失败，或者开发者希望在命令失败的情况下，脚本继续执行下去：
```bash
set +e # 关闭
command1
command2
set -e # 打开
```
`-e`还有另一种写法`-o errexit`。

### -E
一旦设置了-e参数，会导致函数内的错误不会被`trap`命令捕获。`-E`参数可以纠正这个行为，使得函数也能继承`trap`命令。

### -o pipefail
`set -e`有一个例外情况，就是不适用于管道命令。

`set -o pipefail`用来解决这种情况，只要一个子命令失败，整个管道命令就失败，脚本就会终止执行。

### 其它
- `set -n`：等同于`set -o noexec`，不运行命令，只检查语法是否正确。
- `set -f`：等同于`set -o noglob`，表示不对通配符进行文件名扩展。可以使用`set +f`关闭。
- `set -v`：等同于`set -o verbose`，表示打印 Shell 接收到的每一行输入。可以使用`set +v`关闭。
- `set -o noclobber`：防止使用重定向运算符`>`覆盖已经存在的文件。

### 总结
`set`命令的几个参数，一般都放在一起使用：
```bash
# 写法一
set -Eeuxo pipefail

# 写法二
set -Eeux
set -o pipefail
```
这两种写法建议放在所有 Bash 脚本的头部。

另一种办法是在执行 Bash 脚本的时候，从命令行传入这些参数:
```bash
$ bash -euxo pipefail script.sh
```

## shopt 命令
`shopt`命令用来调整 Shell 的参数，跟`set`命令的作用很类似。之所以会有这两个类似命令的主要原因是，`set`是从 Ksh 继承的，属于 POSIX 规范的一部分，而`shopt`是 Bash 特有的。

直接输入`shopt`可以查看所有参数，以及它们各自打开和关闭的状态
```bash
$ shopt
```
`shopt`命令后面跟着参数名，可以查询该参数是否打开:
```bash
$ shopt globstar
globstar        off
```

选项：
- `-s param`：打开某个参数param。
- `-u param`：关闭某个参数param。
- `-q param`：查询某个参数是否打开，但不是直接输出查询结果，而是通过命令的执行状态（`$?`）表示查询结果。如果状态为`0`，表示该参数打开；如果为`1`，表示该参数关闭。主要用于脚本，供`if`条件结构使用。

## mktemp 命令
`mktemp`命令是为安全创建临时文件而设计的。虽然在创建临时文件之前，它不会检查临时文件是否存在，但是它支持唯一文件名和清除机制，因此可以减轻安全攻击的风险。

直接运行`mktemp`命令，就能生成一个临时文件。
```bash
$ mktemp
/tmp/tmp.1AA63DzEwn

$ ls -l /tmp/tmp.1AA63DzEwn
-rw------- 1 qlel qlel 0 Mar 30 16:06 /tmp/tmp.1AA63DzEwn
```
上面命令中，`mktemp`命令生成的临时文件名是随机的，而且权限是**只有用户本人可读写**。

在脚本中使用`mktemp`命令后面最好使用 OR 运算符（`||`），保证创建失败时退出脚本。为了保证脚本退出时临时文件被删除，可以使用`trap`命令指定退出时的清除操作。
```bash
#!/bin/bash

# 遇到 EXIT 信号就执行清除命令
trap 'rm -f "$TMPFILE"' EXIT

TMPFILE=$(mktemp) || exit 1
echo "Our temp file is $TMPFILE"
```
选项：

`-d`参数可以创建一个临时目录。
```bash
$ mktemp -d
/tmp/tmp.Wcau5UjmN6
```

`-p`参数可以指定临时文件所在的目录。默认是使用`$TMPDIR`环境变量指定的目录，如果这个变量没设置，那么使用`/tmp`目录。
```bash
$ mktemp -p /home/qlel/
/home/qlel/tmp.FOKEtvs2H3
```

`-t`参数可以指定临时文件的文件名模板，模板的末尾必须至少包含三个连续的`X`字符，表示随机字符，建议至少使用六个`X`。默认的文件名模板是`tmp.`后接十个随机字符。
```bash
$ mktemp -t mytemp.XXXXXXX
/tmp/mytemp.yZ1HgZV
```

`-u`仅返回一个文件名，并不会真的创建文件，可以用来生成随机数
```bash
$ mktemp -u
/tmp/tmp.VjgZMYMzMm

$ mktemp -u XXX
9M4
```

## trap 命令
`trap`命令用来在 Bash 脚本中响应系统信号。

`trap`命令的`-l`参数，可以列出所有的系统信号：
```bash
$ trap -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```
`trap`的命令格式:
```bash
$ trap [动作] [信号1] [信号2] ...
```
“动作”是一个 Bash 命令，“信号”常用的有以下几个:
- `HUP`：编号1，脚本与所在的终端脱离联系。
- `INT`：编号2，用户按下 Ctrl + C，意图让脚本终止运行。
- `QUIT`：编号3，用户按下 Ctrl + 斜杠，意图退出脚本。
- `KILL`：编号9，该信号用于杀死进程。
- `TERM`：编号15，这是kill命令发出的默认信号。
- `EXIT`：编号0，这不是系统信号，而是 Bash 脚本特有的信号，不管什么情况，只要退出脚本就会产生。

```bash
$ trap 'rm -f "$TMPFILE"' EXIT
```
上面命令中，脚本遇到`EXIT`信号时，就会执行`rm -f "$TMPFILE"`。

`trap` 命令的常见使用场景，就是在 Bash 脚本中指定退出时执行的清理命令
```bash
#!/bin/bash

trap 'rm -f "$TMPFILE"' EXIT

TMPFILE=$(mktemp) || exit 1
ls /etc > $TMPFILE
if grep -qi "kernel" $TMPFILE; then
  echo 'find'
fi
```
上面代码中，不管是脚本正常执行结束，还是用户按 Ctrl + C 终止，都会产生`EXIT`信号，从而触发删除临时文件。

>注意，`trap`命令必须放在脚本的开头。否则，它上方的任何命令导致脚本退出，都不会被它捕获。

如果`trap`需要触发多条命令，可以封装一个 Bash 函数:
```bash
function egress {
  command1
  command2
  command3
}

trap egress EXIT
```