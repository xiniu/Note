# Linux学习积累

## The Art of the Command line

来自[the art of the command line](https://github.com/jlevy/the-art-of-command-line/blob/master/README-zh.md)

### 1 基础
1. `man bash`全文浏览---待完成，英语比较耗时
2. 文本编辑器vi使用
```bash
# 基础的使用我是会的
#I 进入插入模式,从当前位置输入； a 进入插入模式，从下一位置出入；o 就进入插入模式，新起一行 
#Esc 退出插入模式
#： 底行模式 用于保存或者推出
# dd 删除一行 -命令模式
# 2dd 删除两行 命令模式
# G 移动到末尾行 命令模式
# x 删除光标后面的一个字符 X 光标前的一个字符 命令模式
# yy 复制当前行 命令模式
# 6yy 复制6行 命令模式
# p 粘贴 命令模式
# r 替换一个字符 R 一直替换直到esc 命令模式
# u 撤销 命令模式
# ctr+g 显示光标所在的行 命令模式
# 15G 跳转到15行 命令模式
# ：set nu 底行模式列出行号 底行模式
# :7 跳转到第七行 底行模式
#：/XX 往后查找 可以按n一直查找 底行模式
# : ?xx 往前查找 可以按n一直查找 底行模式
```
3. 使用man阅读文档 apropos查找文档 type可以列出一个命令到底是内置命令还是别名
```
type ls
ls 是 `ls --color=auto' 的别名
```
4. 重内向覆盖 < >， 重内向追加 << >> 以及管道 stdout stderr

| handle | meaning|
|------- | ------- |
| 0 | stdin |
| 1 | stdout |
| 2 | stderr|

```bash
program-name 2> error.log  # redirection stderr to a file
command > file-name 2>&1 #  redirection stderr and stdout  to a file
command-name 2>&1 # redirection stderr to stdout
```
5. 学会使用同配符 * 和 ？ [...]，以及'和"的区别
```bash
# valid wildcard operators:
# * : any zero or more characters
# ? : any single character
# [...] : single character between brackets, ex: [A-Z]
# [^...] : single character not between brackets, ex: [A-Z]
# | : seperates alternate paterns
ls
# a.out  export.c  export.c~  narnia0  narnia0.c  narnia0.c~
ls [a-e]*
#a.out  export.c  export.c~
ls export.?
#export.c


#
```

1. 压缩与解压
```bash
tar –cvf jpg.tar *.jpg # 将目录里所有jpg文件打包成tar.jpg
tar –zcvf jpg.tar.gz *.jpg   #将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
tar –xvf file.tar #解压 tar包
tar -xzvf file.tar.gz #解压tar.gz v的意思是整个过程可视

lsb_release -a #ubuntu 查询版本号
nameserver 8.8.8.8 # 因为/etc/resolve.conf有误导致升级到14.04后无法联网.加上这句后ok
nameserver 192.168.1.1 # 因为/etc/resolve.conf有误导致升级到14.04后无法联网.加上这句后ok
chattr +i resolve.conf # 每次重启后都链接不了网络，尝试给这个加个保护
```
