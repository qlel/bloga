+++
title = "PowerShell命令Clear-RecycleBin"
date = "2020-07-28T20:13:16+08:00"
tags = ["powershell"]
showLicense = false
+++
**作用：** `Clear-RecycleBin` cmdlet删除计算机回收站中的内容。

此操作类似于使用Windows清空回收站。
<!--more-->

## 参数
- `-Confirm`
运行cmdlet之前提示用户确认。 

即使未指定`-Confirm`参数，也会提示用户进行确认。

---
- `-DriveLetter`
指定要清除单个驱动器号或驱动器号阵列的回收站。

---
- `-Force`
指定不提示用户确认清除回收站。

---
- `-WhatIf`
显示如果运行Clear-RecycleBin将发生什么。 该cmdlet没有运行。

---
## 示例
### 清除所有回收站
```powershell
PS> Clear-RecycleBin

Confirm
Are you sure you want to perform this action?
Performing the operation "Clear-RecycleBin" on target "All of the contents of the Recycle Bin".
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"):
```
---
### 清除指定的回收站
```powershell
PS> Clear-RecycleBin -DriveLetter C
```
Clear-RecycleBin使用DriveLetter参数在C卷上指定回收站。 提示用户确认运行命令。

--- 
### 不需要确认清空所有回收站
```powershell
PS> Clear-RecycleBin -Force
```
`Clear-RecycleBin`使用Force参数，并且不会提示用户进行确认以清除本地计算机上的所有回收站。

一种替代方法是将`-Force`替换为`-Confirm：$false`。
