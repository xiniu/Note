# 使用linux命令做数据挖掘
*本文内容主要来自互联网：http://teaching.idallen.com/*

因为linux（unix）包含丰富的命令行工具，所以做一些文本解析、文本格式转化等很方便。数据挖掘中很常见的流程是处理文本流并从中解析相关字段。一些命令用于选择需要处理的行而另一些命令用于解析行中的相关字段，这两个流程常常重复、交替地执行（想想管道pipe的特性），直到获取到我们想要的数据。主要有以下类型的工具：
- 从文本流中选择行：grep, awk, sed, head, tail, look, uniq, comm, diff
- 从行中选择字段：awk, sed, cut
- 转换文本（替换字符等）：awk, sed, cut

使用管道，一点一点添加命令，然后观察结果，就会发现其实数据挖掘很简单

## 案例1 输出第5行或者第5个字段
问题：输出`PATH`变量中的第五个路径。可以衍生为打印任意输入流的第五行或者第五个字段。同样的诉求，可以使用多种不同的方法达成：

- 使用`tr`, `head`, `tail`
首先简单看一下`PATH`的内容：
```shell
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```
可以发现都是冒号分隔的路径，想办法将路径输出为每一行一个，使用`tr`命令可以达到目的
```shell
echo $PATH | tr ':' '\n'
/usr/local/sbin
/usr/local/bin
/usr/sbin
/usr/bin
/sbin
/bin
/usr/games
/usr/local/games
/snap/bin
```
为了打印第五行，我们可以`head -n `打印前五行，然后使用`tail -n`打印最有一行
```shell
echo $PATH | tr ':' '\n' | head -5 | tail -1
/sbin
```
如下形式也可以
```shell
echo $PATH | tr ':' '\n' | head -n 5 | tail -n 1
/sbin
```
- 使用`awk`
awk可以用于解析文本字段
- `awk '{print $n}'`打印每行第n个字段
- `awk '{print $NF}'`打印每行最后一个字段
awk默认使用空白作为字段分隔，所以先使用tr命令将冒号转为空格，然后在使用awk可以完成任务：
```shell
echo $PATH | tr ':' ' ' | awk '{print $5}'
/sbin
```
但是其实可以更简单，awk有一个很方便的参数可以指定分隔符
```shell
echo  $PATH | awk -F:  '{print $5}'
/sbin
```
- 使用`cut`
cut可以指定任何分隔符分隔文本，可以使用如下命令完成任务：
```shell
echo  $PATH | cut -d: -f5
/sbin
```
- 使用sed
```shell
echo  $PATH | sed -e 's/^[^:]*:[^:]*:[^:]*:[^:]*:\([^:]*\):.*/\1/'
/sbin
```

## 案例2 输出倒数第2行或者第2个字段
问题：输出`PATH`变量中的倒数第2个路径。可以衍生为打印任意输入流的倒数第2行或者第2个字段。

- 使用`tr`, `head`, `tail`
```shell
echo $PATH | tr ':' '\n' | tail -2 | head -1
/usr/local/games
```
或者
```shell
echo $PATH | tr ':' '\n' | tail -n 2 | head -n 1
/usr/local/games
```

- 使用`awk`
注意下括号。
```shell
echo $PATH | tr ':' ' ' | awk '{print $(NF-1)}'
/usr/local/games
```
```shell
echo  $PATH | awk -F:  '{print $(NF-1)}'
/usr/local/games
```

- 使用sed
```shell
echo  $PATH | sed -e 's/.*:\([^:]*\):[^:]*$/\1/'
/usr/local/games
```

## 案例3 排序多个行或者字段
问题：将`PATH`变量中的路径进行排序后输出
`sort`命令可以排序行，所以这里还是先转化成多行，然后排序然后再转换回去。
```shell
echo  $PATH | tr ':' '\n' | sort | tr '\n' ':'
/bin:/sbin:/snap/bin:/usr/bin:/usr/games:/usr/local/bin:/usr/local/games:/usr/local/sbin:/usr/sbin:
```
可以发现处理后，最后一行多了一个冒号，可以使用sed去除
```shell
echo  $PATH | tr ':' '\n' | sort | tr '\n' ':'
/bin:/sbin:/snap/bin:/usr/bin:/usr/games:/usr/local/bin:/usr/local/games:/usr/local/sbin:/usr/sbin:
```

## 案例4 只保留前5个行或者字段
问题：保留`PATH`变量中前5个元素。
可以使用`head`命令保留前n行，所以这里还是跟案例3相同的思路
```shell
echo  $PATH | tr ':' '\n' | head -5 | tr '\n' ':' | sed 's\:$\\'
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin
```

## 案例5 对字段计数
问题：计算在`/etc/passwd`中有多少中不同的shell
首先，观察该文件，可以发现shell信息存储在每行的第7个字段
```shell
head -2 /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
```
任意使用之前的方法可以获取第7个字段，然后使用`sort`、`uniq`排序去重和在使用`wc`统计即可
```shell
cut -d: -f7 /etc/passwd | sort | uniq | wc -l
4
```
可以只用使用sort -u参数，在排序时就已经去重
```shell
cut -d: -f7 /etc/passwd | sort -u | wc -l
4
```

## 案例7 从web页面解析数据
问题：从http://weather.gc.ca/rss/city/on-118_e.xml 解析Ottawa的天气。
有多种方法可以下载到网页文件。但是经过比较还是`elinks`工具导出的文本更好处理
```shell
url='http://weather.gc.ca/rss/city/on-118_e.xml'
wget -O wget.txt "$url"
# raw XML RSS page downloads here into wget.txt file...
lynx -dump "$url" >lynx.txt
# formatted web page is in lynx.txt...
alias ee='elinks -dump -no-numbering -no-references'
ee "$url" >elinks.txt
## formatted web page is in elinks.txt...
```
文件中温度在`Temperature:`起始的一行（grep：使用Basic Regular Expressions正则，egrep使用Extended Regular Expression，fgrep用于搜索、不关注正则）。
```shell
ee "$url" | grep 'Temperature:'
   Temperature: 23.8°C
```
如果只想提取温度值，可以用awk继续处理下
```shell
ee "$url" | grep 'Temperature:' | awk '{print $NF}'
24.5°C
```

## 一些其他技巧
- 过滤包含一组string内所有string的行
```shell
fgrep 'string one' filename | fgrep 'string two' 
```
- 包含一组string内任意一个string的行
```shell
fgrep -e 'string one' -e 'string two' -e 'string three'...
```
- 寻找匹配两个模式之间的行
grep -A表示除了打印匹配行还要打印之后的xx行；grep -B自然可以猜到
```shell
... | grep -A 1000 'start patterns' | grep -B 1000 'end patterns'
```
sed也可以达到目的
```shell
... | sed -n -e 's/start patterns/,/end patterns/p'
```
