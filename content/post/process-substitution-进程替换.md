+++
title = "Process Substitution 进程替换"
date = 2021-07-10T14:39:29+08:00
description = ""
tags = ['linux', 'shell']
aplayer = false
showToc = true
math = true
showLicense = false
+++
进程替换是一种重定向形式，其中进程的输入或输出（某些命令序列）作为临时文件出现。

<!--more-->

## 概念

```bash
<(list)

# 或者

>(list)
```
进程列表`list`异步运行，其输入或输出显示为文件名。这个文件名作为参数传递给当前命令。
- `<(list)`作为标准输出文件描述符
- `>(list)`作为标准输入文件描述符

以上文件描述符连接到命名管道FIFO 或 `/dev/fd/` 中的文件。然后将文件名（文件描述符连接的地方）用作 `<(...)` 结构的替代。

进程替换的效果是使每个列表都像一个文件。 这是通过在文件系统中为列表指定一个名称，然后在命令行中替换该名称来完成的。 通过将列表连接到命名管道FIFO或使用 `/dev/fd` 中的文件，可以为列表命名。 通过这样做，命令只会看到一个文件名，而不会意识到它正在从命令管道读取或写入命令管道。

**实现机制：**
进程替换有两种实现方式，在支持`/dev/fd`的系统中，他通过调用`pipe()`系统调用来实现，这个系统调用将为新的匿名管道返回一个文件描述符，然后创建字串`/dev/fd/$fd`，再替换命令行。在不支持`/dev/fd`的系统中，它将调用`mkfifo`命令后跟一个临时文件名来创建一个命名管道FIFO,之后在命令行中替换为这个文件名。

>注意的`<`或`>`和左括号之间是没有空格的，否则将被解释为重定向。

使用进程替换时，根据shell设置，可能会收到类似于以下内容的错误消息：
```bash
syntax error near unexpected token `('
```
进程替换不是 POSIX 兼容功能，因此可能必须通过以下方式启用：
```bash
set +o posix
```

## 示例
```bash
# 这段代码没有用，但它演示了它是如何工作的
# 可以通过读取文件 /dev/fd/63 来访问 ls 程序的输出。
$ echo <(ls)
/dev/fd/63
```
比较每个目录的内容:
```bash
$ diff <(ls "$first_directory") <(ls "$second_directory")
```
在这个命令中，每个进程都替换了一个文件，`diff`没有看到`<(bla)`，它看到了两个文件，所以有效的命令是这样的:
```bash
$ diff /dev/fd/63 /dev/fd/64
```
这些文件会被自动写入并销毁。


### 避免子shell
```bash
counter=0
 
find /etc -print0 | while IFS= read -rd '' _; do
    ((counter++))
done
 
echo "$counter files" # 打印 "0 files"
```
由于管道的原因，`while`语句是执行在子shell中的。这意味着计数器只在子shell中递增。当管道完成时，子shell终止，打印计数器仍处于“0”！

进程替换帮助我们避免管道操作符（子shell的原因）：
```bash
counter=0
 
while IFS= read -rN1 _; do
    ((counter++))
done < <(find /etc -printf ' ')
 
echo "$counter files"
```
### exec中使用
```bash
# 将shell脚本中标准输出和错误输出都写入日志文件中同时在屏幕上显示。
exec &> >(tee "$log_file")
```