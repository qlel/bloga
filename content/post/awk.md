+++
title = "Awk"
description = ""
tags = ['linux', 'shell', 'awk']
date =  "2021-04-11T16:26:20+08:00"
lastmod = "2021-04-11T16:26:20+08:00"
+++

awk有多个不同版本的演化：awk->nawk->mawk->gawk
<!--more-->

## awk选项

- `-F value`
设置字段分隔符
- `-f file`
从file文件中读取模式动作。允许使用多个`-f`
- `-v var=value`
设置变量var的值为value
- `--`
表示选项的明确结尾

## 快速入门
基本模式：
- `awk '匹配 {动作}' 数据文件` 

示例：数据文件`employee.txt`
```bash
Beth 4.00 0
Dan 3.75 0
Kathy 4.00 10
Mark 5.00 20
Mary 5.50 22
Susie 4.25 18
```
操作：
```bash
# 打印第3个字段(列)
$ awk '{print $3}' employee.txt
0
0
10
20
22
18

# 匹配第3列大于0的行，然后打印匹配行的第1列和第2，3列的乘积
$ awk '$3 > 0 { print "数据:"$1, $2 * $3 }' employee.txt
数据:Kathy 40
数据:Mark 100
数据:Mary 121
数据:Susie 76.5
```
## 模式汇总
- `BEGIN{statements}`
在输入被读取之前, statements 执行一次
- `END{ statements}`
当所有输入被读取完毕之后, statements 执行一次
- `expression{ statements}`
每碰到一个使 expression 为真的输入行, statements 就执行. expression 为真指的是其值非零或非空.
- `/regular expression/ { statements}`
可以被 regular expression 正则匹配的输入行执行statements
- `compound pattern { statements}`
使用逻辑与`&&`、逻辑或`||`、逻辑非`!`组合的表达式为混合模式
- `pattern1, pattern2 { statements}`
多个输入行的范围匹配。从匹配pattern1的输入行开始，到匹配pattern2的输入行结束(包括匹配的这两行)，对这其中的每一行执行 statements

示例：
```bash
# 在输入被读取前进行的操作，print "" 表示打印一空行
$ awk 'BEGIN{print "AA BB CC";print ""} {print $0}' employee.txt
AA BB CC

Beth 4.00 0
Dan 3.75 0
Kathy 4.00 10
Mark 5.00 20
Mary 5.50 22
Susie 4.25 18

# 设置分隔符为'a'
$ awk 'BEGIN{FS="a"} {print $1}' employee.txt
Beth 4.00 0
D
K
M
M
Susie 4.25 18

# 范围匹配
$ awk '$1=="Dan", $3==22 {print $0}' employee.txt
Dan 3.75 0
Kathy 4.00 10
Mark 5.00 20
Mary 5.50 22

# 正则
$ awk '$1 ~ /Mar/ {print $0}' employee.txt
Mark 5.00 20
Mary 5.50 22

# 混合模式，逻辑与
$ awk '$3>0 && $3==20 {print $0}' employee.txt
Mark 5.00 20
```

## 输出
`print`和`printf`语句可以用于输出。

输出语句形式汇总：
- `print`
将 `$0` 打印到标准输出
- `print 表达式1,表达式2...`
输出表达式之间由内建变量OFS和ORS分隔
- `print 表达式1,表达式2... > filename`
输出至文件 filename
- `print 表达式1,表达式2... >> filename`
累加输出到文件 filename, 不覆盖之前的内容
- `print 表达式1,表达式2... | command`
输出至管道，作为命令 command 的输入
- `printf(格式,表达式1,表达式2,…)`
- `printf(格式,表达式1,表达式2,…) > filename`
- `printf(格式,表达式1,表达式2,…) >> filename`
- `printf(格式,表达式1,表达式2,…) | command`
printf 类似于 print, 但是第 1 个参数规定了输出的格式
- `close(filename)`, `close(command)`
断开 filename 或 command 与输出语句之间的连接
- `system(command)`
执行 command; 函数的返回值是 command 的退出状态

print 语句形式还可以有：`print(表达式1,表达式2...)`

printf 的格式与C语言的格式一致：
格式字符|	意义
-|-
`d`|	以十进制形式输出带符号整数(正数不输出符号)
`o`|	以八进制形式输出无符号整数(不输出前缀0)
`x,X`|	以十六进制形式输出无符号整数(不输出前缀Ox)
`u`|	以十进制形式输出无符号整数
`f`|	以小数形式输出单、双精度实数
`e,E`|	以指数形式输出单、双精度实数
`g,G`|	以%f或%e中较短的输出宽度输出单、双精度实数
`c`|	输出单个字符
`s`|	输出字符串

- `-`标识符表示左对齐，默认右对齐
- `width.prec`：width表示宽度，prec表示精度。为了达到规定的宽度, 必要时填充空格; 前导的 0 表示用零填充。

printf格式示例：
格式|示例数据|printf输出
-|-|-
`%c`|97|a
`%d`|97.5|97
`%5d`|97.5|`此处有3个空格`97
`%e`|97.5|9.750000e+01
`%f`|97.5|97.500000
`%7.2f`|97.5|`此处有2个空格`97.50
`%g`|97.5|97.5
`%.6g`|97.5|97.5
`%o`|97|141
`%06o`|97|000141
`%x`|97|61
`%s`|January|January
`%10s`|January|`此处有3个空格`January
`%-10s`|January|January`此处有3个空格`
`%.3s`|January|Jan
`%10.3s`|January|`此处有7个空格`Jan
`%-10.3s`|January|Jan`此处有7个空格`

示例：
```bash
$ awk '$3==20 {printf("%s\n",$1)}' ../employee.txt
Mark

# 输出到文件
$ awk '$3==20 {printf("%s\n",$1) > "../aa.txt"}' ../employee.txt
$ cat ../aa.txt
Mark

# 输出到管道
$ awk '$3!=0 {print $3 | "sort -r"}' employee.txt
22
20
18
10
```

## 内建变量
只列出mawk的内建变量。
变量|说明
-|-
`$n`|当前记录的第n个字段，字段间由FS分隔
`$0`|完整的输入记录
`ARGC`|命令行参数的数目
`ARGV`|包含命令行参数的数组
`CONVFMT`|数字转换格式(默认值为%.6g)ENVIRON环境变量关联数组
`ENVIRON`|由环境变量索引的数组。 环境字符串var=value存储为ENVIRON[var]=value。
`FILENAME`|当前文件名
`FNR`|各文件分别计数的行号
`FS`|字段分隔符(默认是任何空格)
`NF`|一条记录的字段的数目
`NR`|已经读出的记录数，就是行号，从1开始
`OFMT`|数字的输出格式(默认值是%.6g)
`OFS`|输出字段分隔符，默认值与输入字段分隔符一致。
`ORS`|输出记录分隔符(默认值是一个换行符'\n')
`RLENGTH`|由match函数所匹配的字符串的长度
`RS`|记录分隔符(默认是一个换行符'\n')
`RSTART`|由match函数所匹配的字符串的第一个位置索引
`SUBSEP`|数组下标分隔符(默认值是/034)

示例：
```bash
# 每条记录的字段数量
$ awk '$3==0 {print NF}' employee.txt
3
3

# 当前文件名
$ awk '{print FILENAME}' employee.txt
employee.txt
employee.txt
employee.txt
employee.txt
employee.txt
employee.txt
```
## 内置函数
https://www.runoob.com/w3cnote/awk-built-in-functions.html

## 运算符
只列出mawk的运算符：
运算符|说明
-|-
`=`, `+=`, `-=`, `*=`, `/=` ,`%=`, `^=`|赋值
`?:`|条件，三目运算符
`||`|逻辑或
`&&`|逻辑与
`!`|逻辑非
`in`|数组成员操作符
`~` 与 `!~`|匹配 与 不匹配 正则表达式
`<`, `>`, `<=`, `>=`, `==`, `!=`|关系运算符
空格|连接符
`+`, `-`|加，减
`*`, `/`, `%`|乘，除，求余
`+`, `-`|一元加，一元减
`^`|求幂
`++`, `--`|增加或减少，作为前缀或后缀
`$`|字段引用

示例：
```bash
# 赋值
$ awk '$3>0 {print $3*=2}' employee.txt
20
40
44
36

# 三目条件运算
$ awk '$3>0 {print $3==20?$1:$2}' employee.txt
4.00
Mark
5.50
4.25

# 逻辑与
$ awk '$3>0 && $3==20 {print $0}' employee.txt
Mark 5.00 20

# 正则
$ awk '$1 ~ /Mar/ {print $0}' employee.txt
Mark 5.00 20
Mary 5.50 22
```

## 正则表达式
mawk使用的是egrep同款的扩展正则表达式EREs。

https://www.cnblogs.com/chengmo/archive/2010/10/10/1847287.html

## 流程控制语句
Awk 提供了用于决策的 if-else 语句, 以及循环语句, 所有的这些都来源于 C 语言. 它们只能用在动作(Action) 里.

```c
// if-else
if ( expr ) statement
if ( expr ) statement else statement

// while
while ( expr ) statement

do statement while ( expr )

// for
for ( opt_expr ; opt_expr ; opt_expr ) statement

for ( var in array ) statement

// 结束单个循环
continue

// 结束循环
break
```

## 数组
Awk提供一维数组。

数组元素表示为`Array[expr]`。`expr`在内部转换为字符串类型，因此，例如，`A[1]`和`A["1"]`是同一个元素，实际索引是`"1"`。由字符串索引的数组称为关联数组。如果引用的数组不存在，会自动创建空白数组。

```bash
$ awk 'BEGIN{arr[0]="a";arr[1]="b";arr[2]="c";print arr[2]}'
c
```
遍历数组：
- 如果索引是数字字符串，可以使用for或者for-in循环遍历数组；
- 如果索引是字符串而不是数字字符串，则只能使用for-in循环，此循环是乱序的；

```bash
# for
$ awk 'BEGIN{arr[0]="a";arr[1]="b";arr[2]="c";for(i=0;i<3;i++) print i,arr[i]}'
0 a
1 b
2 c

# for...in 乱序
$ awk 'BEGIN{arr[0]="a";arr[1]="b";arr[2]="c";for(i in arr) print i,arr[i]}'
0 a
1 b
2 c
```
删除数组：
- 删除单个数组元素：`delete arr[index]`
- 删除所有数组元素：`delete arr`

```bash
# 删除某个数组元素
$ awk 'BEGIN{arr[0]="a";arr[1]="b";arr[2]="c"; \
> delete arr[1];for(i in arr) print i,arr[i]}'
0 a
2 c

# 清空数组arr
$ awk 'BEGIN{arr[0]="a";arr[1]="b";arr[2]="c"; \
delete arr;for(i in arr) print i,arr[i]}'
```

## 自定义函数与脚本
自定义函数：`function name( args ) { statements }`，可以有返回值：`return opt_expr`。

可以像写shell一样写awk脚本，写完后赋予执行权限即可执行，如：
```bash
#!/usr/bin/awk -f

function add_salary(num1,num2){
        salary_sum=num1+num2
        return salary_sum
}

BEGIN{
        print "hello func01"
        print add_salary(10,20)
}
```
执行结果：
```bash
hello func01
30
```
脚本也可以与要处理的数据文件一起使用：
```bash
#!/usr/bin/awk -f

function add_salary(num1,num2){
        salary_sum=num1+num2
        return salary_sum
}

BEGIN{
        print "hello func01"
        print add_salary(10,20)
}

$1 ~ /Mark/ { print add_salary($2,$3) }
```
执行：
```bash
$ ./func01.awk employee.txt
hello func01
30
25
```