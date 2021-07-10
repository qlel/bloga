+++
title = "PowerShell命令Get-Acl"
date = "2020-07-28T20:13:16+08:00"
tags = ["powershell"]
showLicense = false
+++
获取资源（例如文件或注册表项）的**安全描述符**。

权限相关。
<!--more-->

## 描述
`Get-Acl` 获取代表文件或资源的**安全描述符**的对象。

安全描述符包含资源的访问控制列表（ACL）。 

ACL指定用户和用户组访问资源所必须具有的权限。

## 参数
参数列表|说明
--|--
`-Audit`|从系统访问控制列表（SACL）获取安全描述符的审核数据。
`-Exclude`|忽略指定的项目。 
`-Filter`|用提供程序的格式或语言指定一个过滤器。
`-Include`|只获取指定的项目。
`-InputObject`|获取指定对象的安全描述符。
`-LiteralPath`|指定资源的路径。
`-Path`|指定资源的路径。

---
`-Exclude`
忽略指定的项目。 
此参数的值限定**Path**参数。
输入路径元素或模式，例如`*.txt`。 允许使用通配符。

---
`-Filter`
用提供程序的格式或语言指定一个过滤器。
过滤器的语法（包括通配符的使用）取决于提供程序。
过滤器比其他参数更有效率，因为提供程序在获取对象时会应用它们，而不是让PowerShell在检索对象后对其进行过滤。
此参数的值限定**Path**参数。

---
`-Include`
只获取指定的项目。
此参数的值限定**Path**参数。
输入路径元素或模式，例如`*.txt`。 允许使用通配符。

---
`-InputObject`
获取指定对象的安全描述符。
输入包含该对象的变量或获取该对象的命令。
你不能将除路径以外的对象通过管道传递到`Get-Acl`。
而是在命令中显式使用`InputObject`参数。

---
`-LiteralPath`
指定资源的路径。
指定资源的路径时，所有的特殊字符被视为路径片段。
与Path不同的是，LiteralPath参数的值是按照键入时的样子使用的。
没有字符被解释为通配符。
如果路径包括转义字符，请用单引号括起来。单引号告诉PowerShell不要将任何字符解释为转义序列。

如：`Get-ChildItem` 默认的参数为`-Path`。假如你当前文件夹下有个文件名为`.\a[0].txt`，因为方括号是PowerShell中的特殊字符，会解释器被解析。为了能正确获取到`.\a[0].txt`的文件信息，此时可以使用`-LiteralPath`参数，它会把你传进来的值当作纯文本。

```powershell
PS> Get-ChildItem .\a[0].txt
PS> Get-ChildItem -Path .\a[0].txt
PS> Get-ChildItem -LiteralPath .\a[0].txt

    Directory: C:\Users\mosser

Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---          2014/1/2     14:04      80370 a[0].txt
```

---
`-Path`
指定资源的路径。
`Get-Acl`获取路径指示的资源的安全描述符。允许使用通配符。
如果省略Path参数，则`Get-Acl`将获取当前目录的安全描述符。

---
## 示例
### 获取一个目录的ACL

```powershell
PS> Get-Acl C:\Windows


    Directory: C:\

Path    Owner                       Access
----    -----                       ------
Windows NT SERVICE\TrustedInstaller CREATOR OWNER Allow  268435456…
```
### 获取ACL的详细信息

```powershell
PS> Get-Acl C:\Windows | Format-List

Path   : Microsoft.PowerShell.Core\FileSystem::C:\Windows
Owner  : NT SERVICE\TrustedInstaller
Group  : NT SERVICE\TrustedInstaller
Access : CREATOR OWNER Allow  268435456
         NT AUTHORITY\SYSTEM Allow  268435456
         NT AUTHORITY\SYSTEM Allow  Modify, Synchronize
         BUILTIN\Administrators Allow  268435456
         BUILTIN\Administrators Allow  Modify, Synchronize
         BUILTIN\Users Allow  -1610612736
         BUILTIN\Users Allow  ReadAndExecute, Synchronize
         NT SERVICE\TrustedInstaller Allow  268435456
         NT SERVICE\TrustedInstaller Allow  FullControl
         APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES Allow  ReadAndExecute, Synchronize
         APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES Allow  -1610612736
         APPLICATION PACKAGE AUTHORITY\所有受限制的应用程序包 Allow  ReadAndExecute, Synchronize
         APPLICATION PACKAGE AUTHORITY\所有受限制的应用程序包 Allow  -1610612736
Audit  :
Sddl   : O:S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464G:S-1-5-80-956008885-3418522649-1831038044-185
         3292631-2271478464D:PAI(A;OICIIO;GA;;;CO)(A;OICIIO;GA;;;SY)(A;;0x1301bf;;;SY)(A;OICIIO;GA;;;BA)(A;;0x1301bf;;;
         BA)(A;OICIIO;GXGR;;;BU)(A;;0x1200a9;;;BU)(A;CIIO;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271
         478464)(A;;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;;0x1200a9;;;AC)(A;OICIIO;GXG
         R;;;AC)(A;;0x1200a9;;;S-1-15-2-2)(A;OICIIO;GXGR;;;S-1-15-2-2)
```
### 使用通配符获取目录的ACL
```powershell
PS> Get-Acl C:\Windows\s*.log | Format-List -Property PSPath, Sddl

PSPath : Microsoft.PowerShell.Core\FileSystem::C:\Windows\setuperr.log
Sddl   : O:SYG:SYD:AI(A;ID;FA;;;SY)(A;ID;FA;;;BA)(A;ID;0x1200a9;;;BU)(A;ID;0x1200a9;;;AC)(A;ID;0x1200a9;;;S-1-15-2-2)
```
### 获取注册表项的ACL
```powershell
Get-Acl -Path HKLM:\System\CurrentControlSet\Control | Format-List
```