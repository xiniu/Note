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
# G 移动到末尾行 命令模式；NG或者:N到第Nhang；gg或者1G :1到第一行
# x 删除光标后面的一个字符 X 光标前的一个字符 命令模式
# yy 复制当前行 命令模式
# 6yy 复制6行 命令模式
# p 粘贴 命令模式
# r 替换一个字符 R 一直替换直到esc 命令模式
# u 撤销 命令模式；c+u redo
# ctr+g 显示光标所在的行 命令模式
# 15G 跳转到15行 命令模式
# ：set nu 底行模式列出行号 底行模式
# :7 跳转到第七行 底行模式
#：/XX 往后查找 可以按n一直查找 底行模式
# : ?xx 往前查找 可以按n一直查找 底行模式
# 移动 0移动到行首，^移动到第一个不是blank的问题值，$行末尾
# :w存盘，:saveas <path/to/file>另存为；:x保存（仅需要）并退出，wq保存并退出，ZZ保存并退出，不需要输入:以及回车
# 打开很多文件后进行切换 :bn :bp
# 重复，.重复上一次命令；N<cmd>执行某个命令N次（2dd删除两行，3p粘贴3次，100idesu[ESC]写100个desu，3.重复三次上次的命令）
# 按单词移动，w到下一个单词开头，e到下一个单词末尾
# %匹配括号移动，需要把光标放在括号上；*或者#匹配光标所在单词，移动下一个或者上一次匹配单词
# <start position><command><end position>  0y$意味着移动到航头，从这里开始拷贝y，到最后一个字符
# 更快移动 fa到下一个为a的字符处，ta到a前的一个字符，3fa在当前行找第三个出现的a。dt“删除内容直到双引号
# 块操作，例如批量添加注释，^移动到行头，C-V开始块操作，C-d或者j向下移动要操作的行，I--[ESC]插入--，ESC键为每一行生效
# C+p 或者C+n自动补齐
# 可视化模式，也可以给多行做一些操作，参照上面的块操作。一旦选中当行，J可以将多行合并，<和>左右缩进。=自动缩进；选中相关的行,$到行最后，输入A在输入字符串按ESC，在每行末尾添加一个字符串
# :split创建分屏
```
如下是一份vimrc配置文件，用于简单的cpp和c的学习
```bash
set number
set showcmd
set mouse=a
set encoding=utf-8
set t_Co=256
set autoindent
set tabstop=4
set expandtab
set softtabstop=4
set cursorline
set laststatus=2
set showmatch
set hlsearch
set incsearch
set wildmenu
set wildmode=longest:list,full
func SetTitleCPP()
	call setline(1,"/******************************************")
    call setline(2,"* 用途：")
    call setline(3,"* 日期：".strftime("%Y-%m-%d %H:%M:%S"))
    call setline(4,"******************************************/")
	normal G
	normal o
	normal o
endfunc
autocmd bufnewfile *.cpp call SetTitleCPP()

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

6. 压缩与解压
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

## 陈皓博客学习

### 你可能不知道的SHELL

- `!$`
代表上一个命令的最后一个字符串，是一个环境变量
```bash
mkdir test
cd !$
```
这个还有一个类似的变量，`$_`,也代表上一次命令的最后一个参数，但是使用`!$`会再次打印一遍命令，类似与多了依次echo。但是`$_`便于记忆。在bash中，
`$#`代表传入脚本的参数个数，`$1`代表第一个参数，`$?`是上次命令或者脚本退出的状态，`$@`代表所有参数，`$*`代表所有参数（将所有参数以一个单字符串体现）。

- `sudo !!` 或者 `!!`
其实可以直接理解`!!`是上条执行的整个命令

https://www.ibm.com/developerworks/cn/linux/l-bash-parameters.html

- `alt + .`
可以理解为快捷键，将之前执行历史中执行的参数回溯出来，可以不停的按（其实就是不停的找每个执行命令的最后一个单词）。这是个快捷键

- `^old^new`
替换上次命令中的old为new，如果输错了这个很有用。相同的命令为`!!:gs/old/new`,这个更容易理解，像sed或者vi的命令

- `du -s * | sort -n | tail`
统计当前文件中大小最大的10个文件或者文件夹（s代表的总计，也就是说如果是一个文件夹，只显示总的大小）。对于du要好好学习。
`du filename`显示一个文件的大小，`du dir`会递归显示dir下每个子文件夹大小，`du -s dir`只显示总大小

- `> file.txt`
创建一个空文件

- `ps aux | sort -nk +4 | tail`
流出10个最耗内存的进程，k代表的是关键字，就是以哪个关键字排序的意思

- `tr -c "[:digit:]" " " < /dev/urandom`
tr命令中的c代表的补集
