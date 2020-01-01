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
如下的命令是统计电子书《Adventures of Huckleberry Finn》中出现top10 的单词，具体解释如下：
- 使用`curl`下载
- 使用`tr`转为，将大写转小写
- 使用`grep`展开所有单词并把单词每行一个放置
- 使用`sort`用字典序排序
- 使用`uniq`移除重复行并计数
- 使用`sort`排序，按照数字顺序,注意是降序
- 使用`top`保留前10行




