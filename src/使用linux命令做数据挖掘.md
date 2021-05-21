# 使用linux命令做数据挖掘
*本文内容主要来自互联网：http://teaching.idallen.com/*

因为linux（unix）包含丰富的命令行工具，所以做一些文本解析、文本格式转化等很方便。数据挖掘中很常见的流程是处理文本流并从中解析相关字段。一些命令用于选择需要处理的行而另一些命令用于解析行中的相关字段，这两个流程常常重复、交替地执行（想想管道pipe的特性），直到获取到我们想要的数据。主要有以下类型的工具：
- 从文本流中选择行：grep, awk, sed, head, tail, look, uniq, comm, diff
- 从行中选择字段：awk, sed, cut
- 转换文本（替换字符等）：awk, sed, cut

使用管道，一点一点添加命令，然后观察结果，就会发现其实数据挖掘很简单

## 案例1 输出第5行或者第5个字段

输出`PATH`变量中的第五个路径。可以衍生为打印任意输入流的第五行或者第五个字段。同样的诉求，可以使用多种不同的方法达成：

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
