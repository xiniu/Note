# 第一章 穿越时空的迷雾

## C语言史前阶段

贝尔实验室的Ken Thompson和 Dennis Ritchie发明了C和Unix

## 预处理器

宏最好只用于常量命名。反面教材是Bourne创建的C语言的变种，事实上促进了 C语言混乱代码大赛

```c
#define STRING      char*
#define IF          if (
#define THEN        ){
#define ELSE        }else(
#define FI          ;}
#define WHILE       while(
#define DO          ){
#define OD          ;}
#define INT         int
#define BEGIN       {
#define END         }

INT compare(STRING s1, STRING s2)
BEGIN
    WHILE *s1++ == *s2
    DO IF *s2++ == 0
        THEN return(0)
       FI
    OD
    return (*--s1 - *s2);
EN
```
预编译（`gcc algolTest.c  -E -o algolTest.i`）后可以看到对应的C语言源码
```c
int compare(char* s1, char* s2)
{
    while( *s1++ == *s2
    ){ if ( *s2++ == 0
        ){ return(0)
       ;}
    ;}
    return (*--s1 - *s2);
}
```

## K & R C 和 ANSI C

- 1978年， 作者Brian Kernighan和Dennis Ritchie 出版的经典名著《The C Programming Language》，这个版本的C语言被称为 **K & R C**
- 1990年， ANSI（美国国家标准化组织）接纳了ISO C。因此从原则上讲，ANSI C就是ISO C。我们平时所说的C语言标准也是ISO C

## 可移植代码
严格遵循标准：
- 只使用已确定的特性
- 不突破任何由编译器实现的限制
- 不产生任何以来编译器定位的或未确定的或未定义特性的输出
严格遵循标准的代码在任何平台上运行都会产生相同的输出。
下面这个程序就不是严格遵循标准的代码
```c
#include<limits.h>
#include <stdio.h>

int main(void) {
  (void)printf("biggest int is %d\n", INT_MAX);
  return 0;
}
```
