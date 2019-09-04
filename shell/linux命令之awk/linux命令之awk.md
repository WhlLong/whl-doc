awk是一个强大的文本分析工具，把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行分析处理。

想要用好awk，必须要熟练使用正则表达式。

# 基本用法

awk [选项参数] ‘pattern1{action1}  pattern2{action2}...’ filename

常用选项参数：

| 选项参数 | 功能                                                     |
| -------- | -------------------------------------------------------- |
| -F       | -F fs      fs 指定输入分隔符，fs可以是字符串或正则表达式 |
| -v       | -v var=value 赋值一个用户定义变量，将外部变量传递给awk   |
| -f       | -f scriptfile 从脚本文件中读取awk命令                    |



pattern：表示AWK在数据中查找的内容，就是匹配模式，模式可以是以下任意一种：

- 正则表达式：使用通配符的扩展集
- 关系表达式：使用运算符进行操作，可以是字符串或数字的比较测试
- 模式匹配表达式：用运算符`～`（匹配）和`~!`不匹配
- BEGIN 语句块， pattern语句块， END语句块

action：action由一个或多个命令、函数、表达式组成，之间由换行符或分号隔开，并位于大刮号内，主要部分是：变量或数组赋值、输出命令、内置函数、控制流语句。

awk命令在执行时，先匹配第一个parttern，如果能匹配的上，就执行第一个parttern对应的action,然后再匹配第二个parttern，如果匹配的上，再执行第二个action.....

一个awk脚本通常由`BEGIN， 通用语句块，END语句块组成`，三部分都是可选的。 脚本通常是被**单引号或双引号包住**。基本规则如下：

```
awk 'BEGIN{ commands } pattern{ commands } END{ commands }' file 
```



## 案例

准备数据

```
[root@iz2zegdhs7pd191av8n8dmz cut]# vim data.txt
[root@iz2zegdhs7pd191av8n8dmz cut]# cat data.txt
an hui
shang hai shi
xiang gang te bie xing zheng qu 1
tai wan sheng
xiang gang te bie xing zheng qu 2
```



搜索data.txt中以xiang关键字开头的所有行，并输出该行的第1列和第7列，中间以“，”号分割。且在所有行前面添加列名this is begin,在最后一行添加"this is end"。

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# awk -F " " ' BEGIN{print "this is begin"} /^xiang/{print $1","$7} END{print "this is end"}' data.txt
this is begin
xiang,qu
xiang,qu
this is end
```

**BEGIN 在所有数据读取行之前执行；END 在所有数据执行之后执行。**



# 分隔符

awk中的分隔符分为输入分隔符和输出分隔符两种。

输入分隔符，简称FS,默认是一个空格，awk默认以空格为分隔符对每一行进行分割

输出分隔符简称OFS，默认也是 一个空格。



	## 输入分隔符

设置输入分隔符有两种方式  一种是使用-F来设置，还有一种是使用-v,通过设置内置变量FS来设置分隔符。比如-v FS='*'

案例如下：

```shell
[whl@iz2zegdhs7pd191av8n8dmz ~]$ awk -F: '/^root/{print $0}' /etc/passwd
root:x:0:0:root:/root:/bin/bash

[whl@iz2zegdhs7pd191av8n8dmz ~]$ awk -v FS=':' '/^root/{print $0}' /etc/passwd
root:x:0:0:root:/root:/bin/bash
```

上面这个案例，通过-F和-v 两种方式将分隔符设置为‘：’，然后找出/etc/passwd文件中以root开头的行，并输出



## 输出分隔符

awk在处理每一行文本时，都是以空格作为默认的输出分隔符，输出每一列时，会使用空格为我们隔开每一列。

我们可以通过配置内置变量OFS来设置输出分隔符：

```shell
[whl@iz2zegdhs7pd191av8n8dmz ~]$ awk -F:  -v OFS='*' '/^root/{print $1,$2,$3,$4,$5}' /etc/passwd
root*x*0*0*root
```



上面这个案例通过-F将输入分隔符设置为‘:’,将输出分隔符设置为“*”













# 内置变量

常用的内置变量：

| 变量名   | 功能                                                         |
| -------- | ------------------------------------------------------------ |
| ARGC     | 命令行参数的数目。                                           |
| ARGV     | 包含命令行参数的数组。                                       |
| FILENAME | 当前输入文件的名。                                           |
| NR       | 表示记录数，在执行过程中对应于当前的行号                     |
| FNR      | 同NR :，但相对于当前文件。                                   |
| FS       | 字段分隔符（默认是任何空格）。                               |
| NF       | 表示字段数，在执行过程中对应于当前的字段数。 `print $NF`答应一行中最后一个字段 |
| OFS      | 输出字段分隔符（默认值是一个空格）。                         |
| ORS      | 输出记录分隔符（默认值是一个换行符）。                       |
| RS       | 输入记录分隔符（默认是一个换行符）。                         |

















