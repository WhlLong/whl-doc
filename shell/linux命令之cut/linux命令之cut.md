cut命令可以从一个文本文件或者文本流中提取文本列，cut命令可以从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段输出。



# 基本语法

cut [选项参数]  filename

说明：默认分隔符是制表符



选项参数：

| 选项参数 | 功能                         |
| -------- | ---------------------------- |
| -f       | 列号，提取第几列             |
| -d       | 分隔符，按照指定分隔符分割列 |
| -c       | 指定具体的字符               |





# 案例

准备数据：

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# vim data.txt
[root@iz2zegdhs7pd191av8n8dmz cut]# cat data.txt
an hui
shang hai shi
xiang gang te bie xing zheng qu
tai wan sheng
```



## 剪切出第一列

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# vim cut1.sh
[root@iz2zegdhs7pd191av8n8dmz cut]# cat cut1.sh
#!/bin/bash
cut -d " " -f 1 data.txt
[root@iz2zegdhs7pd191av8n8dmz cut]# sh cut1.sh
an
shang
xiang
tai
```



## 剪切出第2、3列

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# cut -d " " -f 2,3 data.txt
hui
hai shi
gang te
wan sheng
```



## 从文件中剪切出sheng

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# cat data.txt | grep "sheng" 
tai wan sheng
[root@iz2zegdhs7pd191av8n8dmz cut]# cat data.txt | grep "sheng" | cut -d " " -f 3
sheng
```



## 从系统PATH变量值中剪切出第2个“：”开始后的所有路径

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]# echo $PATH
/usr/local/lib/erlang/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/usr/local/jdk/bin:/usr/local/maven/bin:/usr/local/git/bin:/home/whl/.local/bin:/home/whl/bin
[root@iz2zegdhs7pd191av8n8dmz cut]# echo $PATH | cut -d : -f 3-
/usr/bin:/usr/local/sbin:/usr/sbin:/usr/local/jdk/bin:/usr/local/maven/bin:/usr/local/git/bin:/home/whl/.local/bin:/home/whl/bin
```



## 从ifconfig信息中切割出ip地址

```shell
[root@iz2zegdhs7pd191av8n8dmz cut]#  ifconfig eth0 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.148.69  netmask 255.255.240.0  broadcast 172.17.159.255
        ether 00:16:3e:03:10:95  txqueuelen 1000  (Ethernet)
        RX packets 154381273  bytes 21060461004 (19.6 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 157713067  bytes 21832461232 (20.3 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@iz2zegdhs7pd191av8n8dmz cut]#  ifconfig eth0 | grep "inet"
        inet 172.17.148.69  netmask 255.255.240.0  broadcast 172.17.159.255
[root@iz2zegdhs7pd191av8n8dmz cut]#  ifconfig eth0 | grep "inet" | cut -d " " -f 1

[root@iz2zegdhs7pd191av8n8dmz cut]#  ifconfig eth0 | grep "inet" | cut -d " " -f 2

[root@iz2zegdhs7pd191av8n8dmz cut]#  ifconfig eth0 | grep "inet" | cut -d " " -f 3

[root@iz2zegdhs7pd191av8n8dmz cut]#  ifconfig eth0 | grep "inet" | cut -d " " -f 4

[root@iz2zegdhs7pd191av8n8dmz cut]#  ifconfig eth0 | grep "inet" | cut -d " " -f 5

[root@iz2zegdhs7pd191av8n8dmz cut]#  ifconfig eth0 | grep "inet" | cut -d " " -f 6

[root@iz2zegdhs7pd191av8n8dmz cut]#  ifconfig eth0 | grep "inet" | cut -d " " -f 7

[root@iz2zegdhs7pd191av8n8dmz cut]#  ifconfig eth0 | grep "inet" | cut -d " " -f 10
172.17.148.69
```

