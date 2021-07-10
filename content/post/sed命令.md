+++
title = "Sed命令"
description = ""
tags = ['shell', 'linux', 'sed']
date =  "2021-02-28T20:40:20+08:00"
lastmod = "2021-02-28T20:40:20+08:00"
+++
Linux sed命令, 用于处理文本
<!--more-->

`sed`的选项：
- `-e` -- 多项编辑，在使用多个sed命令时使用
- `-n` -- 取消默认的输出，使用安静(silent)模式。在一般 sed 的用法中，所有来自stdin的资料一般都会被列出到屏幕上。但如果加上` -n` 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来
- `-f` -- 指定脚本文件名
- `-i` -- 直接修改读取的文件内容，而不是由屏幕输出
- `-r` -- sed支持延伸型正则表达式的语法，加-r后 可以换掉不用`\`

`sed`的功能：
参数|说明
--|--
`a`|	插入，当前行后添加一行或多行。多行需在每行最后加“\”
`c`|	替换，把当前行中的文本替换成符号后的新文本。多行需在每行最后加“\”
`s`|	替换，用一个字符串替换另一个。通常和正则表达式搭配使用：5s/旧命令/新命令/g ，5表示行数，默认全行，g是全局
`g`|	全局替换
`d`|	删除行
`i`|	插入，在当前行之前插入文本，多行需在每行最后加“\”
`p`|	打印行
`q`|	结束，退出sed
`r`|	从文件中读取输入行
`w`|	将所选的行追加写入文件末尾
`x`|	交换暂存缓冲区与模式空间的内容
`!`|	取反，对所选行以外的行生效命令
`y`|	将字符替换成另一字符（不支持正则）
`h`|	copy模板块的内容粘贴到内存中的缓冲区
`l`|	列出非打印字符
`n`|	直接读下一行，并从下一条命令处理

***
操作的示例文件：`con.txt`
```bash {.line-numbers}
USSR    8649    275     Asia
Canada  3852    25      North America
China   3705    1032    Asia
USA     3615    237     North America
Brazil  3286    134     South America
India   1267    746     Asia
Mexico  762     78      North America
France  211     55      Europe
Japan   144     120     Asia
Germany 96      61      Europe
England 94      56      Europe
```
- 语法1：
```bash
sed [options] {sed-commands} {input-file}

# -n表示取消默认输出,p表示打印行
# 打印所有行
sed -n 'p' con.txt 
# 打印第3行
sed -n '3p' con.txt
# 打印第2到第5行的所有行
sed -n '2,5p' con.txt
```
![语法示例1](/sed/1.png)
***
- 语法2：将`sed`命令写入到一个文件中，通过`-f`执行这个文件
```bash
sed [options] -f {sed-commands-in-a-file} {input-file}

# -f sed的脚本文件 
cat sed_con.txt
/^China/ p
/^Japan/ p
# 打印以China和Japan开头的行
sed -n -f sed_con.txt con.txt
```
![语法示例2](/sed/2.png)
***
- 语法3：
```bash
sed [options] -e {sed-command-1} -e {sed-command-2} {input-file}

# 打印以China和Japan开头的行
sed -n -e '/^China/ p' -e '/^Japan/ p' con.txt

# 或者
sed -n \
-e '/^China/ p' \
-e '/^Japan/ p' \
con.txt
```
***
- 语法4：
```bash
sed [options] '{
sed-command-1
sed-command-2
}' input-file

# 打印以China和Japan开头的行
[~/awk_sed]$ sed -n '{
quote> /^China/ p
quote> /^Japan/ p
quote> }' con.txt
China   3705    1032    Asia
Japan   144     120     Asia
```
## 基本示例
示例文件：`test_sed.txt`
```bash {.line-numbers}
101,Ian Bicking,Mozilla
102,Hakim El Hattab,Whim
103,Paul Irish,Google
104,Addy Osmani,Google
105,Chris Wanstrath,Github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware
```
### 范围
```bash
# 从第 1 行开始，每隔 2 行读取行
[~/awk_sed]$ sed -n '1~2 p' test_sed.txt
101,Ian Bicking,Mozilla
103,Paul Irish,Google
105,Chris Wanstrath,Github
107,Ask Solem Hoel,VMware

# 从第 2 行开始，每隔 3 行读取行
[~/awk_sed]$ sed -n '2~3 p' test_sed.txt
102,Hakim El Hattab,Whim
105,Chris Wanstrath,Github
```
### 模式匹配
```bash
# 寻找包含Paul的行
[~/awk_sed]$ sed -n '/Paul/ p' test_sed.txt
103,Paul Irish,Google

# 从第一行开始到第五行, 从找到开始打印到第五行
[~/awk_sed]$ sed -n '/Paul/,5 p' test_sed.txt
103,Paul Irish,Google
104,Addy Osmani,Google
105,Chris Wanstrath,Github

# 从匹配Paul行打印达匹配Addy的行
[~/awk_sed]$ sed -n '/Paul/,/Addy/ p' test_sed.txt
103,Paul Irish,Google
104,Addy Osmani,Google

# 匹配Paul行再多输出2行
[~/awk_sed]$ sed -n '/Paul/,+2 p' test_sed.txt
103,Paul Irish,Google
104,Addy Osmani,Google
105,Chris Wanstrath,Github
```
### 删除行
>注意：如果没有加`-i`参数，默认是不会修改源文件的。
```bash
# 删除所有行
[~/awk_sed]$ sed 'd' test_sed.txt

# 只删除第二行
[~/awk_sed]$ sed '2 d' test_sed.txt
101,Ian Bicking,Mozilla
103,Paul Irish,Google
104,Addy Osmani,Google
105,Chris Wanstrath,Github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware

# 删除第一到第四行
[~/awk_sed]$ sed '1,4 d' test_sed.txt
105,Chris Wanstrath,Github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware

# 从第 1 行开始，每隔 2 行删除行
[~/awk_sed]$ sed '1~2 d' test_sed.txt
102,Hakim El Hattab,Whim
104,Addy Osmani,Google
106,Mattt Thompson,Heroku

# 删除符合Paul到Addy的行
[~/awk_sed]$ sed '/Paul/,/Addy/ d' test_sed.txt
101,Ian Bicking,Mozilla
102,Hakim El Hattab,Whim
105,Chris Wanstrath,Github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware

# 删除空行
[~/awk_sed]$ sed '/^$/ d' test_sed.txt
101,Ian Bicking,Mozilla
102,Hakim El Hattab,Whim
103,Paul Irish,Google
104,Addy Osmani,Google
105,Chris Wanstrath,Github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware
```
### 重定向
```bash
# 将source.txt内容重定向写到output.txt
$sed 'w output.txt' source.txt
# 和上面一样,但是没有在终端显示
$sed -n 'w output.txt' source.txt
# 只写第二行
$ sed -n '2 w output.txt' source.txt
# 写一到四行到output.txt
$sed -n '1,4 w output.txt'
# 写匹配Ask的行到结尾行到output.txt
$sed -n '/Ask/,$ w output.txt'
```
### 替换
格式：
```bash
$sed '[匹配模式] s/原str/替str/[标志位]' 文件
```
示例：
```bash
# 替换Google为Github
$sed 's/Google/Github/' test_sed.txt
101,Ian Bicking,Mozilla
102,Hakim El Hattab,Whim
103,Paul Irish,Github
104,Addy Osmani,Github
105,Chris Wanstrath,Github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware

# 替换匹配Addy的行里面的Google为Github
$sed '/Addy/s/Google/Github/' test_sed.txt
101,Ian Bicking,Mozilla
102,Hakim El Hattab,Whim
103,Paul Irish,Google
104,Addy Osmani,Github
105,Chris Wanstrath,Github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware

# 默认s只会替换第1行中的第1个匹配项
$sed '1s/a/A/' test_sed.txt | head -1
101,IAn Bicking,Mozilla

# 替换第2行中的第3个匹配项
$ sed '2s/a/A/3' sed_test.txt
101,Ian Bicking,Mozilla
102,Hakim El HattAb,Whim
103,Paul Irish,Google
104,Addy Osmani,Google
105,Chris Wanstrath,Github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware

# 加标志位 g 可以替换每行的全部符合
$sed '1s/a/A/g' test_sed.txt | head -1
101,IAn Bicking,MozillA

# 可以直接指定想要替换的第N个匹配项,这里是第 2 个
$sed '1s/a/A/2' test_sed.txt | head -1
101,Ian Bicking,MozillA

# 使用w将能够替换的行重定向写到output.txt
$sed -n 's/Mozilla/Github/w output.txt' test_sed.txt
$cat output.txt
101,Ian Bicking,Github

# 使用 i 忽略匹配的大小写
$sed '1s/ian/IAN/i' test_sed.txt | head -1
101,IAN Bicking,Mozilla

# sed分隔符不只可以使用'/'
$sed 's|/usr/local/bin|/usr/bin|' path.txt
$sed 's^/usr/local/bin^/usr/bin^' path.txt
$sed 's@/usr/local/bin@/usr/bin@' path.txt

# 此处有一个新文件
$cat files.txt 
/etc/passwd
/etc/group
# 给每行前和后都添加点字符
$sed 's/\(.*\)/ls -l \1/' files.txt
ls -l /etc/passwd
ls -l /etc/group
# \1 表示前前面第1个括号中匹配的内容，以此类推

# 替换覆盖
sed '{
s/Google/Github/
s/Git/git/ 
}' source.txt 
101,Ian Bicking,Mozilla
102,Hakim El Hattab,Whim
103,Paul Irish,github
104,Addy Osmani,github
105,Chris Wanstrath,github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware

# & 表示已匹配的字符串标记
$sed 's/[0-9][0-9][0-9]/[&]/' test_sed.txt
[101],Ian Bicking,Mozilla
[102],Hakim El Hattab,Whim
[103],Paul Irish,Google
[104],Addy Osmani,Google
[105],Chris Wanstrath,Github
[106],Mattt Thompson,Heroku
[107],Ask Solem Hoel,VMware
```
***
### 插入行与修改行
```bash
# 行后插入 a
# 在第2行后插入
$sed '2 a 108,test test,test' test_sed.txt
101,Ian Bicking,Mozilla
102,Hakim El Hattab,Whim
108,test test,test
103,Paul Irish,Google
104,Addy Osmani,Google
105,Chris Wanstrath,Github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware

# 行前插入 i
# 在第2行前插入
$sed '2 i 108,test test,test' test_sed.txt
101,Ian Bicking,Mozilla
108,test test,test
102,Hakim El Hattab,Whim
103,Paul Irish,Google
104,Addy Osmani,Google
105,Chris Wanstrath,Github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware

# 修改第2行 c
$sed '2 c 108,test test,test' test_sed.txt
101,Ian Bicking,Mozilla
108,test test,test
103,Paul Irish,Google
104,Addy Osmani,Google
105,Chris Wanstrath,Github
106,Mattt Thompson,Heroku
107,Ask Solem Hoel,VMware
```
***
### 其它
```bash
# = 可以显示行号
$sed = test_sed.txt
1
101,Ian Bicking,Mozilla
2
102,Hakim El Hattab,Whim
3
103,Paul Irish,Google
4
104,Addy Osmani,Google
5
105,Chris Wanstrath,Github
6
106,Mattt Thompson,Heroku
7
107,Ask Solem Hoel,VMware
```
***
## 高级
示例文件：`t2.txt`
```bash {.line-numbers}
Ian Bicking
Mozilla
Hakim El Hattab
Whim
Paul Irish
Google
Chris Wanstrath
Github
Mattt Thompson
Heroku	
```

### 运行模式
当处理数据时，Sed 从输入源一次读入一行，并将它保存到所谓的**模式空间**(pattern space)中。所有 Sed 的变换都发生在模式空间。**变换**都是由命令行上或外部 Sed 脚本文件提供的单字母命令来描述的。大多数 Sed 命令都可以由一个地址或一个地址范围作为前导来限制它们的作用范围。

默认情况下，Sed 在结束每个处理循环后输出模式空间中的内容，也就是说，输出发生在输入的下一个行覆盖模式空间之前。我们可以将这种运行模式总结如下：
1. 尝试将下一个行读入到模式空间中
2. 如果读取成功：
    1. 按脚本中的顺序将所有命令应用到与那个地址匹配的当前输入行上
    2. 如果 sed 没有以静默模式（`-n`）运行，那么将输出模式空间中的所有内容（可能会是修改过的）。
    3. 重新回到 1。

因此，在每个行被处理完毕之后，模式空间中的内容将被丢弃，它并不适合长时间保存内容。基于这种目的，Sed 有第二个缓冲区：**保持空间**(hold space)。除非你显式地要求它将数据置入到保持空间、或从保持空间中取得数据，否则 Sed 从不清除保持空间的内容。