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

正则模式就是将模式内容替换为一个正则表达式，正则表达式放在两个/之间，在使用正则模式时，文本如果能够被正则表达式匹配到，就会执行对应的动作，如果文本无法被正则表达式匹配到，就无法执行对应的动作。

比如下面这个awk命令,如果文本以24开头，就将整行文本打印出来：

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk '/^24/{print $0}' info1.txt
24e2 24r2 ygh fds gfd ghe u7u
```



那么如果正则表达式中包含/该怎么办呢？直接将表达式放进awk命令中肯定是不行的，这个在这里就不演示了。

正确的使用方式是将正则表达式中的/使用\来进行转义，转义后即可正常的执行命令。



在使用正则模式时，有两点需要注意:

1. 当在awk中使用正则模式时，使用的正则法则属于“扩展正则表达式”
2. 当使用{x，y}这种次数匹配的正则表达式时，需要配合--posix选项或者--re-interval选项。



## 范围模式

范围模式也叫行范围模式，它也是使用正则表达式来进行匹配，和正则模式不同的是，正则模式中一般只有一个正则表达式，如果文本能够被该表达式匹配到，就会执行对应的操作，而范围模式有两个正则表达式，其语法如下:

```shell
awk '/正则表达式1/,/正则表达式2/{action}' file
```

它的意义是，从被正则表达式1匹配到的行开始，到被正则表达式匹配到的行结束，这中间的所有行都会执行action操作。

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk '/^11/,/^24/{print $0}' info1.txt
118 tery 35ygr 4653tg 
346t 35t34eqe y6y  8i33 653
24e2 24r2 ygh fds gfd ghe u7u
```







# 动作

awk中的动作，也就是action这一部分，我们应该比较熟悉了。。它的基本结构是：

```shell
{ action }
```

{}内部可以是一条语句，也可以是多条语句，比如之前我们用到的print $0就是一条语句。

除了print这种直接的执行语句，{}内部还可以使用一些条件判断，循环等等,它们的用法和书写格式与java中的格式几乎一样。



## 多条语句

注意： 多条语句之间需要以 ; 隔开,

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# cat info1.txt
118 tery 35ygr 4653tg 
346t 35t34eqe y6y  8i33 653
24e2 24r2 ygh fds gfd ghe u7u
[root@iz2zegdhs7pd191av8n8dmz awk]# awk '{print $1;print $2}' info1.txt
118
tery
346t
35t34eqe
24e2
24r2
```





## if条件判断

if条件判断中的条件判断需要放在()中，其包含的动作语句需要放在{}中

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk '{if($1==118){print $1;print $2}}' info1.txt
118
tery
```







## for循环

for循环分为两种。

for循环语法格式一：

for（初始化；布尔表达式；更新）{

​	//动作语句

}

```
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{for(i=1;i<=5;i++){print i}}' info1.txt
1
2
3
4
5
```



for循环语法格式二：

for( 变量 in 数组 ){

​	//动作语句

}

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]#  awk 'BEGIN{arr[1]="one";arr[2]="two";arr[3]="three";for(i in arr){print i;print arr[i]}}' info1.txt
1
one
2
two
3
three
```



## while循环

while循环也分为两种

while循环语法格式一：

while( 布尔表达式 ){

​	//动作语句

}

```
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{i=1;while(i <= 5){print i;i++}}' info1.txt
1
2
3
4
5
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{i=1;while(i <= 0){print i;i++}}' info1.txt
[root@iz2zegdhs7pd191av8n8dmz awk]# 

```





while循环语法格式二：

do{

​	//动作语句

}while( 布尔表达式 )



while循环与do while循环的区别是： 如果while循环中的布尔表达式结果为false，那么while循环中的动作语句可能一次也不会执行，但是do while循环会先执行一次动作语句，然后再去判断布尔表达式的值来决定要不要继续执行，所以do while循环至少要执行一次动作语句。

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{i=1;do{print i;i++}while(i<=5)}' info1.txt
1
2
3
4
5
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{i=1;do{print i;i++}while(i<=0)}' info1.txt
1
```





## continue与break

continue和break都可以用来跳出循环，不同的是continue只会结束本次动作，后面的循环依旧会继续，而break则是会结束整个循环，后面未执行的循环也不会再执行。

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{for(i=1;i<=5;i++){if(i==3){continue};print i}}' info1.txt
1
2
4
5
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{for(i=1;i<=5;i++){if(i==3){break};print i}}' info1.txt
1
2
```







## exit

exit可以用来结束余下未执行的全部的awk模式匹配和动作执行，直接进入END模式。

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{for(i=1;i<=5;i++){if(i==3){break};print i}} {print $0}  END{print "END..."}' info1.txt
1
2
118 tery 35ygr 4653tg 
346t 35t34eqe y6y  8i33 653
24e2 24r2 ygh fds gfd ghe u7u
END...
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{for(i=1;i<=5;i++){if(i==3){exit};print i}} {print $0}  END{print "END..."}' info1.txt
1
2
END...
```





# 数组

在其他语言中，可能你想要使用数组，不管是赋值操作还是取值操作，都需要先声明数组，然后才能使用这个数组。但是在awk中的数组不需要先声明，直接给数组某个下标赋值或者取值就可以了。

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]#  awk 'BEGIN{arr[1]="one";arr[2]="two";arr[3]="three";for(i in arr){print i;print arr[i]}}' info1.txt
1
one
2
two
3
three
```





另外一个不同于其他语言的地方是，awk中的数组可以使用字符串作为下标，这有点类似于java中的map。

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]#  awk 'BEGIN{arr["one"]=1;arr["two"]=2;arr["three"]=3;  for(i in arr){print i;print arr[i]}}' info1.txt
three
3
two
2
one
1
```



awk中的数组可以使用in来判断某个下标是否存在.如果直接通过下标来判断，那么当该下标不存在时awk会自动创建这个下标，并且将它赋值为空字符串。

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]#  awk 'BEGIN{arr["one"]=1;arr["two"]=2;arr["three"]=3;  if("two" in arr){print "two"};if("four" in arr){print "four"}}' info1.txt
two
```



awk中的数组也可以使用for循环来进行遍历

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]#  awk 'BEGIN{arr["one"]=1;arr["two"]=2;arr["three"]=3;  for(i=1;i<10;i++){arr["one"]=i;print arr["one"]}}' info1.txt
1
2
3
4
5
6
7
8
9
```



# 内置函数



## 算数函数

awk中可以使用rand函数来生成随机数，但是如果只使用rand方法，那么每次生成的随机数是不变的。

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{print rand()}'
0.237788
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{print rand()}'
0.237788
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{print rand()}'
0.237788
```



rand函数一般还需要和srand函数配合使用。

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{srand();print rand()}'
0.399304
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{srand();print rand()}'
0.713137
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{srand();print rand()}'
0.782504
```



上面的例子生成的随机数都是小数，如果想要生成整数随机数，比如100以内的随机整数，可以将上面生成的随机数乘以100，然后通过int函数来截取整数部分。

```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{srand();print 100*rand()}'
8.08286
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{srand();print 100*rand()}'
21.1003
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{srand();print 100*rand()}'
85.3872
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{srand();print 100*rand()}'
40.5307
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{srand();print int(100*rand())}'
80
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{srand();print int(100*rand())}'
12
[root@iz2zegdhs7pd191av8n8dmz awk]# awk 'BEGIN{srand();print int(100*rand())}'
73
```



## 字符串函数

gsub函数与sub函数

gsub函数会替换指定范围内所有符合条件的字符。

sub函数只会替换指定范围内第一次匹配到的符合条件的字符。



```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# cat info1.txt
118 tery 35ygr 4653tg 
346t 35t34eqe y6y  8i33 653
24e2 24r2 ygh fds gfd ghe u7u
[root@iz2zegdhs7pd191av8n8dmz awk]# awk '{gsub("4","s",$1);print $0}' info1.txt
118 tery 35ygr 4653tg 
3s6t 35t34eqe y6y 8i33 653
2se2 24r2 ygh fds gfd ghe u7u

```



```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk '{gsub("4","s",$0);print $0}' info1.txt
118 tery 35ygr s653tg 
3s6t 35t3seqe y6y  8i33 653
2se2 2sr2 ygh fds gfd ghe u7u
```



```shell
[root@iz2zegdhs7pd191av8n8dmz awk]# awk '{gsub("[a-z]","s",$0);print $0}' info1.txt
118 ssss 35sss 4653ss 
346s 35s34sss s6s  8s33 653
24s2 24s2 sss sss sss sss s7s
```



length函数可以获取字符串的长度，传参时获取参数的长度，不传参时取整行的长度。



index函数可以获取字符位于整个字符串中的位置



split函数可以将指定的字符串按照指定的分隔符切割，切割后的每一段复制到数组的元素中，从而动态的创建数组。split函数分割产生的数组，其下标从1开始。



## 其他函数

asort函数与asorti函数

asort函数可以根据元素的值进行排序。

asorti函数可以根据元素的下标进行排序。

