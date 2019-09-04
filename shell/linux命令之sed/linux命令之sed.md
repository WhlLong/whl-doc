sed是一种流编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。



# 基本语法

sed [选项参数]  ‘command’  filename



选项参数：

| 选项参数 | 功能                                  |
| -------- | ------------------------------------- |
| -e       | 直接在指令列模式上进行sed的动作编辑。 |
| -i       | 直接编辑文件                          |



命令参数：

| 命令 | 功能描述                              |
| ---- | ------------------------------------- |
| *a*  | 新增，a的后面可以接字串，在下一行出现 |
| d    | 删除                                  |
| s    | 查找并替换                            |



# 案例

## 准备数据

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# vim data.txt
[root@iz2zegdhs7pd191av8n8dmz cut]# cat data.txt
an hui
shang hai shi
xiang gang te bie xing zheng qu
tai wan sheng
```



## 将"guang dong sheng"插入到data.txt的第二行下

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# sed '2a guang dong sheng' data.txt
an hui
shang hai shi
guang dong sheng
xiang gang te bie xing zheng qu
tai wan sheng
[root@iz2zegdhs7pd191av8n8dmz cut]# cat data.txt
an hui
shang hai shi
xiang gang te bie xing zheng qu
tai wan sheng
```

**注意文件并没有改变**





## 删除data.txt中包含“hui”的行

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# sed '/hui/d' data.txt
shang hai shi
xiang gang te bie xing zheng qu
tai wan sheng
[root@iz2zegdhs7pd191av8n8dmz cut]# cat  data.txt
an hui
shang hai shi
xiang gang te bie xing zheng qu
tai wan sheng
[root@iz2zegdhs7pd191av8n8dmz cut]# sed -i '/hui/d' data.txt
[root@iz2zegdhs7pd191av8n8dmz cut]# cat  data.txt
shang hai shi
xiang gang te bie xing zheng qu
tai wan sheng
```



## 将data.txt中的“shang”替换为“qing”

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# sed 's/shang/qing/g' data.txt
qing hai shi
xiang gang te bie xing zheng qu
tai wan sheng
```



## 将data.txt中第一行删除并将"xiang gang"替换为“ao men”

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# sed -e '1d' -e 's/xiang gang/ao men/g' data.txt
ao men te bie xing zheng qu
tai wan sheng
```

