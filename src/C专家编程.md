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
