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
| NF       | 表示字段数，在执行过程中对应于当前的字段数。 `print $NF`打印一行中最后一个字段 |
| FS       | 输入字段列分隔符（默认是任何空格）。                         |
| OFS      | 输出字段分隔符（默认值是一个空格）。                         |
| ORS      | 输出记录行分隔符（默认值是一个换行符）。                     |
| RS       | 输入记录行分隔符（默认是一个换行符）。                       |





## NR和NF

NR代表记录数，表示每一行的行号，NF表示每一行有几列

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# vim info1.txt
[root@iz2zegdhs7pd191av8n8dmz awk]# cat info1.txt
118 tery 35ygr 4653tg 
346t 35t34eqe y6y  8i33 653
24e2 24r2 ygh fds gfd ghe u7u
[root@iz2zegdhs7pd191av8n8dmz awk]# awk '{print NR,NF}' info1.txt
1 4
2 5
3 7
```

info1.txt这个文件中一共有三行数据，通过空格隔开，第一行有四列，第二行有五列，第三行有七列。



## FNR

FNR也是用来显示行号的。

但是如果awk同时处理多个文件，那么情况如下：

```
[root@iz2zegdhs7pd191av8n8dmz awk]# vim info1.txt
[root@iz2zegdhs7pd191av8n8dmz awk]# cat info1.txt
118 tery 35ygr 4653tg 
346t 35t34eqe y6y  8i33 653
24e2 24r2 ygh fds gfd ghe u7u

[root@iz2zegdhs7pd191av8n8dmz awk]# vim info2.txt
[root@iz2zegdhs7pd191av8n8dmz awk]# cat info2.txt
e3rty eete rte4 gvdfg
42342 fsr34  gdgd
543t fsweefw

[root@iz2zegdhs7pd191av8n8dmz awk]# awk '{print NR,NF}' info1.txt info2.txt
1 4
2 5
3 7
4 4
5 3
6 2
```



可以看到在处理多个文件时，awk使用NR输出的行号是多个文件汇总后的结果，而不是指定行在具体文件中的行号，如果我们希望打印出每一行在指定文件中的行号，就可以使用FNR来输出：

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk '{print FNR,NF}' info1.txt info2.txt
1 4
2 5
3 7
1 4
2 3
3 2
```







## RS

RS这个变量代表的是输入记录行分隔符，默认行分隔符就是我们所理解的回车换行。

那么如果我们要把行分隔符修改为一个空格呢？

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk -v RS=" " '{print NR,$0}' info2.txt
1 e3rty
2 eete
3 rte4
4 gvdfg
42342
5 fsr34
6 
7 gdgd
543t
8 fsweefw
```





## ORS

ORS是输出记录行分隔符，与RS对应。

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk -v ORS="**" '{print NR,$0}' info2.txt
1 e3rty eete rte4 gvdfg**2 42342 fsr34  gdgd**3 543t fsweefw**
```





## FILENAME

FILENAME代表的是当前文件的文件名

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk  '{print FILENAME,NR,$0}' info2.txt
info2.txt 1 e3rty eete rte4 gvdfg
info2.txt 2 42342 fsr34  gdgd
info2.txt 3 543t fsweefw
```



## ARGV

ARGV代表当前awk命令中的参数的数量

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{print ARGV[1],ARGV[2]}' info1.txt info2.txt
info1.txt info2.txt

[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{print ARGV[0],ARGV[1],ARGV[2]}' info1.txt info2.txt
awk info1.txt info2.txt

```

ARGV[0]代表的参数是awk，在awk命令中，awk本身被看做是参数，不过'pattern{action}'并不会被看做是参数





## ARGC

ARGC代表的事awk命令中参数的数量

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{print ARGC}' info1.txt info2.txt
3
```





## 自定义变量

awk中自定义变量的方法有两种：

1. 通过-v var=val来定义，变量名区分大小写
2. 在program中直接定义



```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk -v var="value" 'BEGIN{print var}' info1.txt
value

[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{var2="value2";print var2}' info1.txt
value2
```





# 模式

通过上面的介绍想必我们对awk的使用语法应该很了解了：

```shell
awk [选项参数] ‘pattern1{action1}  pattern2{action2}...’ filename
```

我们已经对选项参数这一部分做了介绍，接下来介绍一下一下awk中的模式，也就是pattern这一部分。

pattern模块就是awk中的模式，模式这个词放在这里可能不太好理解，如果换成条件这种说法，应该更好理解一点。

在awk中，每一个模式，都对应着一个action。awk处理文本是逐行处理的，只有前面的模式匹配到了，满足条件的行才会被处理，后面的action才会执行。

awk中的模式一共有五种：空模式、BEGIN-END模式、关系运算符模式、正则模式以及范围模式。



## 空模式

空模式是最简单的一种模式，空模式就是没有模式，只有action，空模式会匹配文本中的每一行，如下

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk '{print NR,$0}' info1.txt
1 118 tery 35ygr 4653tg 
2 346t 35t34eqe y6y  8i33 653
3 24e2 24r2 ygh fds gfd ghe u7u
```





## BEGIN、END模式

BEGIN模式表示在开始处理文本之前需要执行的操作

END模式表示在文本的所有行都处理完毕以后需要执行的操作。

如下：

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{print "BEGIN..."} {print NR,$0} END{print "END..."}' info1.txt
BEGIN...
1 118 tery 35ygr 4653tg 
2 346t 35t34eqe y6y  8i33 653
3 24e2 24r2 ygh fds gfd ghe u7u
END...
```

​	



## 关系运算模式

关系运算模式在上面也有过简单的应用，这里再详细介绍一下，如下：

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk '$1==118{print NR,$0}' info1.txt
1 118 tery 35ygr 4653tg 
```

关系运算模式中的模式是一个关系运算表达式语句，比如案例中的$1==118,它的意思是只有在每一行的第一列为118时，该行才会被处理，该模式对应的action才会被执行。

awk中支持的关系运算符如下：

| 关系运算符 | 含义                       | 用法            |
| :--------: | -------------------------- | :-------------- |
|     <      | 小于                       | x < y           |
|     <=     | 小于等于                   | x <= y          |
|     ==     | 等于                       | x == y          |
|     !=     | 不等于                     | x != y          |
|     >      | 大于                       | x > y           |
|     >=     | 大于等于                   | x >= y          |
|     ~      | 与对应的正则匹配结果为真   | x~/正则表达式/  |
|     !~     | 与对应的正则匹配结果不为真 | x!~/正则表达式/ |



## 正则模式





## 范围模式







# 动作

