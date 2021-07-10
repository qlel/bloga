+++
title = "Python Venv模块"
description = ""
tags = ["python"]
date =  "2021-02-09T01:12:46+08:00"
lastmod = "2021-02-09T01:12:46+08:00"
+++

venv 是python自带的虚拟环境.
<!--more-->

虚拟环境是用于依赖项管理和项目隔离的python工具，它可以将python程序和pip包管理工具安装在本地的隔离目录中（非全局安装）。

在实际开发中，不同项目可能需要的python版本和项目的第三方依赖包的版本不同，因此需要使用到虚拟环境来管理不同的项目。

## 创建虚拟环境
`python -m venv <ENV_DIR>`

查看帮助:
`python -m venv -h`
```bash
usage: venv [-h] [--system-site-packages] [--symlinks | --copies] [--clear] [--upgrade] [--without-pip]
            [--prompt PROMPT] [--upgrade-deps]
            ENV_DIR [ENV_DIR ...]

在一个或多个目标目录中创建虚拟Python环境

位置参数:
  ENV_DIR               要在其中创建环境的目录

可选参数:
  -h, --help            显示帮助信息并退出
  --system-site-packages
                        授予虚拟环境访问系统已安装软件包目录的权限。
  --symlinks            当符号链接不是平台的默认值时，请尝试使用符号链接而不是副本。
  --copies              即使符号链接是平台的默认设置，也请尝试使用副本而不是符号链接。
  --clear               创建环境之前，删除环境目录的内容（如果已经存在）。
  --upgrade             假设Python已升级，升级环境目录以使用此版本的Python。
  --without-pip         跳过在虚拟环境中安装或升级pip（默认情况下，pip是引导的）
  --prompt PROMPT       为此环境提供另一个提示前缀。
  --upgrade-deps        升级核心依赖项：将pip setuptools升级到PyPI中的最新版本

创建环境后，您可能希望激活它，例如 通过在其bin目录中获取激活脚本。
```

示例说明:
	
在`flask_venv`目录创建虚拟环境:
	
`python -m venv .\flask_venv`
```bash
./flask_venv
├── Include
├── Lib
├── pyvenv.cfg
└── Scripts
```
`pyvenv.cfg`文件包含相关环境的信息.

Scripts目录(在POSIX中为bin目录), 包含 Python 二进制文件的副本或符号链接

创建虚拟环境后，可以使用虚拟环境的二进制目录中的脚本来“激活”该环境。不同平台调用的脚本是不同的:
平台|Shell|用于激活虚拟环境的命令
-|-|-
POSIX|bash/zsh|`<venv>/bin/activate`
||fish|`<venv>/bin/activate.fish`
||csh/tcsh|`<venv>/bin/activate.csh`
||PowerShell Core|`<venv>/bin/Activate.ps1`
Windows|cmd.exe|`<venv>\Scripts\activate.bat`
||PowerShell|`<venv>\Scripts\Activate.ps1`

当一个虚拟环境被激活时，`VIRTUAL_ENV` 环境变量会被设为该虚拟环境的路径。 这可被用来检测程序是否运行在虚拟环境中。

>注意:
激活环境不是 必须 的，激活只是将虚拟环境的二进制目录(bin或者Script)添加到搜索路径中，这样 "python" 命令将调用虚拟环境的 Python 解释器，可以运行其中已安装的脚本，而不必输入其完整路径。但是，安装在虚拟环境中的所有脚本都应在不激活的情况下可运行，并自动与虚拟环境的 Python 一起运行。

激活后, 要退出虚拟环境使用: `deactivate` 命令.