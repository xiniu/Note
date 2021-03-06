# 正则表达式学习
https://regexone.com/

## 1.1 ABCs

正则表达式就是一个匹配模式，去匹配一个具体的字符序列
 abc 可以匹配abcdefg abcde abc
 
 ## 1.2 123S
 
 \d可以匹配一个0-9的数字
 
 ## 2 .匹配任意字符
 
 .可以匹配任意一个单字符；如果你确实要匹配一个“.”；你需要转义\.
 
 ## 3 匹配具体的某些字符[]
 
 [abc]匹配一个单一的a,b,c
 
 ## 4 不匹配具体的某些字符[^]或者说排除具体的字符

[^abc]匹配任何除了abc之外的单一字符

## 5 字符序列

当使用[][^]可以使用-指定字符序列，[0-6]匹配人任何0-6的单个字符；[^n-p]匹配任何除了n-p的单一字符；注意大小写敏感.
要注意单纯的[]中不管写多少字符，都只匹配一个字符

## 6 重复字符{num} {nums1,nums2}

{num}前面的字符匹配num次，{nums1,nums2}匹配nums1-nums2次

## 7 一次或者多次 +

* 代表0次或者多次
+ 代表 1次或者多次

## 8 ？0次或者1次

这个的意思是可选的，即可以出现1个也可以不出现，如果徐娅匹配？本身需要转义\?

## 9 空格

空格“ ” tab(\t) 换行（\n）以及回车(\r--windows环境下)
\s可以匹配上述任意一种类型的空格

## 10 开始和结束

^代表开始，$代表结束；注意 使用[^]时含义是不一样的

## 11 组

括号内部的字符会被捕捉为一个组，捕捉到后进一步被处理。例如匹配图片，并且捕捉不带扩展名的文件名
^(IMG\d+)\.png$
捕捉pdf文件的文件名
(.+)\.pdf$


## 12 嵌套的组

可以用前台括号捕捉嵌套的组，顺序是开括号的顺序
使用^(IMG(\d+))\.png$捕捉文件名以及里面的数字序号

## 13 更多的组

可以将* + {m,n}等用到组上。例如，在一个电话号码可能有地区码 也可能没有，那么就可以写如下的正则
(\d{3})?

## 14 条件（逻辑OR）
建议在写正则时写的更精确，这句话的意思时将已有/固定的信息要写出来
Buy more (milk|bread|juice)
可以在逻辑OR种继续使用前面哪些匹配字符：例如([cb]ats*|[dh]ogs?)能匹配 cats，bats或者dogs hogs

## 15 其他特殊的字符

已经学习过的： 数字\d 空格\s 字母或者数字\w
使用这些字符的大写来代表相反的意思， 非数字\D 非空格\S 不是字母且不是数组\W
\b用于捕捉单词之间的分界，可以是空格或者结尾,例如匹配单词\w+\b
回溯引用：你可以去引用你捕捉的组\0代表匹配到的整体串，\1代表第一个组 \2代表第二个组，例如，文本编辑器的查找替换功能：(\d+)-(\d+)然后替换为\2-\1


# 进阶

## 匹配数字

考虑小数点/科学计数法/正负号等甚至分隔符（例如,）
我自己写的：^[\+-]?[\d,]+(\.\d[\d,]*(e\d+)?)?\d*$；可能存在问题（，一般不是乱放的，一般0除非本身是0，不会放首位）

## 匹配电话号码

电话号码的构成：地区码，国家码，空格或者-

1?[\s-]?\(?(\d{3})\)?[\s-]?\d{3}[\s-]?\d{4}

## 匹配邮箱地址

([\w\.]+)(\+\w+)?@hogwarts\.(eu\.)?com
标准答案：标准答案似乎只在意结果，处理的其实并不好
^([\w\.]*)

## 匹配HTML

捕获属性
<(\w+)

捕获标签
>([\w\s]*)<

## 匹配特定的文件名

匹配图像
(\w+)\.(jpg|gif|png)$

## 剔除文本开头和结尾的空格

^\s*(.*)\s*$

## 从日志文件解析信息

(\w+)\((\w+\.java):(\d+)\)

## 解析URL

^(\w+)://([\w\.-]+)(:(\d+))?

# regex101
https://regex101.com/quiz/1 

## 1 Check if a string contains the word word in it

`/\bword\b/i`  注意这个网站需要指明大小写敏感i

## 2 Use substitution to replace every occurrence of the word i with the word I

匹配方法：`/(?:\bi\b|i')/ig`
匹配加替换：`/s/(?:\bi\b|i)/I/ig`

## 3 With regex you can count the number of matches. Can you make it return the number of uppercase consonants (B,C,D,F,..,X,Y,Z) in a given string

`/[B-DF-HJ-NP-TV-Z]/g`

##  4 Count the number of integers in a given string. Integers are, for example: 1, 2, 65, 2579, etc.

`/(?:\d+)/g`

## 5 Find all occurrences of 4 or more whitespace characters in a row throughout the string.

自己的答案貌似不太好`/(?: {4,}+| *\t+[ \t]*)/g`





