+++
title = "Bash One Liners 3 重定向相关翻译与理解"
date = 2021-07-10T14:25:39+08:00
description = ""
tags = ['linux', 'shell', 'bash']
aplayer = false
showToc = true
math = true
showLicense = false
+++
重定向其实是通过操作**文件描述符**来完成的，这样会更容易理解。

<!--more-->
## 0_概述
当 Bash 启动时，会自动创建三个标准的文件描述符，它们分别是 `stdin`（标准输入，文件描述符为`0`），`stdout`（标准输出，文件描述符为`1`）和 `stderr`（标准错误输出，文件描述符为`2`）。你也可以创建更多的文件描述符，例如3，4，5等等，或者关闭它们，又或者拷贝它们。你可以从对应的文件中读取或者写入内容。

文件描述符总是指向某个文件（除非它们被关闭）。通常情况下，Bash 启动的三个文件描述符 —— `stdin`，`stdout` 和 `stderr` 都是指向你的终端，从终端输入中读取内容，并且把标准输出和标准错误都送到终端上。

重定向符号：
- `>` 输出重定向到一个文件或设备 覆盖原来的文件
- `>!` 输出重定向到一个文件或设备 强制覆盖原来的文件
- `>>` 输出重定向到一个文件或设备 追加原来的文件
- `<` 输入重定向到一个程序 
- `>&`、`&>`、`2>&1` 将标准输出和标准错误重定向到覆盖文件
- `|&` 将一个进程的 stdout 和 stderr 发送到另一个进程的 stdin

>这是记住这个结构的一种方法（尽管它并不完全准确）：起初，`2>1` 可能看起来是将 stderr 重定向到 stdout 的好方法。 但是，它实际上会被解释为“将 stderr 重定向到名为 `1` 的文件”。 `&` **表示后面和前面的是文件描述符而不是文件名**。 所以构造变成：`2>&1`。

假设你的终端对应的设备文件是 `/dev/tty0`，下面是 Bash 启动时文件描述符表的样子：

![1.png](/BashOneLiners/1.png)

当 Bash 执行一个命令时，他会 fork 一个子进程（查看`man 2 fork`）。子进程会从父进程继承所有的文件描述符，设置好指定的重定向，最后执行该命令（查看`man 3 exec`）。

以下尝试用图表来可视化展现，重定向发生时文件描述符表的变化过程，这种方法可以帮助你更好的理解重定向功能。

## 1_将命令的标准输出stdout重定向到文件
```bash
$ command >file
```
`>`是输出重定向操作符。Bash 首先会打开文件准备写入，如果文件打开成功，则将命令command的 `stdout` 指向之前打开的文件。如果打开失败，则不会继续执行命令。

`command >file`的写法和`command 1>file`的写法是一样的，`1`是 `stdout` 对应的文件描述符。

下面的图片描述了上述命令执行时文件描述符表的变化过程。Bash 打开文件并将文件描述符 `1` 重定向由`/dev/tty0`替换为指向文件`file`的文件描述符。 因此，从现在开始写入文件描述符 `1` 的所有输出最终都会写入文件：

![redirect-stdout.png](/BashOneLiners/redirect-stdout.png)

## 2_将命令的标准错误stderr重定向到文件
```bash
$ command 2> file
```
Bash 打开文件进行写入，获取该文件的文件描述符，并将文件描述符 `2` 替换为该文件的文件描述符。 所以现在任何写入 `stderr` 的内容都会写入文件。

![redirect-stderr.png](/BashOneLiners/redirect-stderr.png)

## 3_将stdout和stderr都重定向到一个文件
```bash
$ command &>file

# 或者
$ command >&file
```
这一行命令使用了`&>`操作符，它将命令`command`的 stdout 和 stderr 都重定向到文件`file`中。

以下是 bash 重定向两个流后文件描述符表的样子：

![redirect-stdout-stderr.png](/BashOneLiners/redirect-stdout-stderr.png)

如图所示，stdout 和 stderr 现在都指向文件。 因此，任何写入stdout和stderr的内容都会写入文件。

除此之外，还有几种方法可以将 stdout 和 stderr 同时重定向到同一个文件中。
```bash
$ command >file 2>&1
```
这是将两个流重定向到文件的更常见的方法。 首先将 stdout 重定向到文件`file`，然后将 stderr 复制为与 stdout 相同。 所以两个流最终都指向文件。

当 Bash 在命令中遇到多个重定向操作时，它会**从左到右**依次处理。我们通过图表来依次推导这整个过程。初始时文件描述符表的样子：

![1.png](/BashOneLiners/1.png)

现在 Bash 处理第一个重定向`>file`，之前已经解释过，它将使得 stdout 指向文件`file`：

![redirect-stdout.png](/BashOneLiners/redirect-stdout.png)

接下来，Bash 开始处理第二个重定向`2>&1`，它会把 stderr 重定向到 stdout 所指向的文件：

![redirect-stdout-stderr.png](/BashOneLiners/redirect-stdout-stderr.png)

两个流都已重定向到文件。

***
这里要注意不要错误的写成：
```bash
# 错误！
$ command 2>&1 >file
```
在bash中重定向的顺序很重要！此命令仅将 stdout 重定向到文件。stderr 仍将打印到终端。为了理解为什么会发生这种情况，让我们再看一遍这些步骤。因此，在运行命令之前，文件描述符表如下所示：

![1.png](/BashOneLiners/1.png)

现在 bash 从左到右处理重定向。 它首先看到 `2>&1` 所以它复制 stderr 到 stdout。 文件描述符表变为：

![duplicate-stderr-stdout.png](/BashOneLiners/duplicate-stderr-stdout.png)

现在 bash 看到第二个重定向 `>file` 并将stdout重定向到文件`file`：

![duplicate-stderr-stdout-stdout-file.png](/BashOneLiners/duplicate-stderr-stdout-stdout-file.png)

如同所示，stdout 指向了文件 `file`，但是 stderr 依然指向终端屏幕。所以，一定要注意重定向的书写顺序。

## 4_丢弃命令的 stdout 输出
```bash
$ command > /dev/null
```
特殊文件 `/dev/null` 会丢弃所有写入其中的数据。 所以我们在这里做的是将标准输出重定向到这个特殊文件，它会被丢弃。 从文件描述符表的角度来看，它是这样的：

![redirect-stdout-dev-null.png](/BashOneLiners/redirect-stdout-dev-null.png)

类似的，基于前一条命令，我们可以做到把输出到 stdout 和 stderr 的内容都丢弃：
```bash
$ command >/dev/null 2>&1

# 或者
$ command &>/dev/null
```
此时的文件描述符表为：

![redirect-stdout-stderr-dev-null.png](/BashOneLiners/redirect-stdout-stderr-dev-null.png)

## 5_将文件内容重定向到命令的stdin
```bash
$ command <file
```
这里 bash 尝试在运行任何命令之前打开文件进行读取。 如果打开文件失败，bash 会错误退出并且不运行命令。 如果打开文件成功，bash 使用打开文件的文件描述符作为命令的标准输入文件描述符。

完成后，文件描述符表如下所示：

![redirect-stdin.png](/BashOneLiners/redirect-stdin.png)

下面是一个例子，假如你想把文件的第一行读入到变量中：
```bash
$ read -r line < file
```
Bash 的内置读取命令从标准输入中读取一行。 通过使用输入重定向运算符 `<`，我们将其设置为从文件中读取行。

## 6_重定向一堆字符串到命令的 stdin
```bash
$ command <<EOL
your
multi-line
text
goes
here
EOL
```
这里用到了 here document 的语法`<<MARKER`。当 Bash 遇到该操作符是，它会从标准输入读取每一行，直到遇到一行以`MARKER`开头为止。这个例子中，Bash 读取到所有内容并传给`command`的 stdin。

假设你想去除一堆 URL 地址中的`http://`部分，可以用下面的一行命令：
```bash
$ sed 's|http://||' <<EOF
http://url1.com
http://url2.com
http://url3.com
EOF
```
输出结果：
```bash
url1.com
url2.com
url3.com
```

## 7_重定向一行文本到命令的 stdin
```bash
$ command <<< "foo bar baz"

# 等价于
$ echo "foo bar baz" | command
```

## 8_将所有命令的 stderr 永久重定向到一个文件
```bash
$ exec 2>file
$ command1
$ command2
$ ...
```
这一行命令中使用了 Bash 的内置命令`exec`。如果你在它之后指定重定向操作，重定向的效果为一直持续到退出脚本或者shell为止。

在这个例子中，`2>file`处理之后，随后所有命令的 stderr 都会重定向到文件`file`中。通过这种方法，你可以很方便的把脚本中所有命令的 stderr 都汇总到一个文件，同时又不用每一个命令之后都指定`2>file`。

一般情况下，`exec`可以接受命令的可选参数。如果指定了它，bash将使用该命令替换自身。因此，您得到的只是运行那个命令，没有创建更多的shell。

## 9_使用自定义文件描述符打开文件进行读取
```bash
$ exec 3<file
```
这里我们再次使用 `exec` 命令并指定 `3<file` 重定向到它。 它的作用是打开文件进行读取，并将打开的文件描述符分配给 shell 的文件描述符编号 `3`。文件描述符表现在看起来像这样：

![custom-fd.png](/BashOneLiners/custom-fd.png)

现在可以从文件描述符`3`中读取，如下所示：
```bash
$ read -u 3 line
```
这是从文件描述符`3`中读取一行。

一些常规的命令，例如 `grep`，还可以这么用：
```bash
$ grep "foo" <&3
```
这里发生的是文件描述符 `3` 被复制到文件描述符 `1`，也就是 `grep` 的 stdin 。 请记住，一旦读取了文件描述符，它就会被耗尽，需要关闭它并再次打开它才能使用它。 （不能在 bash 中倒带文件描述符 `fd`。）

使用完文件描述符 fd `3` 后，您可以通过以下方式关闭它：
```bash
$ exec 3>&-
```
这里的文件描述符`3`被欺骗到`-`，这是bash关闭这个文件描述符的特殊方式。

## 10_使用自定义文件描述符打开文件进行写入
```bash
$ exec 4>file
```
在这里，我们只需告诉bash打开文件进行写入，并为其分配编号`4`。文件描述符表如下所示：

![custom-fd-writing.png](/BashOneLiners/custom-fd-writing.png)

如您所见，文件描述符不必按顺序使用，您可以打开任何您喜欢的文件描述符编号，从 `0` 到 `255`。

现在我们可以简单地写入文件描述符`4`：
```bash
$ echo "foo" >&4
```
可以关闭文件描述符 `4`：
```bash
$ exec 4>&-
```

## 11_打开一个文件进行写入和读取
```bash
$ exec 3<>file
```
这里我们使用菱形操作符 `<>`。 菱形操作符打开一个文件描述符用于读取和写入。

例如：
```bash
$ echo "foo bar" > file   # 写入字符串 "foo bar" 到文件 "file".
$ exec 5<> file           # 以读写方式打开 "file" 到文件描述符 5
$ read -n 3 var <&5       # 从 fd 5 读取3个字符
$ echo $var
```
现在我们可以向文件中写入一些内容：
```bash
$ echo -n + >&5           # 在第4个位置写入 '+'
$ exec 5>&-               # 关闭 fd 5.
$ cat file
```
这将输出 `foo+bar`，因为我们在文件的第 4 个位置写了 + 字符。

## 12_将多个命令的输出发送到一个文件
```bash
$ (command1; command2) >file
```
这一行命令使用`(commands)`语法，commands 会在一个子 shell 中执行。所以在这里，`command1`和`command2`会在子 shell 中运行，然后 Bash 将子 shell 的 stdout 重定向到文件中。

## 13_通过文件在shell中执行命令
打开两个 shell，在shell_1中执行以下命令：
```bash
mkfifo fifo
exec < fifo
```
而在shell_2中，执行：
```bash
exec 3> fifo;
echo 'echo test' >&3
```
现在看看shell_1。它会执行`echo test`。 可以继续向 `fifo` 写入命令，shell_1 将继续执行它们。

以下是原理：

在 shell_1 中，我们使用 `mkfifo` 命令创建一个名为 `fifo` 的命名管道。 命名管道（也称为 FIFO）类似于常规管道，不同之处在于它作为文件系统的一部分进行访问。 它可以被多个进程打开以进行读取或写入。 当进程通过 FIFO 交换数据时，内核会在内部传递所有数据，而不会将其写入文件系统。 因此，FIFO 特殊文件在文件系统上没有内容； 文件系统条目仅用作参考点，以便进程可以使用文件系统中的名称访问管道。

接下来，我们通过`exec < fifo`命令，使用 `fifo` 作为当前 shell 的标准输入stdin。

现在在 shell_2 中，我们打开命名管道进行写入，并为其分配一个自定义文件描述符 `3`。接下来我们简单地将 `echo test` 写入文件描述符 `3`，该文件将转到 `fifo`。

由于 shell_1 的 stdin 连接到这个管道，它会执行它！

## 14_通过bash访问一个网站
```bash
$ exec 3<>/dev/tcp/www.baidu.com/80
$ echo -e "GET / HTTP/1.1\n\n" >&3
$ cat <&3
```
Bash 将 `/dev/tcp/host/port` 视为特殊文件。 它不需要存在于您的系统中。 这个特殊文件用于通过 bash 打开 tcp 连接。

在本例中，我们首先打开文件描述符 `3` 进行读写，并将其指向 `/dev/tcp/www.baidu.com/80` 特殊文件，该文件是通过端口 `80` 连接到 `www.baidu.com`。

接下来我们将 `GET / HTTP/1.1\n\n` 写入文件描述符 `3`。然后我们使用 `cat` 从相同的文件描述符读取响应。

同样，您可以通过 `/dev/udp/host/port` 特殊文件创建 UDP 连接。

使用 `/dev/tcp/host/port`，您甚至可以在 bash 中编写诸如端口扫描器之类的东西！

## 15_重定向输出时防止覆盖文件内容
```bash
$ set -o noclobber
```
这将打开当前 shell 的 `noclobber` 选项。 `noclobber` 选项可防止您使用 `>` 运算符覆盖现有文件。

如果你尝试将输出重定向到一个存在的文件，你会得到一个错误：
```bash
$ program > file
bash: file: cannot overwrite existing file
```
如果100%确定要覆盖文件，请使用`>|`重定向操作符：
```bash
$ program >| file
```
这会成功，因为它覆盖了 `noclobber` 选项。

## 16_将标准输入重定向到文件并将其打印到标准输出
```bash
$ command | tee file
```
`tee` 命令非常方便。 它不是 bash 的一部分，但您会经常使用它。 它接受一个输入流并将其打印到标准输出和文件中

在这个例子中，它接受命令的标准输出，把它放在文件中，然后把它打印到标准输出。

以下是其工作原理的图示：

![tee.png](/BashOneLiners/tee.png)

## 17_将一个进程的标准输出发送到另一个进程的标准输入
```bash
$ command1 | command2
```
一个管道`|`将 `command1` 的 stdout 与 `command2` 的 stdin 连接起来。

可以用图形来说明：

![pipe.png](/BashOneLiners/pipe.png)

## 18_将一个进程的 stdout 和 stderr 发送到另一个进程的 stdin
```bash
$ command1 |& command2
```
这适用于从 4.0 开始的 bash 版本。 `|&` 重定向操作符通过管道将 `command1` 的 stdout 和 stderr 发送到 `command2` 的 stdin。

由于 bash 4.0 的新特性没有被广泛使用，旧的、更便携的方法是：
```bash
$ command1 2>&1 | command2
```
下图显示了文件描述符发生的情况：

![pipe-stdout-stderr.png](/BashOneLiners/pipe-stdout-stderr.png)

## 19_给出文件描述符名称
```bash
$ exec {filew}>output_file
```
命名文件描述符是 bash 4.1 的一个特性。 命名文件描述符看起来像 `{varname}`。 您可以像使用常规的数字文件描述符一样使用它们。 Bash 在内部选择一个空闲的文件描述符并为其分配一个名称。

## 20_重定向顺序
可以把重定向放在你想要的任何地方。看看这3个例子，它们都是一样的：
```bash
$ echo hello >/tmp/example

$ echo >/tmp/example hello

$ >/tmp/example echo hello
```

## 21_交换stdout和stderr
```bash
$ command 3>&1 1>&2 2>&3
```
这里我们首先复制文件描述符 `3` 作为 stdout 的副本。 然后我们复制stdout 为stderr 的副本，最后我们复制stderr 为文件描述符`3` 的副本，即stdout。 因此，我们交换了 stdout 和 stderr。

让我们通过插图来了解每个重定向。 在运行命令之前，我们有指向终端的文件描述符：

![1.png](/BashOneLiners/1.png)

接下来 bash 设置 `3>&1` 重定向。 这将创建文件描述符 `3` 作为文件描述符 `1` 的副本：

![fd3-copy-of-fd1.png](/BashOneLiners/fd3-copy-of-fd1.png)

接下来 bash 设置 `1>&2` 重定向。 这使得文件描述符 `1` 成为文件描述符 `2` 的副本：

![fd1-copy-of-fd2.png](/BashOneLiners/fd1-copy-of-fd2.png)

接下来 bash 设置 `2>&3` 重定向。 这使得文件描述符 `2` 成为文件描述符 `3` 的副本：

![fd2-copy-of-fd3.png](/BashOneLiners/fd2-copy-of-fd3.png)

如果你是一个追求完美的人，可以将文件描述符3关闭：
```bash
$ command 3>&1 1>&2 2>&3 3>&-
```
最终的文件描述符图会是这样的：

![fd1-fd2-swap.png](/BashOneLiners/fd1-fd2-swap.png)

## 22_将 stdout 发送到一个进程并将 stderr 发送到另一个进程
```bash
$ command > >(stdout_cmd) 2> >(stderr_cmd)
```
这一行命令用到了进程替换（Process Substitution）语法。`>(...)`操作符的执行过程是，运行里面的命令，同时将命令的标准输入连接到一个命名管道的读取部分。Bash 随后会用命名管道的实际文件名替换这个操作符。

例如，第一个替换 `>(stdout_cmd)` 可能返回 `/dev/fd/60`，第二个替换可能返回 `/dev/fd/61`。 这两个文件都是 bash 即时创建的命名管道。 两个命名管道都有作为读取器的命令。 命令等待某人写入管道，以便他们可以读取数据。

替换后，最初的命令变成以下形式：
```bash
$ command > /dev/fd/60 2> /dev/fd/61
```
以上，标准输出被重定向到 `/dev/fd/60`，标准错误被重定向到 `/dev/fd/61`。

当命令执行是输出内容到 stdout，则管道`/dev/fd/60`后面的进程（stdout_cmd）会从另外一侧读取到数据。同样的，进程 stderr_cmd 也能从命令的 stderr 输出中读取。

## 23_查找所有管道命令的退出代码
假设您运行了几个通过管道连接在一起的命令：
```bash
$ cmd1 | cmd2 | cmd3 | cmd4
```
然后你想获取所有命令的退出码，但是这里并没有一种简单的做法可以实现，因为 Bash 只会返回最后一个命令的退出码。

Bash 开发人员考虑到了这一点，他们添加了一个特殊的 `PIPESTATUS` 数组，用于保存管道流中所有命令的退出代码。

`PIPESTATUS` 数组的元素对应于命令的退出代码。 下面是一个例子：
```bash
$ echo 'pants are cool' | grep 'moo' | sed 's/o/x/' | awk '{ print $1 }'
$ echo ${PIPESTATUS[@]}
0 1 0 0
```