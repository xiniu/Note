# 第一章 穿越时空的迷雾


这个网上在线编译工具挺好用的：
https://repl.it/@xiniu/FriendlyExcitedSquare#main.c

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
继续考虑下述的例子，考虑为什么x没有赋值成功
```c
#include<limits.h>
#include <stdio.h>

int array[] = {5,15,20,21};
#define TOTLE_ELEMENTS  sizeof(array) / sizeof(array[0])
int main(int argc, char ** argv) {   
    int d = -1, x = 0;

    if (d < TOTLE_ELEMENTS - 1) {
        x = array[d + 1];
    }
    printf("x = %d",x); // why x = 0???
}
```

# 第二章 这不是bug，而是语言特性

## 多做之过

### fallthrough以及switch语句

switch语句的几个问题：
- 对于一个匹配都没匹配的场景不会有报错
- 语句表达式从匹配的case执行，也就是说你期望执行switch后面各个case公共的一些代码，但是这些肯定是不会执行的
- case都是可选的，且是任何形式的语句，包括带标签的语句都是合法的
- fallthrough，除非在某个case后主动break，否则从这个case后的语句都会执行
- 另外break的用法要格外注意，它跳出的是离他最近的switch或者循环语句，不会跳出if等

```c
#include<limits.h>
#include <stdio.h>

int array[] = {5,15,20,21};
#define TOTLE_ELEMENTS  sizeof(array) / sizeof(array[0])
int main(int argc, char ** argv) {   
    int d = -1, x = 0;

    if (d < TOTLE_ELEMENTS - 1) {
        x = array[d + 1];
    }
    printf("x = %d\n",x); // why x = 0???

    switch(x){
        case 0:
        {
            printf("1x = %d\n",x); // 

        } 
        case 2:
        {
            printf("2x = %d\n",x); // 

        }
        defat:
       {
            printf("are you kidding?x = %d\n",x); // compile ok?!!
        }
    }
}
```

### 相邻的字符串常量会被合并为一个字符串常量

相邻的字符串常量会被合并，哪怕在不同的行。

```
int main(){

    printf("abcde""fghi"
                    "lmn\n");
    char *available_resource[] = {
    "color monitor",
    "big disk",

    "Cray"      // here miss , by mistake
    "on-line drawing routhines",
    "mouse",
    "key board",  // does this , matter?
    };
    printf("%s\n",available_resource[2]);
}

```

### 太多的缺省可见性

定义C函数时，在缺省情况下函数的名字是全局可见的，加或者不加extern关键字都是一样的；如果想限制这个方法的访问，需要加static关键字。这种缺省的全局可见性是个巨大的错误

## 误做之过

### 重载 许多符号是被重载的，在不同的上下文环境中有不同的含义

|   符号                 |  含义   |
| -----------           | --------- |
| static        | 在函数内部，表示静态变量，在各个调用间保持一致延续性；但是在函数这一级，表示函数只对本文件可见 |
|   extern      | 用于函数定义，表示全局可见（冗余的）；用于变量，说明这个变量在其他地方定义 |
|   void        | 作为函数的返回类型，表示不返回任何值；用于指针声明，表示通用指针类型； 位于参数列表，表示没有参数 |
| \*            | 乘法运算符；用于指针，解引用；用于指针的定义    |
| &             | 位的AND操作符；用于取地址    |
| <=  <<=       | 小于等于和左移赋值 |
| <             | 小于号； include的左定界 |
| （）          | 函数定义中包含形式参数； 函数调用；改变表达式的求值次序；强制类型转化；定义带参数的宏；包围sizeof的操作数|

```c
if (x >> 4)...  // x 远远大于4？i
p =  N * sizeof * q;  // 到底有几个乘法？ 注意，sizeof后如果是变量不必加括号，如果是类型名必须加括号
app = sizeof(int) * p;  // 到底是将int长度乘以p还是将p先解指针并强制类型转为int再取sizeof？
```

### 有些运算符的优先级顺序是错误的

C语言运算符优先级

| 运算符 | 结合性 |
| ------ | ------ |
| () [] -> .| 从左至右 |
| ! ~ ++ --  + - * & (type) sizeof | 从右至左 |
| * / % |   从左至右 | 
| +    - |  从左至右 |
| <<  >>  | 从左至右 |
| < <= > >= | 从左至右 |
| == != | 从左至右 |
| &   | 从左至右 |
| ^ | 从左至右 | 
| \| | 从左至右 |
| && | 从左至右 |
| \|\| | 从左至右 |
| ?: |  从右至左 | 
| = += -+ \*= /+ %= ^= \|=  <<= >>=| 从右至左 |
| , | 从左至右 |

C语言运算符优先级存在的问题

| 问题 | 表达式 | 误解 |  实际 |
| .的优先级比\*高，建议使用->消除这个问题 | \*p.f | p所指对象的f字段 (\*p).f | p的f字段是指针，对其解引用 | 
| []高于\* | int \* ap [] | ap是个指向int数组的指针 int (\*ap)[] | ap是个数组，每个元素为int指针 int \* (ap[]) |
| int \*fp() | int \*fp()  | fp是个函数指针，所指函数返回int 即int(\*fp)() | 实际是fp是个函数，该函数返回int* | 
| == 和 != 高于位操作符 | (val & mask != 0) | (val & mask) != 0 | val & (mask != 0) |
|== 和 != 高于赋值符| c = getchar() != EOF | (c = getchar()) != EOF | c = (getchar() != EOF) | 
| 算术运算高于移位 | msb << 4 + lsb | (mab << 4) +  1sb | msb << (4 + 1sb) | 

结合性：标示同一优先级的各个运算符的先后运算顺序，所有赋值符（包括符合）都具有右结合性，表达式最右边的操作最先执行

### gets导致蠕虫病毒

因为会导致缓冲区越界，在C语言的官方手册中，建议把`gets(line)`替换位`fgets(line, sizeof(line), stdin)`

## 少做之过

### 标准参数处理

选项开关和标准文件名之间区分不清

shell参数解析中右类似的问题，
如下方式寻转链接文件均不可行 `ls -l |  grep ->` `ls -l | grep "->"`
要解决这个问题，只能使用file等工具 `file -h * | grep link`
如果创建了连字符开始的文件名，无法直接rm，一种规避方法是需要输入完整的路径名

### 空格

有些是空格必不可少

`ratio = *x / *y` 如果确实空格会导致编译错误，因为会将/\*当成注释
 
 跟错误的编写了一个注释符相比，还有一类常见的注释相关的错误是未写注释结束导致的异常

















