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

## const

> 每个实参都应该具有自己的类型，这样它的值就可以赋给与他所对应的形参类型的对象：
> 要使得上述赋值合法，必须满足下面条件之一：
> 两个操作数都是指向又限定符或者无限定符的相容类型的指针，左边指针所指向的类型必须是具有右边指针所指向类型的全部限定符。

```C
    char *cp;
    const char *ccp;
    ccp = cp
```
如上赋值是合法的，因为：
- 左操作数是一个指向有const限定符的char的指针
- 右操作数是一个指向没有限定符的char的指针
- char类型和char类型是相容的，左边操作数所指具备右边操作数所指类型的全部限定符

但是上述赋值反过来就不行。那么`char **` 和 `const char **`能否相互赋值呢？ 前者是指向 `char *`的指针，后者是指向`const char *`的指针，都是不带限定符，但是他们所指的对象不同，是不能相互赋值的。相容性不具备传递性。
这个地方有点绕，专门总结下：
```C
const char* pcc;    // point to const-char
char* pc;           // point to char
char* const cpc;    // const-point to char
const char** ppcc;   // point to (point to const-char)
char** ppc;        // point to (point to char)
char** const cppc; // const-point to point to char

pcc     =   pc;           // OK
pc      =   pcc;          // Error
cpc     =   pc;           // Error
pc      =   cpc;          // OK
cpc     =   pcc;          // Error
pcc     =   cpc;          // OK

ppcc    =   ppc;          // Error
ppc     =   ppcc;         // Error
cppc    =   ppc;          // Error
ppc     =   cppc;         // OK
cppc    =   ppcc;         // Error
ppcc    =   cppc;         // Error
```
const和\*的组合经常用在函数传参中，“我给你一个指向它的指针但是你不能修改它”

## 整数升级和算数转化

整数升级：
char short 以及int型位段，可以在必要的时候转为位int 或者 unsigned int。如果int可以完整表示源类型所有值，就用int，否则用unsiged int

算数转化：
K & R C采用的是无符号保留，当一个无符号类型与int或者更小的类型混合使用时，结果类型是无符号型。
但是ANSI C采用的值保留，参照上述整数升级说明
```C
#include<limits.h>
#include <stdio.h>

int main(int argc, char ** argv) {   
    if (-1 < (unsigned char)1) {
        printf("-1 is less then (unsigned char)1:ANSI\n");
    } else {
        printf("-1 NOT is less then (unsigned char)1:K & R C\n");
    }

    if (-1 < (unsigned int)1) {
        printf("-1 is less then (uint)1\n");
    } else {
        printf("-1 NOT is less then (uint)1\n");
    }
}
```
```
-1 is less then (unsigned char)1:ANSI
-1 NOT is less then (uint)1
```

