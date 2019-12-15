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

#### 2.3.1
