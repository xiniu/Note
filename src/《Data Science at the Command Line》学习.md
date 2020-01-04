# 《Data Science at the Command Line》学习

原书参照这个链接：https://www.datascienceatthecommandline.com/

## 第一章 简介

### 1.2 数据科学（Data Science）就是OSEMN

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


