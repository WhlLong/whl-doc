awk是一个强大的文本分析工具，把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行分析处理。

想要用好awk，必须要熟练使用正则表达式。

# 基本用法

awk [选项参数] ‘pattern1{action1}  pattern2{action2}...’ filename

pattern：表示AWK在数据中查找的内容，就是匹配模式

action：在找到匹配内容时所执行的一系列命令

awk命令在执行时，先匹配第一个parttern，如果能匹配的上，就执行第一个parttern对应的action,然后再匹配第二个parttern，如果匹配的上，再执行第二个action.....



选项参数：

| 选项参数 | 功能                 |
| -------- | -------------------- |
| -F       | 指定输入文件折分隔符 |
| -v       | 赋值一个用户定义变量 |





# 案例

## 准备数据

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# vim data.txt
[root@iz2zegdhs7pd191av8n8dmz cut]# cat data.txt
an hui
shang hai shi
xiang gang te bie xing zheng qu 1
tai wan sheng
xiang gang te bie xing zheng qu 2
```



## 输出data.txt中以xiang开头的行

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# awk -F " " '/xiang/{print}' data.txt
xiang gang te bie xing zheng qu 1
xiang gang te bie xing zheng qu 2
```



## 搜索data.txt中以xiang开头的行，并输出第七列

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# awk -F " " '/^xiang/{print $7}' data.txt
qu
qu
```



## 搜索data.txt中以xiang开头的行，并输出第一列和第七列

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# awk -F " " '/^xiang/{print $1$7}' data.txt
xiangqu
xiangqu
```





## 搜索data.txt中以xiang关键字开头的所有行，并输出该行的第1列和第7列，中间以“，”号分割。

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# awk -F " " '/^xiang/{print $1","$7}' data.txt
xiang,qu
xiang,qu
```



## 搜索data.txt中以xiang关键字开头的所有行，并输出该行的第1列和第7列，中间以“，”号分割。且在所有行前面添加列名this is begin,在最后一行添加"this is end"。

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# awk -F " " ' BEGIN{print "this is begin"} /^xiang/{print $1","$7} END{print "this is end"}' data.txt
this is begin
xiang,qu
xiang,qu
this is end
```

**BEGIN 在所有数据读取行之前执行；END 在所有数据执行之后执行。**



