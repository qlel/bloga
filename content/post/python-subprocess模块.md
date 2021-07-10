+++
title = "Python Subprocess模块"
description = ""
tags = ["python"]
date =  "2021-02-09T01:09:57+08:00"
lastmod = "2021-02-09T01:09:57+08:00"
+++

subprocess 子进程管理

生成新的进程，连接它们的输入、输出、错误管道，并且获取它们的返回码。
<!--more-->

## run()
subprocess 模块首先推荐使用的是它的 run 方法，更高级的用法可以直接使用底层的 Popen 接口。

`subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None, capture_output=False, shell=False, cwd=None, timeout=None, check=False, encoding=None, errors=None, text=None, env=None, universal_newlines=None, **other_popen_kwargs)`

运行被 args 描述的指令。等待指令完成，然后返回一个 `CompletedProcess` 实例。

- `args`可以是一个字符串或者列表序列
- `stdin`,`stdout`,`stderr`: 执行程序命令的标准输入、输出和错误。
其值可以是 `subprocess.PIPE`、`subprocess.DEVNULL`、一个已经存在的文件描述符、已经打开的文件对象或者 `None`。`subprocess.PIPE` 表示为子进程创建新的管道。`subprocess.DEVNULL` 表示使用 `os.devnull`。默认使用的是 None，表示什么都不做。另外，stderr 可以合并到 stdout 里一起输出。
- `capture_output`: `stderr`和`stdout`的合并, 如果设置为True, stdout 和 stderr 将会被捕获, 即:`stdout=subprocess.PIPE`,`stderr=subprocess.PIPE`
- `timeout`: 设置命令超时时间。如果命令执行时间超时，子进程将被杀死，并弹出 TimeoutExpired 异常。
- `check`: 如果该参数设置为 True，并且进程退出状态码returncode不是 0，则弹 出 CalledProcessError 异常。
- `text`: 如果为True, 则 stdin、stdout 和 stderr 可以接收字符串数据, 否则只接收 bytes 类型的数据。
- `shell`: 如果该参数为 True，将通过操作系统的 shell 执行指定的命令。在win10中必须设置为True才会执行命令.
在 Windows，使用 shell=True，环境变量 COMSPEC 指定了默认 shell。在 Windows 你唯一需要指定 shell=True 的情况是你想要执行内置在 shell 中的命令（例如 dir 或者 copy）。在运行一个批处理文件或者基于控制台的可执行文件时，不需要 shell=True。
- `cwd`: 在执行子进程前会将当前工作目录改为 cwd。 cwd 可以是一个字符串、字节串或 路径类对象 
- `env`: 用于指定子进程的环境变量。如果 env = None，子进程的环境变量将从父进程中继承。字典.

返回的`CompletedProcess`实例属性和方法有:
- `args`: 运行的指令
- `returncode`: 子进程的退出状态码. 通常来说, 一个为 0 的退出码表示进程运行正常. 
一个负值 -N 表示子进程被信号 N 中断 (仅 POSIX).
- `stdout`, `stderr`: 从子进程捕获到的标准输出, 错误. 一个字节序列.
如果 `run()` 是设置了 encoding, errors 或者 text=True 来运行的, 则可以是字符串
如果未有捕获, 则为 None.
- `check_returncode()`: 为实例方法, 如果 returncode 非零, 抛出 CalledProcessError.

示例:
```py
>>> subprocess.run(args='ls', shell=True)
CompletedProcess(args='ls', returncode=1)
>>> subprocess.run(args='ls', shell=True, capture_output=True)
CompletedProcess(args='ls', returncode=1, stdout=b'', stderr=b"'ls' \xb2\xbb\xca\xc7\xc4\xda\xb2\xbf\xbb\xf2\xcd\xe2\xb2\xbf\xc3\xfc\xc1\xee\xa3\xac\xd2\xb2\xb2\xbb\xca\xc7\xbf\xc9\xd4\xcb\xd0\xd0\xb5\xc4\xb3\xcc\xd0\xf2\r\n\xbb\xf2\xc5\xfa\xb4\xa6\xc0\xed\xce\xc4\xbc\xfe\xa1\xa3\r\n")

# text=True 会转化输出为字符串类型, win10中的dos没有ls命令
>>> subprocess.run(args='ls', shell=True, capture_output=True, text=True)
CompletedProcess(args='ls', returncode=1, stdout='', stderr="'ls' 不是内部或外部命令，也不是可运行的程序\n或批处理文件。\n")

>>> subprocess.run(args=['echo',r'你好'], shell=True, capture_output=True, text=True)
CompletedProcess(args=['echo', '你好'], returncode=0, stdout='你好\n', stderr='')
```

## Popen()
`subprocess.Popen(args, bufsize=-1, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=True, shell=False, cwd=None, env=None, universal_newlines=None, startupinfo=None, creationflags=0, restore_signals=True, start_new_session=False, pass_fds=(), *, group=None, extra_groups=None, user=None, umask=-1, encoding=None, errors=None, text=None)`

在一个新的进程中执行子程序。返回Popen实例.

在 POSIX，此类使用类似于 `os.execvp()` 的行为来执行子程序。

在 Windows，此类使用了 `Windows CreateProcess()` 函数

参数与`run()`类似. 还有些其它参数
- `bufsize`: 缓冲区大小。当创建标准流的管道对象时使用，默认-1。
0表示不使用缓冲区.
1表示行缓冲，仅当universal_newlines=True时可用，也就是文本模式
正数：表示缓冲区大小
负数：表示使用系统默认的缓冲区大小。
- `preexec_fn`：只在 Unix 平台下有效，用于指定一个可执行对象（callable object），它将在子进程运行之前被调用

Popen 对象支持通过 with 语句作为上下文管理器，在退出时关闭文件描述符并等待进程:
```py
with Popen(["ifconfig"], stdout=PIPE) as proc:
    log.write(proc.stdout.read())
```

Popen实例对象方法:
- `poll()`
检查进程是否终止，如果终止返回 returncode，否则返回 None。
- `wait(timeout=None)`
等待子进程被终止。
- `communicate(input=None, timeout=None)`
和子进程交互，发送和读取数据。
- `send_signal(signal)`
将信号 signal 发送给子进程
- `terminate()`
停止子进程。 在 POSIX 操作系统上，此方法会发送 SIGTERM 给子进程。 在 Windows 上则会调用 Win32 API 函数 TerminateProcess() 来停止子进程。
- `kill()`
杀死子进程。 在 POSIX 操作系统上，此函数会发送 SIGKILL 给子进程。 在 Windows 上 kill() 则是 terminate() 的别名。

Popen实例对象属性:
- `args`: 指令
- `stdin`, `stdout`, `stderr`: 
如果 stdin 或 stdout 或 stderr 参数为 PIPE，此属性是一个类似 open() 返回的可写的流对象。如果 encoding 或 errors 参数被指定或者 universal_newlines 参数为 True，则此流是一个文本流，否则是字节流。如果 stdin 参数非 PIPE， 此属性为 None。
- `pid`
子进程的进程号。
- `returncode`
子进程的退出码

