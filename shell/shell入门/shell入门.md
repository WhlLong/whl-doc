# Shell是什么

Shell是一个命令行解释器，它接收应用程序或者是用户的命令，然后调用操作系统的内核。同时Shell还是一个功能强大、容易编写、调试方便、灵活性强的编程语言。

![Shell是什么](Shell是什么.png)





# helloworld

先来一个helloworld吧,脚本名称helloworld.sh

```shell
#!/bin/bash
echo "hello world!"
```



shell脚本执行有两种方式，一种是通过解释器来执行，一种是自己执行，不同处在于通过解释器来执行不需要可执行权限，而自己执行的话脚本需要有可执行权限。



## 解释器执行

1. sh+脚本的相对路径

   ```shell
   sh helloworld.sh
   ```

   

2. sh+脚本的绝对路径

   ```shell
   sh /whl/shell/helloworld.sh
   ```

   

3. bash+脚本的相对路径

   ```shell
   bash helloworld.sh
   ```

   

4. bash+脚本的绝对路径

   ```shell
   bash /whl/shell/helloworld.sh
   ```



执行效果都是一样的：

```
helloworld
```







## 自己执行

如果选择自己来执行Shell脚本，那么首先要赋予脚本可执行权限，否则将会出现如下错误:

```
[root@iz2zegdhs7pd191av8n8dmz shell]# ./helloworld.sh
bash: ./helloworld.sh: Permission denied
```



赋予脚本可执行权限：

```shell
chmod 777 helloworld.sh
```



执行脚本：

相对路径：

```
./helloworld.sh
```



绝对路径：

```
/whl/shell/helloworld.sh
```





## 升级版helloworld

使用Shell脚本在/whl/shell/helloworld目录下创建一个hello.txt,并且在文件中增加“hello world ！”

```shell
#!/bin/bash
cd /whl/shell/helloworld
touch hello.txt
echo "hello world !" >> hello.txt
```



执行sh helloworld.sh,然后会发现/whl/shell/helloworld目录下多了一个hello.txt文件,

然后执行cat hello.txt命令:

```shell
[root@iz2zegdhs7pd191av8n8dmz helloworld]# ./helloworld.sh
[root@iz2zegdhs7pd191av8n8dmz helloworld]# ls -l
total 8
-rw-r--r-- 1 root root  14 Sep  2 21:45 hello.txt
-rwxrwxrwx 1 root root 108 Sep  2 21:45 helloworld.sh
[root@iz2zegdhs7pd191av8n8dmz helloworld]# cat hello.txt
hello world !
```



# Shell中的变量

## 系统变量

Shell中常用的系统变量有$HOME 、$PWD、$SHELL、$USER等等

1. 查看当前Shell中所有变量

   ```shell
   [root@iz2zegdhs7pd191av8n8dmz variable]# set
   BASH=/usr/bin/bash
   BASHOPTS=checkwinsize:cmdhist:expand_aliases:extquote:force_fignore:histappend:hostcomplete:interactive_comments:progcomp:promptvars:sourcepath
   BASH_ALIASES=()
   BASH_ARGC=()
   BASH_ARGV=()
   BASH_CMDS=()
   BASH_LINENO=()
   BASH_SOURCE=()
   BASH_VERSINFO=([0]="4" [1]="2" [2]="46" [3]="2" [4]="release" [5]="x86_64-redhat-linux-gnu")
   BASH_VERSION='4.2.46(2)-release'
   CLASSPATH=/usr/local/jdk/lib/
   COLUMNS=191
   DIRSTACK=()
   ERLANG_HOME=/usr/local/lib/erlang
   EUID=0
   GIT_HOME=/usr/local/git
   GROUPS=()
   HISTCONTROL=ignoredups
   HISTFILE=/root/.bash_history
   HISTFILESIZE=1000
   HISTSIZE=1000
   HOME=/root
   HOSTNAME=iz2zegdhs7pd191av8n8dmz
   HOSTTYPE=x86_64
   ID=0
   IFS=$' \t\n'
   JAVA_HOME=/usr/local/jdk
   LANG=en_US.UTF-8
   LESSOPEN='||/usr/bin/lesspipe.sh %s'
   LINES=35
   LOGNAME=whl
   LS_COLORS='rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;05;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.axa=01;36:*.oga=01;36:*.spx=01;36:*.xspf=01;36:'
   MACHTYPE=x86_64-redhat-linux-gnu
   MAIL=/var/spool/mail/whl
   MAILCHECK=60
   MAVEN_HOME=/usr/local/maven
   OLDPWD=/whl/shell
   OPTERR=1
   OPTIND=1
   OSTYPE=linux-gnu
   PATH=/usr/local/lib/erlang/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/usr/local/jdk/bin:/usr/local/maven/bin:/usr/local/git/bin:/home/whl/.local/bin:/home/whl/bin
   PIPESTATUS=([0]="0")
   PPID=1204
   PROMPT_COMMAND='printf "\033]0;%s@%s:%s\007" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/~}"'
   PS1='[\u@\h \W]\$ '
   PS2='> '
   PS4='+ '
   PWD=/whl/shell/variable
   SHELL=/bin/bash
   SHELLOPTS=braceexpand:emacs:hashall:histexpand:history:interactive-comments:monitor
   SHLVL=2
   SSH_CLIENT='101.88.251.35 23363 22'
   SSH_CONNECTION='101.88.251.35 23363 172.17.148.69 22'
   SSH_TTY=/dev/pts/0
   TERM=xterm
   UID=0
   USER=whl
   XDG_RUNTIME_DIR=/run/user/1000
   XDG_SESSION_ID=8304
   _=system_v.sh
   colors=/root/.dircolors
   _rabbitmqctl_complete () 
   { 
       if [ -x /usr/lib/rabbitmq/bin/rabbitmqctl ]; then
           COMPREPLY=();
           local LANG=en_US.UTF-8;
           local word="${COMP_WORDS[COMP_CWORD]}";
           local completions="$(export LANG=en_US.UTF-8; export LC_CTYPE=en_US.UTF-8; /usr/lib/rabbitmq/bin/rabbitmqctl --auto-complete $COMP_LINE)";
           COMPREPLY=($(compgen -W "$completions" -- "$word"));
       fi
   }
   
   ```

   



2. 使用系统变量,创建system_v.sh脚本：

   ```shell
     1 #!/bin/bash
     2 echo $HOME
     3 echo $PWD
     4 echo $SHELL
     5 echo $USER
   ```

   执行脚本

   ```shell
   [root@iz2zegdhs7pd191av8n8dmz variable]# sh system_v.sh
   /root
   /whl/shell/variable
   /bin/bash
   whl
   ```

   



## 自定义变量

### 基本语法

（1）定义变量：变量=值 

（2）撤销变量：unset 变量

（3）声明静态变量：readonly 变量，注意：不能unset



### 变量定义规则

（1）变量名称可以由字母、数字和下划线组成，但是不能以数字开头，环境变量名建议大写。

（2）等号两侧不能有空格

（3）在bash中，变量默认类型都是字符串类型，无法直接进行数值运算。

（4）变量的值如果有空格，需要使用双引号或单引号括起来。



### 案例

案例一,定义变量：

```shell
[root@iz2zegdhs7pd191av8n8dmz variable]# name=whl
[root@iz2zegdhs7pd191av8n8dmz variable]# echo $name
whl
```



案例二，撤销变量

```shell
[root@iz2zegdhs7pd191av8n8dmz variable]# name=whl
[root@iz2zegdhs7pd191av8n8dmz variable]# echo $name
whl
[root@iz2zegdhs7pd191av8n8dmz variable]# unset name
[root@iz2zegdhs7pd191av8n8dmz variable]# echo $name

[root@iz2zegdhs7pd191av8n8dmz variable]# 

```



案例三，声明静态变量

```shell
[root@iz2zegdhs7pd191av8n8dmz variable]# readonly name=whl
[root@iz2zegdhs7pd191av8n8dmz variable]# echo $name
whl
```



案例四，撤销静态变量

```shell
[root@iz2zegdhs7pd191av8n8dmz variable]# readonly name=whl
[root@iz2zegdhs7pd191av8n8dmz variable]# echo $name
whl
[root@iz2zegdhs7pd191av8n8dmz variable]# unset name
bash: unset: name: cannot unset: readonly variable
```



案例五，尝试以数字开头定义一个变量

```shell
[root@iz2zegdhs7pd191av8n8dmz variable]# count=1
[root@iz2zegdhs7pd191av8n8dmz variable]# echo $count
1
[root@iz2zegdhs7pd191av8n8dmz variable]# 1count=1
bash: 1count=1: command not found
```



案例五，尝试定义变量时在等号两边加上空格

```shell
[root@iz2zegdhs7pd191av8n8dmz variable]# blank = b
bash: blank: command not found
```



案例六，尝试定义变量时进行数值计算

```shell
[root@iz2zegdhs7pd191av8n8dmz variable]# sum=1+2
[root@iz2zegdhs7pd191av8n8dmz variable]# echo $sum
1+2
```



案例七，尝试创建含有空格的变量

```shell
[root@iz2zegdhs7pd191av8n8dmz variable]# blank=I Love You
bash: Love: command not found
[root@iz2zegdhs7pd191av8n8dmz variable]# blank="I Love You"
[root@iz2zegdhs7pd191av8n8dmz variable]# echo $blank
I Love You
```



案例八,将变量提升为全局变量供其他shell脚本使用

提升方法： export 变量

```shell
[root@iz2zegdhs7pd191av8n8dmz variable]# v=hello
[root@iz2zegdhs7pd191av8n8dmz variable]# touch variable.sh
[root@iz2zegdhs7pd191av8n8dmz variable]# vim variable.sh
[root@iz2zegdhs7pd191av8n8dmz variable]# cat variable.sh
#!/bin/bash
echo $PWD
echo $v
[root@iz2zegdhs7pd191av8n8dmz variable]# sh variable.sh
/whl/shell/variable

[root@iz2zegdhs7pd191av8n8dmz variable]# export v
[root@iz2zegdhs7pd191av8n8dmz variable]# sh variable.sh
/whl/shell/variable
hello
```





## 特殊变量

Shell中的特殊变量有$n、$#、$*、$@、$？等等

### $n

功能描述：n为数字，$0代表该脚本名称，$1-$9代表第一到第九个参数，十以上的参数，十以上的参数需要用大括号包含，如${10}

```shell
[root@iz2zegdhs7pd191av8n8dmz parameter]# touch parameter.sh
[root@iz2zegdhs7pd191av8n8dmz parameter]# vim  parameter.sh
[root@iz2zegdhs7pd191av8n8dmz parameter]# cat parameter.sh
#!/bin/bash
echo "$0  $1   $2"
[root@iz2zegdhs7pd191av8n8dmz parameter]# sh  parameter.sh hello world
parameter.sh  hello   world
```





### $#

### $*

### $@

### $？





# Shell中的运算符









# Shell中的条件判断





# Shell中的流程控制





# Shell中的函数