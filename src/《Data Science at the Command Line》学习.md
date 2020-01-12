# 《Data Science at the Command Line》学习

原书参照这个链接：https://www.datascienceatthecommandline.com/

## 第一章 简介

### 1.2 数据科学（Data Science）就是OSEMI

- 1.2.1 O-Obtaining Data（获取数据）
- 1.2.2 S-Scrubbing Data（清洗数据）
- 1.2.3 E-Exploring Data（浏览数据）
- 1.2.4 M-Modeling Data（建模数据）
- 1.2.5 I-Interpreting Data（解释数据）


### 1.3 为什么要使用命令行做数据科学

- 敏捷
- 参数化
- 可扩展
- 普遍

### 1.6 一个实际的例子

这个例子当前有点困难，后面再回顾一下

### 1.7 docker安装

这是我临时插入的一节，如何再ubuntu上安装docker。参照了如下指导：
https://cloud.tencent.com/developer/article/1167995
备注：我再win10的WSL上安装ubuntu然后安装docker时报错了，因此尝试了自己安装虚拟机/购买腾讯云的方式安装，没有遇到问题。any way， 有钱是真好，腾讯云上的主机挺好用的

## 第二章  开始

### 2.1 概览

- 安装docker镜像（image）
- 基本的概念和工具

### 2.2 安装docker镜像

- 下载镜像
```bash
$ docker pull datascienceworkshops/data-science-at-the-command-line # dowaload a image
```
- docker镜像很慢，使用国内的源试试，参照这个链接：https://cloud.tencent.com/developer/article/1335788 ,速度稍微快一点，但还是需要三个多小时。下载完提示如下
```bash
# Status: Downloaded newer image for datascienceworkshops/data-science-at-the-command-line:latest
# docker.io/datascienceworkshops/data-science-at-the-command-line:latest
```
- 运行镜像,运行成功后命令提示符就已经变了。运行whoami提示是root用户
```bash
$ docker run --rm -it datascienceworkshops/data-science-at-the-command-line
alias dr='docker run --rm -it datascienceworkshops/data-science-at-the-command-line' 
```
- 根据书的提示，可以试一下cowsay这个有趣的命令
```bash
 _______________
< I LOVE NAONAO >
 ---------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
- 退出容器，直接输入```exit```
- 数据交互，将本地的一个路径可以映射为容器内的一个路径。例如，opt目录下新建一个datascienceworkspace，将其映射到容器内data目录
```bash
ubuntu@VM-0-5-ubuntu:/opt/datascienceworkspace$ pwd
/opt/datascienceworkspace
ubuntu@VM-0-5-ubuntu:/opt/datascienceworkspace$ ls
test.txt
ubuntu@VM-0-5-ubuntu:/opt/datascienceworkspace$ docker run --rm -it -v`pwd`:/data datascienceworkshops/data-science-at-the-command-line # 路径映射
[/data]$ cd /data/
[/data]$ ls
test.txt
[/data]$ 
```

### 2.3 一些GNU/Linux的基本概念

#### 2.3.1 环境

环境被以下四层定义
- 命令行工具：例如`ls`,`cat`,`jq`
- 终端
- shell：shell是一个解析命令的程序。docker镜像中默认使用Bash，但是当你成为专家后，你会想使用Z-sh
- 操作系统：GNU, which stands for GNU’s not UNIX, refers to the set of basic tools

#### 2.3.2 执行一些基本的命令

- `pwd`

- 一个很长的命令，可以用backslash(\)和管道符分割；分割之后下一行的提示符会变为>，代表continue提示符
```bash
[/]$ echo 'Hello'\
> 'Word' |
> wc
      1       1      10
```
- `head`
```
[/home/data/ch02/data]$ cd /home/data/ch02/data
[/home/data/ch02/data]$ head -n 3 movies.txt 
Matrix
Star Wars
Home Alone
```

#### 2.3.3 5种类型命令行工具

- 二进制可执行文件：由源码编译而成，无法通过文本编辑器查看
- Shell内置程序（Builtin）：例如`cd`,`help`等
- 解释脚本：是一个文本文件，被二进制可执行文件调用：Python， R， Bash等。你可以阅读并查看他,例如如下脚本是计算阶乘的
```python
#!/usr/bin/env python

def factorial(x):
    result = 1
    for i in xrange(2, x + 1):
        result *= i
    return result

if __name__ == "__main__":
    import sys
    x = int(sys.argv[1])
    print factorial(x:q
```
执行方法,因为上面的脚本已经指定了解释器为 #!/usr/bin/env python
```
[/home/data/ch02]$ ./fac.py 5
120
```
- shell函数：通常比shell脚本要短小。例如如下用法，定义了一个shell函数，然后可以直接调用。一个可以定义shell函数的好地方是.bashrc
```
# paste是粘贴合并，将多行合并为一行，默认应该是tab分割，然后把tab分割改为指定的分割符*；最终拼接出的是一个计算表达式，传入bc获取结果
[/home/data/ch02]$ fac() { (echo 1; seq $1) | paste -s -d\* | bc; }
[/home/data/ch02]$ fac 5
120seqq
```
- 别名：别名有点像宏，如果经常执行一个命令但是包含相同的命令字，可以使用别名；例如，针对某个特定的命令经常拼写错误，也可以使用别名.别名的限制是无法使用参数。一般定义在 *.bashrc* 和 *.bash_aliases*
```bash
[/home/data]$ alias l='ls -l --group-directories-first'
[/home/data]$ l
total 36
drwxr-xr-x 3 root root 4096 Sep 11  2018 ch01
drwxr-xr-x 3 root root 4096 Sep 11  2018 ch02
drwxr-xr-x 3 root root 4096 Sep 11  2018 ch03
drwxr-xr-x 3 root root 4096 Sep 11  2018 ch04
drwxr-xr-x 3 root root 4096 Sep 11  2018 ch05
drwxr-xr-x 3 root root 4096 Sep 11  2018 ch06
drwxr-xr-x 3 root root 4096 Sep 11  2018 ch07
drwxr-xr-x 3 root root 4096 Sep 11  2018 ch08
drwxr-xr-x 3 root root 4096 Sep 11  2018 ch09
[/home/data]$ alias moer=more
```
- 在本书中，主要关注后三种命令行工具。可用通过`type` 命令获取一个命令行工具的类型,`type`是一个bash内置的命令
```
type -a pwd
pwd is a shell builtin
pwd is /bin/pwd
```

#### 2.3.4 将命令行工具结合起来

将命令行工具结合起来最重要的方法就是管道。第一个工具的输出传递到第二个工具
```bash
[/home/data]$ seq 100 | grep 3 | wc -l  # -l  specifies that wc should only output the number of lines
19
```

#### 2.3.5 重定向输入和输出

使用重定向符`>`和`>>`可以将输出保存到文件中，>代表重写或者覆盖，>>代表追加。
```bash
[/home/data/ch02/data]$ seq 10 > ten-numbers
[/home/data/ch02/data]$ echo -n "Hello" > hello-word # -n means should not output a trailing newline.
[/home/data/ch02/data]$ echo " Word" >> hello-word
[/home/data/ch02/data]$ cat hello-word | wc
      1       2      11
```
使用重定向符`<`可以将文件当成标准输入读入到程序中。
```
[/home/data/ch02/data]$ < hello-word  wc
 1  2 11
[/home/data/ch02/data]$ wc < hello-word 
 1  2 11
```
另外很多工具可以支持文件作为参数，如下
```
[/home/data/ch02/data]$ wc hello-word 
 1  2 11 hello-word
[/home/data/ch02/data]$ 
```

#### 2.3.6 文件

```bash
$ mv hello.txt ~/book/ch02/data/  # 移动
$ cd data
$ mv hello.txt bye.txt # 重命名
$ rm bye.txt # 删除
$ rm -r book/ch02/data/old # 删除文件夹以及所有文件
$ cp server.log server.log.bak # 拷贝
$ cd data
$ mkdir logs # 创建目录
# 以上所有工具支持 -v 参数，代表verbose，输出详细信息；除了mkdir外支持 -i 参数，代表交互式
# 工具会不停的需要你确认
```

#### 2.3.7 帮助

- man命令（manual），包含大部分工具的信息
- help命令，一些shell内置的命令可以用help获取帮助
```
$ help cd | head -n 20
cd: cd [-L|[-P [-e]] [-@]] [dir]
    ...
```
- 一些新的工具，一般可能没有man帮助信息，但是一般会提供`--help`或者`-h`参数，输出帮助信息

#### 2.3.8 进一步阅读的扩展支持

##### Bash快捷键

https://www.howtogeek.com/howto/ubuntu/keyboard-shortcuts-for-bash-command-shell-for-ubuntu-debian-suse-redhat-linux-etc/
- 进程管理
  - Ctrl + C: Interrupt(kill) the current foreground process running in in the terminal.Send SIGINT signal
  - Ctrl + Z: Suspend the current foreground process running in in the terminal. Send SIGSTP signal
  - Ctrl + D: 关闭shell，即exit
- 屏幕管理
  - Ctrl + L：Clear the Screen. Same as "clear" command
  - Ctrl + S: Stop all output to screen. 当有程序输出很多内容，但是不想停止这个程序只是不想看到这么多输出
  - Ctrl + Q：使用了Ctrl + S 后恢复
- 移动光标  
  - Ctrl + A或者Home,移动到行首
  - Ctrl + E或者End，移动到行末
  - Alt + B： 向左一个单词
  - Ctrl + B： 向左一个字符
  - Alt + F： 向右一个单词
  - Ctrl + F： 向右一个字符
  - Ctrl +XX：移动到行首再从行首移动到原位置
- 删除字符
  - Ctrl + D 或者Delete，删除光标所在字符
  - Ctrl + H 或者BackSpace，删除光标前的一个字符
  - Alt + D：删除光标后的所有字符，但事实上删除的是一个单词而不是所有字符
- 修改
  - Alt + T： 交互当前的单词和前一个单词，没啥用
  - Ctr + T： 交互光标的前的两个字符，但是这个和Google浏览器的新建tab冲突了
  - Ctr + _ ： Undo，但是这个和googl浏览器的缩放冲突了
- 拷贝和粘贴（略）
- 大小写转化
  - Alt + U： 将当前光标到当前单词结尾，转为 
  - Alt + L： 将当前光标到当前单词结尾，转为小写
- Tab 补全，非常有用
- 历史

#####  Unix Power Tools. 3rd

## 第三章  获取数据

可以通过各种方式获取数据，例如，从服务器下载，查询数据库，链接web api。另外有些数据是压缩的或者再二进制格式的文件，例如微软的Excel。在本章节，我们将会学习很多工具帮助我们完成这些工作，例如`curl`，`in2csv`，`sql2csv`，`tar`

### 3.1 概览

学习：
- 从网上获取数据
- 查询数据库
- 链接web api
- 解压文件
- 将微软Excel的文件转为可用的文件

#### 3.2 拷贝本地文件到工具箱

##### 3.2.1 本地工具箱

之前已经讲过文件夹映射的方法，请查看之前的说明

#### 3.2.2 远程工具箱

主要使用scp命令，可以学习下。后面将最基础的用法补充下，方便用于在linux各机器间传递数据

#### 3.3 解压文件

- 常见的压缩格式： .tar.gz  .zip  .rar
- 解压一个.tar.gz包
```
$ cd ~/book/ch03
$ tar -xzvf data/logs.tar.gz   # x means extract, z means gzip commpressed, v
 means verbose, f means use file which after
```
- 一个非常有用的脚本，unpack： 他自己根据文件的扩展名，调用合适的的命令行工具
```bash
#!/usr/bin/env bash
# unpack: Extract common file formats

# Display usage if no parameters given
if [[ -z "$@" ]]; then
    echo " ${0##*/} <archive> - extract common file formats)"
    exit
fi

# Required program(s)
req_progs=(7z unrar unzip)
for p in ${req_progs[@]}; do
    hash "$p" 2>&- || \
    { echo >&2 " Required program \"$p\" not installed."; exit 1; }
done

# Test if file exists
if [ ! -f "$@" ]; then
    echo "File "$@" doesn't exist"
    exit
fi

# Extract file by using extension as reference
case "$@" in
    *.7z ) 7z x "$@" ;;
    *.tar.bz2 ) tar xvjf "$@" ;;
    *.bz2 ) bunzip2 "$@" ;;
    *.deb ) ar vx "$@" ;;
    *.tar.gz ) tar xvf "$@" ;;
    *.gz ) gunzip "$@" ;;
    *.tar ) tar xvf "$@" ;;
    *.tbz2 ) tar xvjf "$@" ;;
    *.tar.xz ) tar xvf "$@" ;;
    *.tgz ) tar xvzf "$@" ;;
    *.rar ) unrar x "$@" ;;
    *.zip ) unzip "$@" ;;
    *.Z ) uncompress "$@" ;;
    * ) echo " Unsupported file format" ;;
esac
```
```
$ unpack logs.tar.gz
```

#### 3.4 转化Excel

CSV文件格式：
- 每行一个Record，用CRLF分割
- 最后一行可以有也可以没有CRLF
- 第一行可能携带头

基本的用法：
- 转化数据 
```
[/home/data/ch03/data]$ ls
finn.txt  imdb-250.xlsx  iris.db  test.csv
[/home/data/ch03/data]$ in2csv imdb-250.xlsx > test.csv
```
- 简单浏览
```
$ in2csv imdb-250.xlsx | head | cut -c1-80  # cut 指定了一行要显示的字符，从多少列到多少列
```
- 可视化浏览
```
$ in2csv data/imdb-250.xlsx | head | csvcut -c Title,Year,Rating | csvlook
|------------------------------------------+------+---------|
|  Title                                   | Year | Rating  |
|------------------------------------------+------+---------|
|  Sherlock Jr. (1924)                     | 1924 | 8       |
|  The Passion of Joan of Arc (1928)       | 1928 | 8       |
|  His Girl Friday (1940)                  | 1940 | 8       |
|  Tokyo Story (1953)                      | 1953 | 8       |
|  The Man Who Shot Liberty Valance (1962) | 1962 | 8       |
|  Persona (1966)                          | 1966 | 8       |
|  Stalker (1979)                          | 1979 | 8       |
|  Fanny and Alexander (1982)              | 1982 | 8       |
|  Beauty and the Beast (1991)             | 1991 | 8       |
|------------------------------------------+------+---------|
```
- 多个sheet页如何转化：一个电子表格如果含有多个sheet页，需要在in2csv中各个--sheet指定；默认只转化第一个sheet页

#### 3.5 查询关系数据库

多数公司将数据存储在关系型数据库中，例如：Mysql，PostgreSql,SQlite等。不同数据库的交互方式差异很大。幸运的是存在一个工具`sql2csv`，可以使用这个工具去跟不同的数据库交互，使用该工具读取SQLite数据库的例子如下：
```
 sql2csv --db 'sqlite:///iris.db' --query 'SELECT * FROM iris WHERE sepal_length > 7.5'  
 # 这个地方有点反人类，默认是从当前路径下找的，
 # 按照注释这个地方应该是dialect+driver://username:password@host:port/database。
 # 那么也就是说，三个///不是路径。如果要写绝对路径
 sql2csv --db 'sqlite:////home/data/ch03/data/iris.db' --query 'SELECT * FROM iris WHERE sepal_length > 7.5'
 # 具体可以参照这个文档 https://csvkit.readthedocs.io/en/0.9.1/tutorial.html
```

#### 3.6 从网络下载

cURL工具是从网络下载数据的瑞士军刀。使用cURL下载数据时，这个工具只是打印到标准输出，下面的工具下载一部小说，并打印前10行
```
curl -s http://www.gutenberg.org/files/76/76-0.txt | head -n 10 # -s means scilent, if not, it will print progress and so on
curl  http://www.gutenberg.org/files/76/76-0.txt > a.txt # -s  is no need this time
curl  http://www.gutenberg.org/files/76/76-0.txt > a.txt
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
 97  601k   97  584k    0     0   9889      0  0:01:02  0:01:00  0:00:02  7948
# 其实也没啥好奇怪的，看起来进度信息等是用stderr输出的
curl  http://www.gutenberg.org/files/76/76-0.txt -o ./b.txt # 也可以直接使用-o参数

# 如果是ftp，使用相同的方法，如果需要密码，可以使用 -u 参数。如果指定的url是个文件夹，会列出所有文件
curl -u username:password ftp://host/file

# 使用短网址时要指定-L/--location参数
# -I/--head参数代表，只获取http头相关信息
curl -I http://www.gutenberg.org/files/76/76-0.txt
HTTP/1.1 200 OK
Server: Apache
Last-Modified: Fri, 23 Feb 2018 20:53:52 GMT
Accept-Ranges: none
Content-Length: 616320
X-Frame-Options: sameorigin
X-Connection: Close
Content-Type: text/plain
X-Powered-By: 1
Date: Fri, 27 Dec 2019 13:43:35 GMT
X-Varnish: 382372871
Age: 0
Via: 1.1 varnish
```

#### 3.7 访问webAPI

web API指的是WEB 应用可编程接口（Application Programming Interface）.他一般提供的数据不是美观的，但是一般时结构化的，例如JSON或者XML，可以很方便的被其他工具处理，例如jq。
```
curl -s https://randomuser.me/api/1.2/ | jq .
{
  "results": [
    {
      "gender": "male",
      "name": {
        "title": "mr",
        "first": "armand",
        "last": "richard"
      },
      ...
```
文中举例子的从twitter获取数据的例子暂时无法执行，所以需要鉴权的一些接口暂时还不会使用


#### 3.8 进一步阅读

- SQL Cookbook
- List of Http Status Codes

## 第四章  创建可重用的命令行工具

有些任务你只需要执行一次，但是一些需要多次执行。有些任务非常具体，但是有些可以抽象并且泛化，因此是时候去关注命令行工具了，而不是简单的一两句命令行。命令行工具只需要创建一次，它可以从命令行调用、接受参数；命令行工具肯定会使你成为效率更高、更具创意的数据科学家。

### 4.1 概览

- 将命令行转为Shell脚本
- 使得Python/R/Java代码成为工具的一部分

### 4.2 将命令行转为Shell脚本
如下的命令是统计电子书《Adventures of Huckleberry Finn》中出现top10 的单词:
```shell
curl -s http://www.gutenberg.org/files/76/76-0.txt | 
> tr '[:upper:]' '[:lower:]' | 
> grep -oE '\w+' | 
> sort | 
> uniq -c | 
> sort -nr | 
> head -n 10
   6439 and
   5077 the
   3666 i
   3258 a
   3022 to
   2567 it
   2086 t
   2044 was
   1847 he
   1777 of
```
具体解释如下：
- 使用`curl`下载
- 使用`tr`转为，将大写转小写
- 使用`grep`展开所有单词并把单词每行一个放置,去除符号---可以和之前的paste命令对比，将多行合并一行
```
echo "Hello world! Hello dream!" | grep -oE '\w+'
Hello
world
Hello
dream
```
- 使用`sort`用字典序排序
- 使用`uniq`移除重复行并计数
- 使用`sort`排序，按照数字顺序,注意是降序
```
# sort和uniq的用法要学习一下
```shell
echo "Hello world! Hello dream!" | grep -oE '\w+' |
> sort
Hello
Hello
dream
world

echo "Hello world! Hello dream!" | grep -oE '\w+' | sort |
> uniq -c
      2 Hello
      1 dream
      1 world
echo "Hello world! Hello dream!" | grep -oE '\w+' | sort | uniq -c | sort -nr
      2 Hello
      1 world
      1 dream
echo "Hello world! Hello dream!" | grep -oE '\w+' | sort | uniq -c | sort -n
      1 dream
      1 world
      2 Hello
```
- 使用`top`保留前10行

通过以下几个步骤将这些one-lines转化成命令行工具--其实就是脚本：
- 粘贴复制到文件
- 增加可执行权限
- 增加shebang
- 移除固定的输入
- 增加参数
- 扩展PATH变量

#### 4.2.1 粘贴复制

将我们的粘贴到文件top-words-1.sh 
```
$vi top-words-1.sh 
curl -s http://www.gutenberg.org/files/76/76-0.txt |
tr '[:upper:]' '[:lower:]' | grep -oE '\w+' | sort |
uniq -c | sort -nr | head -n 10
```
然后可以通过如下方式运行了,无法自己单独执行：
```
bash top-words-1.sh 
```
一个shell的小技巧：`!!`会扩展为上次运行的命令。因此如果你想要将上次运行的命令保存的脚本，直接可以使用echo "!!" > scriptname

#### 4.2.2  添加可执行权限

一个shell技巧，将top-words-1.sh复制到top-words-2.sh
```
cp top-words-{1,2}.sh
cp top-words-1.{sh,sh.back}  # 这个可以快速用于备份
ls -l
total 44
...
-rw-r--r-- 1 root root  139 Sep 11  2018 top-words-1.sh
-rw-r--r-- 1 root root  139 Jan  4 04:02 top-words-1.sh.back
```
以下命令用于添加可执行权限
```
chmod u+x top-words-2.sh
```
- u means we want to change the permission for the user whon owns the file
- + means add
- x means permission to excute
增加权限后就可以用如下方式直接运行，不需要显式调用bash
```
./top-words-2.sh
```
如果没有添加可执行权限，执行会报错
```
./top-words-1.sh
bash: ./top-words-1.sh: Permission denied
```

#### 4.2.3  添加shebang

shebang是告诉系统，需要用哪个可执行文件取解析脚本。shebang的意思（#->hash, !->bang）
```
#!/usr/bin/env bash
curl -s http://www.gutenberg.org/files/76/76-0.txt |
tr '[:upper:]' '[:lower:]' | grep -oE '\w+' | sort |
uniq -c | sort -nr | head -n 10
```
shebang必须要写，尽管大部分unix上默认shell就是bash shell.还有一种写法如下，直接指定bash程序的路径，但是一旦bash安装目录
不是这个，将无法正常工作。建议使用env,因为env会识别bash安装目录（python也是如此）

#### 4.2.4  移除固定的输入

在我们已经完成的脚本本，我们从网上下载ebook并获取其中使用量最多的单词，我们没有将数据和操作隔离。如果我们需要从其他的ebbok或者txt统计呢？
因此最好将数据和操作隔离开来。
```
#!/usr/bin/env bash
tr '[:upper:]' '[:lower:]' | grep -oE '\w+' | sort |
uniq -c | sort -nr | head -n 10
```
这样可以用以下方法，通过管道处理我们向处理的文档
```
cat data/finn.txt | top-words-4.sh
```

#### 4.2.5  增加参数

例如，我们可能需要让客户指定，需要获取使用量前10或者前20的单词，最好是将此参数作为命令行的参数，调用时传入
```
#!/usr/bin/env bash
NUM_WORDS="$1"                                        
tr '[:upper:]' '[:lower:]' | grep -oE '\w+' | sort |
uniq -c | sort -nr | head -n $NUM_WORDS               
```
对于变量，定义时不需要$,使用时需要$. $1代表传入的第一个参数
```
cat data/finn.txt | top-words-5.sh 5   
bash: top-words-5.sh: command not found  # befor you extend PATH, it will not work like this

cat data/finn.txt | ./top-words-5.sh 5
   6441 and
   5082 the
   3666 i
   3258 a
   3022 to
   
cat data/finn.txt | ./top-words-5.sh   # if not give param, it will report error now
head: option requires an argument -- 'n'
Try 'head --help' for more information.
```

#### 4.2.6  扩展PATH环境变量

如果你向在任何地方都能执行你写命令行工具，你需要把该命令行工具所在路径加到环境变量$PATH中。该环境变量其实就时指定Bash应该去那些地方找可执行的命令行或者其他可执行文件。

- 可以用以下方法，列出当前的PATH变量，每行一个
```
echo $PATH | tr ':' '\n'
/usr/local/sbin
/usr/local/bin
/usr/sbin
/usr/bin
/sbin
/bin
```
- 

### 4.3 使用Python和R创建命令行工具

使用编程语言而不是Bash脚本的原因：

- 有一些现成的代码
- 命令行工具可能很复杂，超过100行
- 需要执行效率高

### 4.3.1 移植shell脚本

python版本如下， R语言版本略
```
#!/usr/bin/env python
import re
import sys
from collections import Counter
num_words = int(sys.argv[1])
text = sys.stdin.read().lower()
words = re.split('\W+', text)
cnt = Counter(words)
for word, count in cnt.most_common(num_words):
    print "%7d %s" % (count, word)
```
使用重定向的方法执行
```
< data/76.txt top-words.py 5
```

### 4.3.2 处理标准输入流

在命令行中，大部分命令行工具需要使用pipe将上个程序的输出作为下个程序的输入（有些程序会要求在输入之前先读入全部的数据，例如sort和awk，这对输入有限且固定的场景来说没有影像，但是当输入如果时不停止且连续输入的，这种工具就没法正常工作）。对于python来说，可以使用如下脚本处理标准输入
```python
#!/usr/bin/env python
from sys import stdin, stdout
while True:
    line = stdin.readline()
    if not line:
        break
    stdout.write("%d\n" % int(line)**2)
    stdout.flush()
```

### 4.4 进一步阅读

- 方便构建命令行的工具，简化参数说明/参数解析等。Docopt http://docopt.org/
- 《Classic Shell Scripting》
- 《Unix Power Tools》
- 《Python Text Processing with Nltk 2.0 Cookbook.》
- 《Python for Data Analysis》
- 《Learning Ipython for Interactive Computing and Data Visualization》
- Man 帮助文档书写 http://liw.fi/manpages/
- 在线收集faq和RFC文档 http://www.faqs.org/faqs/

## 第五章 清洗数据

- 大部分工具都只能支持一种格式的数据，因此需要关注数据转化
- CSV是一种我们常用的格式，但是这种格式并不利于工作，经常会被破坏
- 一旦数据格式转成我们想要到，我们有大量的工作取做一些清洗的工作（例如，过滤，替换，合并等）。有大量的工具可以使用：cut， sed， jq， csvgrep
- 有时我们也需要将我们的输出数据重新转化。例如，将uniq -c的结果转成CSV格式,如下所示
```
$ echo -e 'foo\nbar\nfoo' | sort | uniq -c | sort -nr   # 为了使用转义字符，使用-e
      2 foo
      1 bar

$ echo -e "foo\nbar\nfoo" | sort | uniq -c | sort -nr | awk '{print $2","$1}' |  header -a value,count
value,count
foo,2
bar,1
```

### 5.1 概览

- 转化数据的格式
- 在csv文件上做SQL查询
- 解析和替换值
- 分割，合并和解析某列

### 5.2 针对纯文本数据的清洗操作

纯文本指的是人类可阅读字符序列在加上一些特定的控制字符（tab/newline）。在本文中，我们指的是不是那些表格形式（CSV）或者结构化（JSON或者HTML）的数据。

#### 5.2.1 过滤行

##### 5.2.1.1 基于位置

先创建一个测试的文件，tee命令（将标准输入写入到文件）
```
 seq -f "Line %g" 10 | tee lines
Line 1
Line 2
Line 3
Line 4
Line 5
Line 6
Line 7
Line 8
Line 9
Line 10
```
- 打印前三行
```
< lines head -n 3
Line 1
Line 2
Line 3
< lines sed -n '1,3p'
Line 1
Line 2
Line 3
< lines awk 'NR<=3'
Line 1
Line 2
Line 3
```
- 打印最后三行
```
< lines tail -n 3
Line 8
Line 9
Line 10
```
- 移除前三方
```
< lines tail -n +4
Line 4
Line 5
Line 6
Line 7
Line 8
Line 9
Line 10
< lines sed '1,3d'
Line 4
Line 5
Line 6
Line 7
Line 8
Line 9
Line 10
< lines sed -n '1,3!p'
Line 4
Line 5
Line 6
Line 7
Line 8
Line 9
Line 10
```
- 移除最后三方
```
< lines head -n -3
Line 1
Line 2
Line 3
Line 4
Line 5
Line 6
Line 7
```
- 打印中间行（4-6）
```
$ < lines sed -n '4,6p'
Line 4
Line 5
Line 6
$ < lines awk '(NR>=4)&&(NR<=6)'
Line 4
Line 5
Line 6
$ < lines head -n 6 | tail -n 3
Line 4
Line 5
Line 6
```
- 打印奇数行
```
$ < lines sed -n '1~2p'  # 指定开始和step即可
Line 1
Line 3
Line 5
Line 7
Line 9
$ < lines awk 'NR%2'  # 使用模运算
Line 1
Line 3
Line 5
Line 7
Line 9
```
- 打印偶数行
```
$ < lines sed -n '0~2p'
Line 2
Line 4
Line 6
Line 8
Line 10
$ < lines awk '(NR+1)%2'
Line 2
Line 4
Line 6
Line 8
Line 10
```

##### 5.2.1.2 基于模式匹配

grep是经典的文本过滤工具，会打印每一行匹配到的行，如下，打印所有包含chapter的行，忽略大小写
```
grep -i chapter alice.txt
CHAPTER I. Down the Rabbit-Hole
CHAPTER II. The Pool of Tears
CHAPTER III. A Caucus-Race and a Long Tale
CHAPTER IV. The Rabbit Sends in a Little Bill
CHAPTER V. Advice from a Caterpillar
CHAPTER VI. Pig and Pepper
CHAPTER VII. A Mad Tea-Party
CHAPTER VIII. The Queen's Croquet-Ground
CHAPTER IX. The Mock Turtle's Story
CHAPTER X. The Lobster Quadrille
CHAPTER XI. Who Stole the Tarts?
CHAPTER XII. Alice's Evidence
```
通常情况下，grep按普通的字符串处理partten，指定-E参数可以是能正则匹配
```
grep -E '^CHAPTER (.*)\. The' alice.txt 
CHAPTER II. The Pool of Tears
CHAPTER IV. The Rabbit Sends in a Little Bill
CHAPTER VIII. The Queen's Croquet-Ground
CHAPTER IX. The Mock Turtle's Story
CHAPTER X. The Lobster Quadrille
```

##### 5.2.1.3 随机匹配
命令行工具sample随机获取一个子集，可以指定采样率(每一行有%1的几率被打印出来，但并代表最后一定有10行数据)。
```
seq 1000 | sample -r 1% | jq -c '{line: .}'
{"line":243}
{"line":392}
{"line":409}
{"line":495}
{"line":502}
{"line":562}
{"line":619}
{"line":714}
{"line":715}
{"line":732}
{"line":854}
{"line":878}
{"line":890}
{"line":896}
{"line":897}
```
sample在你debug工具时有其他的用途：1，如果输入数据非常大，且时连续的，可以使用sample降低数据量；2，当你不想影响采样率，你可以使用定时器。如下时每行输出后停顿1s，这样执行5s后.
```
seq 1000 | sample -r 1% -d 1000 -s 5 | jq -c '{line: .}'
{"line":8}
{"line":151}
{"line":182}
{"line":312}
{"line":547}
{"line":584}
```

#### 5.2.2 提取值

为了电子书中每一章节的名字，我们可以这样
```
grep -i chapter alice.txt | cut -d' ' -f3-
```
通过cut命令将输入行按照指定的分割符切成fields，然后打印指定的fileds（-f后面跟参数N，N-代表只到最后一列）。使用sed也可以做到相同的效果
```
sed -rn 's/^CHAPTER ([IVXLCDM]{1,})\. (.*)$/\2/p' alice.txt 
sed -rn 's/^CHAPTER ([IVXLCDM]{1,})\. (.*)$/\2/p' alice.txt > /dev/null  # if you can not watch this
```
另外，cut命令可以基于字符的位置切割,这不就是列编辑吗？ 哈哈
```
$ grep -i chapter alice.txt
CHAPTER I. Down the Rabbit-Hole
CHAPTER II. The Pool of Tears
CHAPTER III. A Caucus-Race and a Long Tale
CHAPTER IV. The Rabbit Sends in a Little Bill
CHAPTER V. Advice from a Caterpillar
CHAPTER VI. Pig and Pepper
CHAPTER VII. A Mad Tea-Party
CHAPTER VIII. The Queen's Croquet-Ground
CHAPTER IX. The Mock Turtle's Story
CHAPTER X. The Lobster Quadrille
CHAPTER XI. Who Stole the Tarts?
CHAPTER XII. Alice's Evidence
$ grep -i chapter alice.txt | cut -c9-
I. Down the Rabbit-Hole
II. The Pool of Tears
III. A Caucus-Race and a Long Tale
IV. The Rabbit Sends in a Little Bill
V. Advice from a Caterpillar
VI. Pig and Pepper
VII. A Mad Tea-Party
VIII. The Queen's Croquet-Ground
IX. The Mock Turtle's Story
X. The Lobster Quadrille
XI. Who Stole the Tarts?
XII. Alice's Evidence
```
grep有一个重要特性时可以将匹配的串分割到不同的行,之前已经见识过
```
$ < alice.txt grep -oE '\w{2,}' | head
Project
Gutenberg
Alice
Adventures
in
Wonderland
by
Lewis
Carroll
This
```
如果我们要统计那些以a开始/以e结束的单词，可以使用如下的方法
```
< alice.txt  tr '[:upper:]' '[:lower:]' | grep -oE '\w{2,}' | grep -E '^a.*e$'| sort | uniq -c | sort -nr | awk '{print $2","$1}' | header -a word,count | head | csvlook
| word       | count |
| ---------- | ----- |
| alice      |   403 |
| are        |    73 |
| archive    |    13 |
| agree      |    11 |
| anyone     |     5 |
| alone      |     5 |
| age        |     4 |
| applicable |     3 |
| anywhere   |     3 |
```

#### 5.2.2 替换和删除值

- 使用tr替换一个值（将空格替换为_）
```
[/home/data/ch05/data]$ echo 'hello word!' | tr ' ' '_'
hello_word!
```
- 使用tr替换多个值（将空格替换为_， ！替换为？）
```
[/home/data/ch05/data]$ echo 'hello word!' | tr ' !' '_?'
hello_word?
```
- 删除在集合中的字符
```
[/home/data/ch05/data]$ echo 'hello word!' | tr -d '[a-z]'
 !
[/home/data/ch05/data]$ echo 'hello word!' | tr -d -c '[a-z]'  # c 代表的补集
helloword[/home/data/ch05/data]$ 
```
- 小写转大写
```
$ echo 'hello word!' | tr '[a-z]' '[A-Z'
HELLO WORD!
$ echo 'hello word!' | tr '[:lower:]' '[:upper:]'
HELLO WORD!
```
- 在删除，替换这类文本操作上，sed非常用用。例如，删除开头空格以及多个连续空格，并且做替换（hello替换成bay）.-g代表global，代表在一个一行多次匹配而不是只匹配一次
```
$ echo ' hello      word!' | sed -re 's/hello/bye/;s/\s+/ /g;s/\s+//'
bye word!
```

### 5.3 处理CSV

#### 5.3.1 body&header&cols

首先介绍几个有用的工具
- body：使用body，将命令只应用到CSV文件的body上，忽略列名（header）。
几个简单的示例如下：
```
$ echo -e "value\n7\n2\n5\n3" | body sort -n
value
2
3
5
7
$ seq 5 | header -a count | wc -l
6
$ seq 5 | header -a count | body wc -l
count
5
```
body的实现如下
```
#!/usr/bin/env bash
IFS= read -r header        
printf '%s\n' "$header"    
$@                         
```
- header:
header允许我们操作csv的列名。其实现如下,使用了之前扩展阅读中的getopt工具。
```
#!/usr/bin/env bash
get_header () {
        for i in $(seq $NUMROWS); do
                IFS= read -r LINE
                OLDHEADER="${OLDHEADER}${LINE}\n"
        done
}

print_header () {
        echo -ne "$1"
}

print_body () {
        cat
}

OLDHEADER=
NUMROWS=1

while getopts "dn:ha:r:e:" OPTION
do
        case $OPTION in
                n)
                        NUMROWS=$OPTARG
                        ;;
                a)
                        print_header "$OPTARG\n"
                        print_body
                        exit 1
                        ;;
                d)
                        get_header
                        print_body
                        exit 1
                        ;;
                r)
                        get_header
                        print_header "$OPTARG\n"
                        print_body
                        exit 1
                        ;;
                e)
                        get_header
                        print_header "$(echo -ne $OLDHEADER | eval $OPTARG)\n"
                        print_body
                        exit 1
                        ;;
                h)
                        usage
                        exit 1
                        ;;
        esac
done

get_header
print_header $OLDHEADER
```
用法如下
```
OPTIONS:
  -n      Number of lines to consider as header [default: 1]
  -a      Add header
  -r      Replace header
  -e      Apply expression to header
  -d      Delete header
  -h      Show this message

Example usage:
  $ seq 10 | header -a 'values'
  $ seq 10 | header -a 'VALUES' | header -e 'tr "[:upper:]" "[:lower:]"'
  $ seq 10 | header -a 'values' | header -d
  $ seq 10 | header -a 'multi\nline' | header -n 2 -e "paste -sd_"
```
几个使用的例子：
```
cat iris.csv | header  # print header
sepal_length,sepal_width,petal_length,petal_width,species

cat iris.csv | header -d | head # delete header

seq 5 | header -a line | header -e "tr '[a-z]' '[A-Z]'"  # add and replace
LINE
1
2
3
4
5
```
- cols: 允许你在某一些行上做一些操作。代码实现如下,就是先拆分，再合并（中间会生成临时文件，但最后会删除）
```
#!/usr/bin/env bash
ARG="$1"
shift
COLUMNS="$1"
shift
EXPR="$@"
DIRTMP=$(mktemp -d)
mkfifo $DIRTMP/other_columns
tee $DIRTMP/other_columns | csvcut $ARG $COLUMNS | ${EXPR} |
paste -d, - <(csvcut ${ARG~~} $COLUMNS $DIRTMP/other_columns)
rm -rf $DIRTMP
```
下面时使用示例
```
< tips.csv cols -c day body "tr '[a-z]' '[A-Z]'" | head -n 5 | csvlook -I
| day | bill  | tip  | sex    | smoker | time   | size |
| --- | ----- | ---- | ------ | ------ | ------ | ---- |
| SUN | 16.99 | 1.01 | Female | No     | Dinner | 2    |
| SUN | 10.34 | 1.66 | Male   | No     | Dinner | 3    |
| SUN | 21.01 | 3.5  | Male   | No     | Dinner | 3    |
| SUN | 23.68 | 3.31 | Male   | No     | Dinner | 2    |
```
 csvlook -I的目的时禁用类型推理，这个推理有时候比较弱智
 
 #### 5.3.2 在csv文件上做sql查询
 ```
seq 5 | header -a value | csvsql --query "SELECT SUM(value) AS sum from stdin"
sum
15
 ```
 
 
### 5.4 处理XML/HTML和JSON

一般需要将XML/HTML和JSON格式的数据转为CSV：
- 很多数据可视化和机器学习算法需要表格数据
- 很多命令行工具一般只能在纯文本数据操作，例如cut和grep

另外有些时候其实页不需要转化，我们可以将之前的工具直接用在JSON数据上。例如，我们想要把gender属性改成sex，可以直接使用sed进行全局替换
```
$ cat data/users.json | jq | grep gender
      "gender": "male",
      "gender": "female",
      "gender": "male",
      "gender": "female",
      "gender": "male",
 $ sed -e 's/"gender":/"sex":/g' data/users.json | jq | grep sex
      "sex": "male",
      "sex": "female",
      "sex": "male",
      "sex": "female",
      "sex": "male",
```
- 查看HTML中的表格，使用grep查看我们看兴趣的元素，可以使用-A参数指定打印匹配到打印后面多少行
```
 < wiki.html grep wikitable -A 21
<table class="wikitable sortable">
<tr>
<th>Rank</th>
<th>Country or territory</th>
<th>Total length of land borders (km)</th>
<th>Total surface area (km²)</th>
<th>Border/area ratio (km/km²)</th>
</tr>
<tr>
<td>1</td>
<td>Vatican City</td>
<td>3.2</td>
<td>0.44</td>
<td>7.2727273</td>
</tr>
<tr>
<td>2</td>
<td>Monaco</td>
<td>4.4</td>
<td>2</td>
<td>2.2000000</td>
</tr>
```
- 将我们感兴趣的元素从HTML中提取出来。我们使用scrape工具。我们使用scrape提取"wikitable sortable" 这个class下的<tr>元素或者说行（但是不想要第一行，因为第一个tr元素代表的表格的头），并且放在一个body标签中。
 ```
$ < data/wiki.html scrape -b -e 'table.wikitable > tr:not(:first-child)' \
> > data/table.html
$ head -n 21 data/table.html
<!DOCTYPE html>
<html>
<body>
<tr><td>1</td>
<td>Vatican City</td>
<td>3.2</td>
<td>0.44</td>
<td>7.2727273</td>
</tr>
<tr><td>2</td>
<td>Monaco</td>
<td>4.4</td>
<td>2</td>
<td>2.2000000</td>
</tr>
<tr><td>3</td>
<td>San Marino</td>
<td>39</td>
<td>61</td>
<td>0.6393443</td>
</tr>
```
 - xml2json可以将XML转成JSON
 ```
 $ < data/table.html xml2json > data/table.json
$ < data/table.json jq '.' | head -n 25
{
  "html": {
    "body": {
      "tr": [
        {
          "td": [
            {
              "$t": "1"
            },
            {
              "$t": "Vatican City"
            },
            {
              "$t": "3.2"
            },
            {
              "$t": "0.44"
            },
            {
              "$t": "7.2727273"
            }
          ]
        },
        {
          "td": [
 ```
 - 我们将数据转为json的原因是有一个强大的工具jq，可以很好的处理json数据。下面的命令将提取json中某些部分并整理成格式
```
$ < data/table.json jq -c '.html.body.tr[] | {country: .td[1][],border:''.td[2][], surface: .td[3][]}' > data/countries.json
$ head -n 10 data/countries.json
{"country":"Vatican City","border":"3.2","surface":"0.44"}
{"country":"Monaco","border":"4.4","surface":"2"}
{"country":"San Marino","border":"39","surface":"61"}
{"country":"Liechtenstein","border":"76","surface":"160"}
{"country":"Sint Maarten (Netherlands)","border":"10.2","surface":"34"}
{"country":"Andorra","border":"120.3","surface":"468"}
{"country":"Gibraltar (United Kingdom)","border":"1.2","surface":"6"}
{"country":"Saint Martin (France)","border":"10.2","surface":"54"}
{"country":"Luxembourg","border":"359","surface":"2586"}
{"country":"Palestinian territories","border":"466","surface":"6220"}
```
 - 将json数据转为csv
 ```
$  < data/countries.json json2csv -p -k border,surface > data/countries.csv
$ head -n 11 data/countries.csv | csvlook
| border |  surface |
| ------ | -------- |
|    3.2 |     0.44 |
|    4.4 |     2.00 |
|   39.0 |    61.00 |
|   76.0 |   160.00 |
|   10.2 |    34.00 |
|  120.3 |   468.00 |
|    1.2 |     6.00 |
|   10.2 |    54.00 |
|  359.0 | 2,586.00 |
|  466.0 | 6,220.00 |

 ```

### 5.5 CSV文件的一些惯用操作

#### 5.5.1  提取和保存列

- 使用csvcut -c提取关注的行 
```
$ head data/iris.csv
sepal_length,sepal_width,petal_length,petal_width,species
5.1,3.5,1.4,0.2,Iris-setosa
4.9,3.0,1.4,0.2,Iris-setosa
4.7,3.2,1.3,0.2,Iris-setosa
4.6,3.1,1.5,0.2,Iris-setosa
5.0,3.6,1.4,0.2,Iris-setosa
5.4,3.9,1.7,0.4,Iris-setosa
4.6,3.4,1.4,0.3,Iris-setosa
5.0,3.4,1.5,0.2,Iris-setosa
4.4,2.9,1.4,0.2,Iris-setosa

$ < data/iris.csv csvcut -c sepal_length,petal_length,sepal_width,petal_width |
> head -n 5 | csvlook
| sepal_length | petal_length | sepal_width | petal_width |
| ------------ | ------------ | ----------- | ----------- |
|          5.1 |          1.4 |         3.5 |         0.2 |
|          4.9 |          1.4 |         3.0 |         0.2 |
|          4.7 |          1.3 |         3.2 |         0.2 |
|          4.6 |          1.5 |         3.1 |         0.2 |
```

- 也可以通过-C参数指定不关注的行
```
$ < data/iris.csv csvcut -C species |  head -n 5 | csvlook
| sepal_length | sepal_width | petal_length | petal_width |
| ------------ | ----------- | ------------ | ----------- |
|          5.1 |         3.5 |          1.4 |         0.2 |
|          4.9 |         3.0 |          1.4 |         0.2 |
|          4.7 |         3.2 |          1.3 |         0.2 |
|          4.6 |         3.1 |          1.5 |         0.2 |
```
- 也可以指定列号，从1开始
```
$ echo -e 'a,b,c,d,e,f,g,h,i\n1,2,3,4,5,6,7,8,9' |
> csvcut -c $(seq 1 2 9 | paste -sd,)
a,c,e,g,i
1,3,5,7,9
$ echo $(seq 1 2 9 | paste -sd,)
1,3,5,7,9
```
- 如果值里没有“,”,完全可以使用cut完成相同的功能,注意，cut影响不了列的顺序
```
$ echo -e 'a,b,c,d,e,f,g,h,i\n1,2,3,4,5,6,7,8,9' | cut -d, -f5,1,3
a,c,e
1,3,5
```
- 最后，我们也可以直接在CSV格式数据上使用SQL做提取和保存
```
$ < iris.csv csvsql --query "SELECT sepal_length, petal_length, ""sepal_width, petal_width FROM stdin" | head -n 10 | csvlook
| sepal_length | petal_length | sepal_width | petal_width |
| ------------ | ------------ | ----------- | ----------- |
|          5.1 |          1.4 |         3.5 |         0.2 |
|          4.9 |          1.4 |         3.0 |         0.2 |
|          4.7 |          1.3 |         3.2 |         0.2 |
|          4.6 |          1.5 |         3.1 |         0.2 |
|          5.0 |          1.4 |         3.6 |         0.2 |
|          5.4 |          1.7 |         3.9 |         0.4 |
|          4.6 |          1.4 |         3.4 |         0.3 |
|          5.0 |          1.5 |         3.4 |         0.2 |
|          4.4 |          1.4 |         2.9 |         0.2 |
```

#### 5.5.2  过滤行

- 基于位置的过滤跟纯文本一样，但是注意，要考虑头header。注意使用header和body两个命令
```
$ seq 5 | sed -n '3,5p'
3
4
5
$ seq 5 | header -a count| sed -n '3,5p'
2
3
4
$ seq 5 | header -a count| body sed -n '3,5p'
count
3
4
5
```
- 基于某一列的模式匹配过滤，需要使用csvgrep，awk，csvsql
 - 排除size不大于4的订单
 ```
$  head tips.csv | csvlook
|  bill |  tip | sex    | smoker |        day | time   | size |
| ----- | ---- | ------ | ------ | ---------- | ------ | ---- |
| 16.99 | 1.01 | Female |  False | 0001-01-07 | Dinner |    2 |
| 10.34 | 1.66 | Male   |  False | 0001-01-07 | Dinner |    3 |
| 21.01 | 3.50 | Male   |  False | 0001-01-07 | Dinner |    3 |
| 23.68 | 3.31 | Male   |  False | 0001-01-07 | Dinner |    2 |
| 24.59 | 3.61 | Female |  False | 0001-01-07 | Dinner |    4 |
| 25.29 | 4.71 | Male   |  False | 0001-01-07 | Dinner |    4 |
|  8.77 | 2.00 | Male   |  False | 0001-01-07 | Dinner |    2 |
| 26.88 | 3.12 | Male   |  False | 0001-01-07 | Dinner |    4 |
| 15.04 | 1.96 | Male   |  False | 0001-01-07 | Dinner |    2 |

$ csvgrep -c size -i -r "[1-4]" tips.csv | csvlook
|  bill |  tip | sex    | smoker | day  | time   | size |
| ----- | ---- | ------ | ------ | ---- | ------ | ---- |
| 29.80 | 4.20 | Female |  False | Thur | Lunch  |    6 |
| 34.30 | 6.70 | Male   |  False | Thur | Lunch  |    6 |
| 41.19 | 5.00 | Male   |  False | Thur | Lunch  |    5 |
| 27.05 | 5.00 | Female |  False | Thur | Lunch  |    6 |
| 29.85 | 5.14 | Female |  False | Sun  | Dinner |    5 |
| 48.17 | 5.00 | Male   |  False | Sun  | Dinner |    6 |
| 20.69 | 5.00 | Male   |  False | Sun  | Dinner |    5 |
| 30.46 | 2.00 | Male   |   True | Sun  | Dinner |    5 |
| 28.15 | 3.00 | Male   |   True | Sat  | Dinner |    5 |
 ```
 - awk也可以做数字类型的比较，例如，获取超过40$的菜单，且在Saturday和Sunday
```
 cat tips.csv |  awk -F, '($1 > 40.0) && ($5 ~ /S/)' |
> header -a bill,tip,sex,smoker,day,time,size | csvlook
|  bill |   tip | sex    | smoker |        day | time   | size |
| ----- | ----- | ------ | ------ | ---------- | ------ | ---- |
| 48.27 |  6.73 | Male   |  False | 0001-01-06 | Dinner |    4 |
| 44.30 |  2.50 | Female |   True | 0001-01-06 | Dinner |    3 |
| 48.17 |  5.00 | Male   |  False | 0001-01-07 | Dinner |    6 |
| 50.81 | 10.00 | Male   |   True | 0001-01-06 | Dinner |    3 |
| 45.35 |  3.50 | Male   |   True | 0001-01-07 | Dinner |    3 |
| 40.55 |  3.00 | Male   |   True | 0001-01-07 | Dinner |    2 |
| 48.33 |  9.00 | Male   |  False | 0001-01-06 | Dinner |    4 |
```
- csvsql更鲁棒
```
< tips.csv csvsql --query "SELECT * FROM stdin WHERE bill > 40 AND day LIKE '%S%'" | csvlook -I
| bill  | tip  | sex    | smoker | day | time   | size |
| ----- | ---- | ------ | ------ | --- | ------ | ---- |
| 48.27 | 6.73 | Male   | 0      | Sat | Dinner | 4    |
| 44.3  | 2.5  | Female | 1      | Sat | Dinner | 3    |
| 48.17 | 5    | Male   | 0      | Sun | Dinner | 6    |
| 50.81 | 10   | Male   | 1      | Sat | Dinner | 3    |
| 45.35 | 3.5  | Male   | 1      | Sun | Dinner | 3    |
| 40.55 | 3    | Male   | 1      | Sun | Dinner | 2    |
| 48.33 | 9    | Male   | 0      | Sat | Dinner | 4    |
```

$ < names-comma.csv  csvsql --query "SELECT id, first_name || ' ' || last_name AS full_name, born FROM stdin" | csvlook -I
| id | full_name             | born |
| -- | --------------------- | ---- |
| 1  | John Williams         | 1932 |
| 2  | Danny Elfman          | 1953 |
| 3  | James Horner          | 1953 |
| 4  | Howard Shore          | 1946 |
| 5  | Hans Zimmer           | 1957 |
| 6  | Ludwig Beethoven, van | 1770 |
[/home/data/ch05/data]$ < names-comma.csv  Rio -e 'df$full_name <- paste(df$first_name, df$last_name);df[c("id","full_name","born")]' | csvlook -I
| id | full_name             | born |
| -- | --------------------- | ---- |
| 1  | John Williams         | 1932 |
| 2  | Danny Elfman          | 1953 |
| 3  | James Horner          | 1953 |
| 4  | Howard Shore          | 1946 |
| 5  | Hans Zimmer           | 1957 |
| 6  | Ludwig Beethoven, van | 1770 |

当我们想要提取的值分布在不同的列时，我们需要合并这些列。常见的例如：时间或者名字。假设有以下一个csv文件，现在需要将名和姓合并
```
$ < names.csv csvlook -I
| id | last_name | first_name | born |
| -- | --------- | ---------- | ---- |
| 1  | Williams  | John       | 1932 |
| 2  | Elfman    | Danny      | 1953 |
| 3  | Horner    | James      | 1953 |
| 4  | Shore     | Howard     | 1946 |
| 5  | Zimmer    | Hans       | 1957 |
```
- 使用sed：需要两条语句，第一条替换头，第二条时一个正则，带引用的正则
```shell
$ < names.csv sed -re '1s/.*/id,full_name,born/g;2,$s/(.*),(.*),(.*),(.*)/\1,\3 \2,\4/g' | csvlook -I
| id | full_name     | born |
| -- | ------------- | ---- |
| 1  | John Williams | 1932 |
| 2  | Danny Elfman  | 1953 |
| 3  | James Horner  | 1953 |
| 4  | Howard Shore  | 1946 |
| 5  | Hans Zimmer   | 1957 |
```
- awk
```
< names.csv awk -F, 'BEGIN{OFS=","; print "id,full name,born"}''{if(NR > 1){print $1,$3" "$2,$4}}' | csvlook -I
| id | full name     | born |
| -- | ------------- | ---- |
| 1  | John Williams | 1932 |
| 2  | Danny Elfman  | 1953 |
| 3  | James Horner  | 1953 |
| 4  | Howard Shore  | 1946 |
| 5  | Hans Zimmer   | 1957 |
```
- cols + tr
```
$ cat  names.csv | cols -c first_name,last_name tr \",\" \" \" | header -r full_name,id,born | csvcut -c id,full_name,born | csvlook -I
| id | full_name     | born |
| -- | ------------- | ---- |
| 1  | John Williams | 1932 |
| 2  | Danny Elfman  | 1953 |
| 3  | James Horner  | 1953 |
| 4  | Howard Shore  | 1946 |
| 5  | Hans Zimmer   | 1957 |
```
- cscsql： 链接字符串需要使用||
```
< names.csv  csvsql --query "SELECT id, first_name || ' ' || last_name AS full_name, born FROM stdin" | csvlook -I
| id | full_name     | born |
| -- | ------------- | ---- |
| 1  | John Williams | 1932 |
| 2  | Danny Elfman  | 1953 |
| 3  | James Horner  | 1953 |
| 4  | Howard Shore  | 1946 |
| 5  | Hans Zimmer   | 1957 |
```

如果last_name包含一个逗号怎么处理呢？只有csvsql的方法能成功，其他三种会失败；另外还有一种R语言的版本,后面会继续讨论
```
$ cat names-comma.csv
id,last_name,first_name,born
1,Williams,John,1932
2,Elfman,Danny,1953
3,Horner,James,1953
4,Shore,Howard,1946
5,Zimmer,Hans,1957
6,"Beethoven, van",Ludwig,1770

$ < names-comma.csv  csvsql --query "SELECT id, first_name || ' ' || last_name AS full_name, born FROM stdin" | csvlook -I
| id | full_name             | born |
| -- | --------------------- | ---- |
| 1  | John Williams         | 1932 |
| 2  | Danny Elfman          | 1953 |
| 3  | James Horner          | 1953 |
| 4  | Howard Shore          | 1946 |
| 5  | Hans Zimmer           | 1957 |
| 6  | Ludwig Beethoven, van | 1770 |
$ < names-comma.csv  Rio -e 'df$full_name <- paste(df$first_name, df$last_name);df[c("id","full_name","born")]' | csvlook -I
| id | full_name             | born |
| -- | --------------------- | ---- |
| 1  | John Williams         | 1932 |
| 2  | Danny Elfman          | 1953 |
| 3  | James Horner          | 1953 |
| 4  | Howard Shore          | 1946 |
| 5  | Hans Zimmer           | 1957 |
| 6  | Ludwig Beethoven, van | 1770 |
```

#### 5.5.4  将多个csv文件结合

- 垂直连接
先使用一个工具fieldsplit，将原先使用的csv拆分一下( the delimiter (-d), that we want to keep the header in each file (-k), the column whose values dictate the possible output files (-F), the relative output path (-p), and the filename suffix (-s))
```
$ < iris.csv fieldsplit -d, -k -F species -p . -s .csv
$ ls -lrt
total 248
-rw-r--r-- 1 root root  60758 Sep 11  2018 wiki.html
lrwxrwxrwx 1 root root     22 Sep 11  2018 users.json -> ../../.data/users.json
lrwxrwxrwx 1 root root     20 Sep 11  2018 tips.csv -> ../../.data/tips.csv
-rw-r--r-- 1 root root    129 Sep 11  2018 names.csv
-rw-r--r-- 1 root root    160 Sep 11  2018 names-comma.csv
-rw-r--r-- 1 root root    179 Sep 11  2018 irismeta.csv
lrwxrwxrwx 1 root root     20 Sep 11  2018 iris.csv -> ../../.data/iris.csv
-rw-r--r-- 1 root root 167518 Sep 11  2018 alice.txt
-rw-r--r-- 1 root root   3216 Jan  5 06:14 Iris-virginica.csv
-rw-r--r-- 1 root root   3316 Jan  5 06:14 Iris-versicolor.csv
-rw-r--r-- 1 root root   2916 Jan  5 06:14 Iris-setosa.csv
```
需要cat命令即可完成，但是要注意删除除第一个文件外的头
```
 $ cat Iris-setosa.csv <(< Iris-versicolor.csv header -d)  <(< Iris-virginica.csv header -d) | sed -n '1p;49,54p' | csvlook
| sepal_length | sepal_width | petal_length | petal_width | species     |
| ------------ | ----------- | ------------ | ----------- | ----------- |
| 4.6          | 3.2         | 1.4          | 0.2         | Iris-setosa |
| 5.3          | 3.7         | 1.5          | 0.2         | Iris-setosa |
| 5.0          | 3.3         | 1.4          | 0.2         | Iris-setosa |
| sepal_length | sepal_width | petal_length | petal_width | species     |
| 5.1          | 3.5         | 1.4          | 0.2         | Iris-setosa |
| 4.9          | 3.0         | 1.4          | 0.2         | Iris-setosa |
```
更简单的方式是使用csvstack
```
csvstack Iris-*.csv | sed -n '1p;49,54p' |  csvlook
| sepal_length | sepal_width | petal_length | petal_width | species         |
| ------------ | ----------- | ------------ | ----------- | --------------- |
|          4.6 |         3.2 |          1.4 |         0.2 | Iris-setosa     |
|          5.3 |         3.7 |          1.5 |         0.2 | Iris-setosa     |
|          5.0 |         3.3 |          1.4 |         0.2 | Iris-setosa     |
|          7.0 |         3.2 |          4.7 |         1.4 | Iris-versicolor |
|          6.4 |         3.2 |          4.5 |         1.5 | Iris-versicolor |
|          6.9 |         3.1 |          4.9 |         1.5 | Iris-versicolor |
```
可以添加一列，基于文件名
```
$ csvstack Iris-*.csv -n origin --filenames | sed -n '1p;49,54p' |  csvlook
| origin              | sepal_length | sepal_width | petal_length | petal_width | species         |
| ------------------- | ------------ | ----------- | ------------ | ----------- | --------------- |
| Iris-setosa.csv     |          4.6 |         3.2 |          1.4 |         0.2 | Iris-setosa     |
| Iris-setosa.csv     |          5.3 |         3.7 |          1.5 |         0.2 | Iris-setosa     |
| Iris-setosa.csv     |          5.0 |         3.3 |          1.4 |         0.2 | Iris-setosa     |
| Iris-versicolor.csv |          7.0 |         3.2 |          4.7 |         1.4 | Iris-versicolor |
| Iris-versicolor.csv |          6.4 |         3.2 |          4.5 |         1.5 | Iris-versicolor |
| Iris-versicolor.csv |          6.9 |         3.1 |          4.9 |         1.5 | Iris-versicolor |
```
- 水平连接
假设一个csv按列拆分，分到不同的文件，现在如何合并呢
```
$ < tips.csv csvcut -c bill,tip | tee bills.csv | head -n 3| csvlook -I
| bill  | tip  |
| ----- | ---- |
| 16.99 | 1.01 |
| 10.34 | 1.66 |
$ < tips.csv csvcut -c day,time | tee datetime.csv | head -n 3| csvlook -I
| day | time   |
| --- | ------ |
| Sun | Dinner |
| Sun | Dinner |
$ < tips.csv csvcut -c sex,smoker,size | tee customer.csv | head -n 3| csvlook -I
| sex    | smoker | size |
| ------ | ------ | ---- |
| Female | No     | 2    |
| Male   | No     | 3    |
```
直接使用paste即可
```
$ paste -d, {bills,customer,datetime}.csv | head | csvlook -I
| bill  | tip  | sex    | smoker | size | day | time   |
| ----- | ---- | ------ | ------ | ---- | --- | ------ |
| 16.99 | 1.01 | Female | No     | 2    | Sun | Dinner |
| 10.34 | 1.66 | Male   | No     | 3    | Sun | Dinner |
| 21.01 | 3.5  | Male   | No     | 3    | Sun | Dinner |
| 23.68 | 3.31 | Male   | No     | 2    | Sun | Dinner |
| 24.59 | 3.61 | Female | No     | 4    | Sun | Dinner |
| 25.29 | 4.71 | Male   | No     | 4    | Sun | Dinner |
| 8.77  | 2.0  | Male   | No     | 2    | Sun | Dinner |
| 26.88 | 3.12 | Male   | No     | 4    | Sun | Dinner |
| 15.04 | 1.96 | Male   | No     | 2    | Sun | Dinner |
```
- Joining
有时候不仅需要水平或者垂直链接，很多数据是分开存储的，我们希望像关系型数据库那样，做几个表的联合查询
假设除了iris数据，还有一个数据存储的是species和usdaid的对应关系，我们希望将这两份数据联合查询,csvjion可以完成这个任务
```
$ csvlook irismeta.csv
| species         | wikipedia_url                                | usda_id |
| --------------- | -------------------------------------------- | ------- |
| Iris-versicolor | http://en.wikipedia.org/wiki/Iris_versicolor | IRVE2   |
| Iris-virginica  | http://en.wikipedia.org/wiki/Iris_virginica  | IRVI    |
| Iris-setosa     |                                              | IRSE    |

$ csvjoin -c species iris.csv irismeta.csv | csvcut -c sepal_length,sepal_width,species,usda_id | sed -n '1p;49,54p' | csvlook -I
| sepal_length | sepal_width | species         | usda_id |
| ------------ | ----------- | --------------- | ------- |
| 4.6          | 3.2         | Iris-setosa     | IRSE    |
| 5.3          | 3.7         | Iris-setosa     | IRSE    |
| 5.0          | 3.3         | Iris-setosa     | IRSE    |
| 7.0          | 3.2         | Iris-versicolor | IRVE2   |
| 6.4          | 3.2         | Iris-versicolor | IRVE2   |
| 6.9          | 3.1         | Iris-versicolor | IRVE2   |
```
另外csvsql肯定也可以
```
]$ csvsql --query 'SELECT i.sepal_length, i.sepal_width, i.species, m.usda_id '\
> 'FROM iris i JOIN irismeta m ON (i.species = m.species)' \
> iris.csv irismeta.csv | sed -n '1p;49,54p' | csvlook
| sepal_length | sepal_width | species         | usda_id |
| ------------ | ----------- | --------------- | ------- |
|          4.6 |         3.2 | Iris-setosa     | IRSE    |
|          5.3 |         3.7 | Iris-setosa     | IRSE    |
|          5.0 |         3.3 | Iris-setosa     | IRSE    |
|          7.0 |         3.2 | Iris-versicolor | IRVE2   |
|          6.4 |         3.2 | Iris-versicolor | IRVE2   |
|          6.9 |         3.1 | Iris-versicolor | IRVE2   |
```

### 5.6 进一步阅读
- SQL Cookbook 
- Regular Expressions
- Sed & Awk

## 第六章 管理数据工作流

Drake是一个工具，它可以让你：
- 从输入和输出，格式化数据工作流
- 执行工作流中特定的的数据流
- 使用内联代码（意思是没有额外的效率损失？）
- 从外部源存储和获取数据

### 6.1 概览

- 在Drakefile中定义工作流
- 从输入和输出的角度考虑工作流
- 构建具体的目标

### 6.2 介绍Drake

Drake组织命令的执行以及相关步骤之间的依赖，数据工作流被具体描述在一个单独的文件中，每个步骤有一个或者多个输入输出，Drake会自动解决这些步骤之间的依赖，决定哪些步骤会被执行以及用怎样的数据

### 6.3 安装Drake

Drake是用Clojure语言写的，所以需要运行在JVM上。这里因为直接使用的是docker，已经安装完成，用书中所写的方法验证下：
docker镜像中的drake使用有问题，重新安装时又遇到网站被墙，相关内容暂时无法学习

## 第七章 浏览数据

### 7.1 概览

- 检查数据的格式和属性
- 计算描述性统计信息：输出时简洁的文本
- 数据可视化


### 7.2 检查数据的格式和属性

#### 7.2.1 是否有头
```
head tips.csv
```
#### 7.2.2 检查全部数据

cat会打印全部的数据，因此最好使用less -S（-S的目的是不要自动换行wrapped）
```
less -S file.csv
< file.csv csvlook | less -S
```

#### 7.2.3 对象的名称和数据类型

- 获取对象（列的名字）
```
< iris.csv sed -e 's/,/\n/g;q'

# you can define a function in .bashrc
names () { sed -e 's/,/\n/g;q'; }

# and then you can use:
< investments2.csv names
```
- 如果我们还想打印出数据类型的话，可以使用csvsql。如果直接使用csvsql不加任何参数的话，会打印出创建这样一张表所需要的语句
```
$ csvsql datatypes.csv
CREATE TABLE datatypes (
        a DECIMAL NOT NULL,
        b DECIMAL,
        c BOOLEAN NOT NULL,
        d VARCHAR NOT NULL,
        e TIMESTAMP,
        f DATE,
        g VARCHAR
);

```


#### 7.2.5 唯一标识符/连续变量/因子

为了识别那个是唯一标识符/那个是类型变量（在R中叫因子），你可以计算下某些列的值去重后的个数
```
$ cat iris.csv | csvcut -c species | body "sort | uniq | wc -l"
species
3
```
更好的方法是使用csvstate，统计每一列的唯一值个数
```
[/home/data/ch07/data]$ csvstat investments2.csv  --unique
  1. company_permalink: 27336
  2. company_name: 27322
  3. company_category_list: 8758
  4. company_market: 442
  5. company_country_code: 150
  6. company_state_code: 146
  7. company_region: 1079
  8. company_city: 3302
  9. investor_permalink: 11174
 10. investor_name: 11133
 11. investor_category_list: 463
 12. investor_market: 131
 13. investor_country_code: 106
 14. investor_state_code: 75
 15. investor_region: 545
 16. investor_city: 1197
 17. funding_round_permalink: 41793
 18. funding_round_type: 13
 19. funding_round_code: 15
 20. funded_at: 3595
 21. funded_month: 294
 22. funded_quarter: 120
 23. funded_year: 34
 24. raised_amount_usd: 6146
```
如果某一列的唯一值个数和行数一行多，他就可以当作一个唯一值；否则只能当作类别（因子）

### 7.3 计算描述性统计信息

#### 7.3.1 csvstat

csvstat可以给出以下信息：
- 每一列的数据类型（在python中的数据类型）
- 是否有控制
- 唯一值的个数
- 各种统计指标（最大值，最小值，和，平均值，标准偏差，中位数等）
```
$ csvstat datatypes.csv
  1. "a"

        Type of data:          Number
        Contains null values:  False
        Unique values:         3
        Smallest value:        2
        Largest value:         66
        Sum:                   110
        Mean:                  36.667
        Median:                42
        StDev:                 32.332
        Most common values:    2 (1x)
                               42 (1x)
                               66 (1x)

  2. "b"

        Type of data:          Number
        Contains null values:  True (excluded from calculations)
        Unique values:         3
        Smallest value:        0
        Largest value:         3.142
        Sum:                   3.142
        Mean:                  1.571
        Median:                1.571
        StDev:                 2.221
        Most common values:    0 (1x)
                               3.142 (1x)
                               None (1x)

  3. "c"

        Type of data:          Boolean
        Contains null values:  False
        Unique values:         2
        Most common values:    False (2x)
                               True (1x)

  4. "d"

        Type of data:          Text
        Contains null values:  False
        Unique values:         3
        Longest value:         8 characters
        Most common values:    "Yes!" (1x)
                               Oh, good (1x)
                               2198 (1x)

  5. "e"

        Type of data:          DateTime
        Contains null values:  True (excluded from calculations)
        Unique values:         3
        Smallest value:        2011-11-11 11:00:00
        Largest value:         2014-09-15 00:00:00
        Most common values:    2011-11-11 11:00:00 (1x)
                               2014-09-15 00:00:00 (1x)
                               None (1x)

  6. "f"

        Type of data:          Date
        Contains null values:  True (excluded from calculations)
        Unique values:         3
        Smallest value:        1970-12-06
        Largest value:         2012-09-08
        Most common values:    2012-09-08 (1x)
                               1970-12-06 (1x)
                               None (1x)

  7. "g"

        Type of data:          Text
        Contains null values:  True (excluded from calculations)
        Unique values:         3
        Longest value:         7 characters
        Most common values:    12:34 (1x)
                               0:07 PM (1x)
                               None (1x)

Row count: 3
```
可以指定，给出以下指标
- `--max`
- `--min`
- `--sum`
- `--mean`
- `--median`
- `--stdv`
- `--nulls`
- `--unique`
- `--freq`
- `--len`
例如：
```
 csvstat datatypes.csv --null
  1. a: False
  2. b: True
  3. c: False
  4. d: False
  5. e: True
  6. f: True
  7. g: True
```
可以通过-c参数指定要分析的列
```
$ csvstat datatypes.csv --null -c 1
False
```
但是要注意，就跟cscsql一样，他用的启发式推断可能不准确，一定要手动再确认。还有一个不错的技巧是通过她获取数据的行数
```
$ csvstat iris.csv | tail -n 1
Row count: 150
```

#### 7.3.1 使用Rio

Rio是一个轻量级的精巧包装过的R语言运行环境。R语言是一个非常有用的统计语言，是解释语言，丰富的扩展包，提供REPL环境。但是，R是一个独立的环境，
跟命令行工具是隔离开的，所以你无法使用管道pipe等技巧。例如你只是想计算tips.csv中的tip比率，为此你先唤起R运行环境，然后读入，然后保存到一个文件再退出
```
R
> tips <- read.csv('tips.csv', header = T, sep = ',', stringsAsFactors = F)
> tips.percent <- tips$tip / tips$bill * 100
> cat(tips.percent, sep = '\n', file = 'percent.csv')
> q("no")
```
以上一个很简单的过程，但是非常麻烦。因此，Rio就出现了，Rio代表 R input/output,你可以再命令行中使用R语言。你可以直接将csv文件通过管道传递给它，然后直接使用R语言(在我本地，产生的结果包含\n的转义字符，所以使用echo -e处理)
```
echo -e ` < tips.csv Rio -e 'df$tip / df$bill * 100' `
5.944673
16.05416
```
可以计算均值/方差/和等
```
$ < iris.csv Rio -e 'mean(df$sepal_length)'
5.843333[/home/data/ch07/data]$ < iris.csv Rio -e 'sd(df$sepal_length)'
0.8280661[/home/data/ch07/data]$ < iris.csv Rio -e 'sum(df$sepal_length)'
876.5
```
计算某一列的5个统计指标
```
< iris.csv Rio -e 'summary(df$sepal_length)'
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
  4.300   5.100   5.800   5.843   6.400   7.900
```
计算分布的对称性和峰值（这两个概念没太懂）,计算两个值之间的相关性和矩阵
```
< tips.csv Rio -e 'cor(df$bill,df$tip)'
0.6757341

 < tips.csv  csvcut -c bill,tip | Rio -f cor | csvlook
|   bill |    tip |
| ------ | ------ |
| 1.000… | 0.676… |
| 0.676… | 1.000… |

```
可以创建一个命令行格式的分布统计图
```
 < iris.csv Rio -e 'stem(df$sepal_length)'
  The decimal point is 1 digit(s) to the left of the |
  42 | 0
  44 | 0000
  46 | 000000
  48 | 00000000000
  50 | 0000000000000000000
  52 | 00000
  54 | 0000000000000
  56 | 00000000000000
  58 | 0000000000
  60 | 000000000000
  62 | 0000000000000
  64 | 000000000000
  66 | 0000000000
  68 | 0000000
  70 | 00
  72 | 0000
  74 | 0
  76 | 00000
  78 | 0
```

### 7.4 数据可视化

#### 7.4.1 Gnuplot 和 Feedgnuplot

feedgnuplot可以读取流数据
```
while true; do echo $RANDOM; done | sample -d 10 | feedgnuplot --stream \
> --terminal 'dumb 80,25' --lines --xlen 10

  35000 +-------------------------------------------------------------------+
        |      +            +             +             +            +      |
        |      :            :             :      *      :            :      |
  30000 |-+....:............:.............:....**.*.....:............:....+-|
        |      :            :             :  **    *    :            :      |
        |      :            :             :**      *    :            :      |
        |*     :            :             *         *   :            :      |
  25000 |-*....:............*............*:..........*..:............*....+-|
        |  *   :            **          * :           * :           *:*     |
        |   *  :           *: *         * :           * :           *:*     |
  20000 |-+..*.:...........*:..*.......*..:............*:..........*.:.*..+-|
        |     *:          * :   *     *   :             *          * : *    |
        |      *          * :   *    *    :             :*        *  :  *   |
        |      :*        *  :    *   *    :             :*        *  :   *  |
  15000 |-+....:.*.......*..:.....*.*.....:.............:.*......*...:...*+-|
        |      :  *     *   :      *      :             :  *     *   :    * |
        |      :   *    *   :             :             :  *    *    :    * |
  10000 |-+....:...*...*....:.............:.............:...*...*....:....+*|
        |      :    *  *    :             :             :    * *     :     *|
        |      :     **     :             :             :    * *     :      |
        |      +      *     +             +             +     *      +      |
   5000 +-------------------------------------------------------------------+
               98          100           102           104          106

```

#### 7.4.2 ggplot2

ggplot2始终R语言中的可视化实现。可以通过Rio很方便的产生各类图表。Rio期望数据是csv格式，将环境中自带的数据先转化一把
```
head immigration.dat
# IMMIGRATION BY REGION AND SELECTED COUNTRY OF LAST RESIDENCE
#
Region  Austria Hungary Belgium Czechoslovakia  Denmark France  Germany Greece  Ireland Italy   Netherlands     Norway       Sweden  Poland  Portugal        Romania Soviet_Union    Spain   Switzerland     United_Kingdom  Yugoslavia  Other_Europe     TOTAL
1891-1900       234081  181288  18167   -       50231   30770   505152  15979   388416  651893  26758   95015   226266       96720   27508   12750   505290  8731    31179   271538  -       282     3378014
1901-1910       668209  808511  41635   -       65285   73379   341498  167519  339065  2045877 48262   190505  249534       -       69149   53008   1597306 27935   34922   525950  -       39945   7387494
1911-1920       453649  442693  33746   3426    41983   61897   143945  184201  146181  1109524 43718   66395   950744813    89732   13311   921201  68611   23091   341408  1888    31400   4321887
1921-1930       32868   30680   15846   102194  32430   49610   412202  51084   211234  455315  26948   68531   97249227734  29994   67646   61742   28958   29676   339570  49064   42619   2463194
1931-1940       3563    7861    4817    14393   2559    12623   144058  9119    10973   68028   7150    4740    396017026    3329    3871    1370    3258    5512    31572   5835    11949   377566
1941-1950       24860   3469    12189   8347    5393    38809   226578  8973    19789   57661   14860   10100   106657571    7423    1076    571     2898    10547   139306  1576    8486    621147
1951-1960       67106   36637   18575   918     10984   51121   477765  47608   43362   185491  52277   22935   216979985    19588   1039    671     7894    17675   202824  8225    16350   1325727
```
使用如下命令转化(删除#开始的行，将tab替换为逗号，替换空值，替换列名Region)
```
 < immigration.dat  sed -re '/^#/d;s/\t/,/g;s/,-,/,0,/g;s/Region/''Period/' | tee immigration.csv | head | cut -c1-80
Period,Austria,Hungary,Belgium,Czechoslovakia,Denmark,France,Germany,Greece,Irel
1891-1900,234081,181288,18167,0,50231,30770,505152,15979,388416,651893,26758,950
1901-1910,668209,808511,41635,0,65285,73379,341498,167519,339065,2045877,48262,1
1911-1920,453649,442693,33746,3426,41983,61897,143945,184201,146181,1109524,4371
1921-1930,32868,30680,15846,102194,32430,49610,412202,51084,211234,455315,26948,
1931-1940,3563,7861,4817,14393,2559,12623,144058,9119,10973,68028,7150,4740,3960
1941-1950,24860,3469,12189,8347,5393,38809,226578,8973,19789,57661,14860,10100,1
1951-1960,67106,36637,18575,918,10984,51121,477765,47608,43362,185491,52277,2293
1961-1970,20621,5401,9192,3273,9201,45237,190796,85969,32966,214111,30606,15484,

< immigration.csv csvcut -c Period,Denmark,Netherlands,Norway,Sweden | head | csvlook
| Period    | Denmark | Netherlands |  Norway |  Sweden |
| --------- | ------- | ----------- | ------- | ------- |
| 1891-1900 |  50,231 |      26,758 |  95,015 | 226,266 |
| 1901-1910 |  65,285 |      48,262 | 190,505 | 249,534 |
| 1911-1920 |  41,983 |      43,718 |  66,395 |  95,074 |
| 1921-1930 |  32,430 |      26,948 |  68,531 |  97,249 |
| 1931-1940 |   2,559 |       7,150 |   4,740 |   3,960 |
| 1941-1950 |   5,393 |      14,860 |  10,100 |  10,665 |
| 1951-1960 |  10,984 |      52,277 |  22,935 |  21,697 |
| 1961-1970 |   9,201 |      30,606 |  15,484 |  17,116 |

```
这一部分内容就这样吧。因为没有图形化界面，无法直接使用display显示图形







