# 《Data Science at the Command Line》学习

https://www.datascienceatthecommandline.com/

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

### 1.7 docker安装

这是我临时插入的一节，如何再ubuntu上安装docker
https://cloud.tencent.com/developer/article/1167995
备注：我再win10的WSL上安装ubuntu然后安装docker时报错了，因此尝试了自己安装虚拟机/购买腾讯云的方式安装，没有遇到问题。并且有钱是真好，腾讯云上的主机挺好用的

## 第二章  开始

### 2.1 概览

- 安装docker镜像（image）
- 基本的概念和工具

### 2.2 安装docker镜像

- 下载镜像
```bash
$ docker pull datascienceworkshops/data-science-at-the-command-line # dowaload a image
```
- docker镜像很慢，使用国内源试试https://cloud.tencent.com/developer/article/1335788 ,速度稍微快一点，下载完提示如下
```bash
# Status: Downloaded newer image for datascienceworkshops/data-science-at-the-command-line:latest
# docker.io/datascienceworkshops/data-science-at-the-command-line:latest
```
- 运行镜像,运行成功后命令提示符就已经变了。运行whoami提示是root用户
```bash
$ docker run --rm -it datascienceworkshops/data-science-at-the-command-line
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
执行方法,因为上面的脚本已经指定了解释器为#!/usr/bin/env python
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
- 别名：别名有点像宏，如果经常执行一个命令但是包含相同的命令字，可以使用别名；例如，针对某个特定的命令经常拼写错误，也可以使用别名
