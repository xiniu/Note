# ***Xiniu*** 的学习笔记

## *Git* 基本操作
1. 克隆远程库
```bash
git clone https://github.com/xiniu/Note.git
```

2. 添加文件
```bash
git add test.txt
```

3. 提交修改
```
git commit -m somemsg
```

4. 推送到远程库：将`matser`分支推向远程库
```
git push origin matser
```
5. 察看远程库信息:当从远程库 *clone* 时自动会把本地的`master`和远程库的`master`分支对应起来，并且远程库的名称为默认`origin`。用一下命令可以察看：
```
git remote -v
```

## *Effective* *C++* 读书笔记

### 0 导读 ###

1. 中英文对照（只列举不会的）

中文 | 英文
------------| -------------
inheritance | 继承
composition | 复合
assignment  | 赋值
constructor | 构造
destructor  | 析构
declaration | 声明式
signature   | 签名式
definition  | 定义式
initialization |初始化
implicit type conversions | 隐式类型转化
explicit type conversions | 显式类型转化

2. 一些术语
- 声明式：告诉编译器某个东西的名称和类型，省略细节。
```C++
extern int x;                           //对象声明式
std::size_t numDigits(int number);      //函数声明式
class Widget;                           //类声明式

templete<typename T>
class GraphNode;                        //模版类声明式
```

- 签名式：签名可以理解为标示函数的类型，在本书中我们指定参数类型（参数签名）和返回类型。例如`numDigits`的签名是`std::size_t (int)`.需要额外注意的是，C++签名式的官方定义不包含返回类型，这也是重载函数的定义，就是只有参数签名不同
- 定义式：补充声明式遗漏的一些细节。对对象而言，定义式是划分内存的地方；对`function`和`functionc templete`而言，是实现代码逻辑本体的地方；对于`class`和`class templete`而言，是指明他们成员的地方

```C++
int x;                                  //对象定义式
std::size_t numDigits(int number)       //函数定义式
{
    //some logic code.
}
class Widget
{
public:
    Widget();
    ~Widget();
    //some code...
};                                      //类定义式
templete<typename T>
class GraphNode;                        //模版类定义式
{
public:
    GraphNode();
    
    ~GraphNode();
    //some code...
}
```

- `explicit`关键字。explicit关键字修饰构造函数，用于阻止隐式。但显式类型转化依旧是允许的。以下例子说明。在编译期间识别非预期的类型转化。
```C++
class B
{
public:
    explicit B(int x = 0, bool b = true);
};
//some code
void doSomething(B);

doSomething(28);        //error!
doSomething(B(28))      //ok!
```

- *copy*构造函数和*copy assignment*函数。*copy*构造函数用来用另一个对象初始化自我对象 ，*copy assignment*函数用来从另一个对下拷贝值到自我对象。一个是构造函数，一个是赋值操作。如果有新对象被创建，用的肯定是构造函数；如果没有新对象被创建，肯定是赋值操作。值传递方式传递参数时肯定会调用*copy*构造函数，往 STL 容器放值也应当是*copy*构造函数被调用。
```C++
class Widget
{
public:
    Widget();                               //f1
    Widget(const Widget& rhs);              //f2
    Widget& operator=(const Widget& rhs);   //f3
    //some code...
}

Widget w1;                                  //called f1
Widget w2(w1);                              //called f2;
w2 = w1;                                    //called f3;
Widget w3 = w1;                             //called f2 bcz there is new object created!
```

- 未定义或者未明确行为UB: 例如，对null指针取值，数组越界访问。

-对操作符的理解。如果声明了如下`operate*`:
```C++
const Rational operate*(const Rational& lhs, const Rational& rhs);
```
则 `a * b`理解为`operate*(a, b)`




### 1 让自己习惯C++ ###  


1. 条款 01：视 C++ 为一个语言联邦
- 中英文对照

中文 | 英文
------------| -------------
muliparadigm | 多重范型
procedure    | 过程形式
object-oriented  | 面向对象
generic | 范型
metaprogramming  | 元编程
programming paradigm | 编程范型
encapsulation   | 封装
polymorphism  | 多态

- 条款内容：将 C++ 视为一个语言的联邦，而非单一语言。主要有以下4个次语言：C；Object-Oriented C++; Temeplete C++; STL.

2. 条款 02：尽量以`const`, `enum`, `inline` 替换 `#define`
- 条款理解01：对于单纯常量，最好用`const`对象或者`enum`对象替换`#define`
    - why：对于用`#define` 定义常量的情况：`#define` 的名称未进入符号表，难以跟踪调试；使用常量会减少少量的码。
    - 代码讲解：
    ```C++
    #define ASPEC_RATIO 1.653;           //bad
    const double AspecRatio = 1.653;     //good
    
    const char* const authoName = "Scott Meyers";    //指针和所指之物都是const。
    const std::string authoName("Scott Meyers");
    
    class GamePlayer 
    {
    private:
        static const int NumTurns = 5;  //static class常量声明式
        int scores[NumTurns];
        //some code...
    
    };
    const int GamePlayer::NumTurns;     //如果要取常量地址或者编译器要求，你必须要定义。放进实现文件
    
    //旧式编译器不支持以上语法，不允许static常量声明时获得初值即in class初值设定。可以将初值放在定义式。
    class CostEstimate
    {
    private:
        static const double FudgeFactor;   //static class常量声明 放在头文件中
        //some code...
    }
    
    const static double CostEstimate::FudgeFactor = 1.35;   //static class常量定义 放在实现文件中
    
    //如果编译器不允许in class初值设定，但编译器又在编译期间必须知道初值，例如GamePlayer中的数组。此时
    //改用 the enum hack补偿做法。一个属于枚举类型的数值可充当int使用。
        
    class GamePlayer 
    {
    private:
        enum { NumTurns = 5};  //the enum hack补偿做法
        int scores[NumTurns];
        //some code...
     
    };
    ```
    
    
- 条款理解02：对于函数宏，最好用`inline`函数替换`#define`。
    - why:`#define` 实现宏函数：宏操心的事情很多，例如要注意括号的用法；并且宏会有一些异常的表现。 
    - 代码讲解：
    
    ```C++
    #define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))    //必须注意加括号，否则有明显bug
    
    int a = 5, b = 0;
    CALL_WITH_MAX(++a, b);      //a++ for 2 times
    CALL_WITH_MAX(++a, b+10);   //a++ fro 1 times
    
    
    templete<typename T>
    inline void callWithMax(const T& a, const T& b)
    {
        f(a > b ? a : b);   //类型控制const 用引用穿参数 是一个真正的函数
    }                       //much better
    ```    

3. 条款 03：尽可能使用`const`
- 条款理解01：对于变量，可以增加一个语义描述，描述该变量不可修改，用以获取编译器的帮助。主要要注意跟指针使用时，到底是指定指针自身、指针所指之物以及两者都是常量。
    - why：明确告诉编译器你的诉求，用以获取编译器的帮助；另外，在函数参数中使用时，有助于调用者判断函数的行为，例如会不会改变参数等；
    - 代码讲解
    ```C++
    char greeting[] = "Hello"
    char* p = greeting;                 //non-const pointer, non-const data
    const char* p = greeting;           //non-const pointer, const data
    char const * p = greeting;          //non-const pointer, const data
    char* const p = greeting;            //const pointer, non-const data
    //如果const出现在* 左边意味着所指指物是const。出现在* 右边代表本身是const。这块对于复杂表达式的声明，有通用的方法。
    ```


## 网络资源
1. 容器Docker
- Docker解决的问题：软件安装的时候环境配置很麻烦，最好能根据原始环境复刻出来。虚拟机(VM)是个解决方案，但是面临资源占用多，冗余步骤多，启动慢的问题。Linux容器(linux containers,lxc)应运而生，它不是模拟一个完整的操作系统，只是提供进程的隔离，具备资源占用少，体积小，启动快的特点。Docker本身是对linux 容器的一种封装。
- Docker的用途或者使用场景:提供一次性的环境，提供弹性的云服务(因为Docker可以随开随关)，组建微服务架构(一台机器可以通过容器跑多个微服务，模拟出微服务架构)。
- Docker的安装：[Get Docker CE for Ubuntu](https://docs.docker.com/install/linupx/docker-ce/ubuntu/)。不过我安装的时候，参照了博客[Ubuntu14.04（32位）下安装使用docker](https://blog.csdn.net/su272009/article/details/46415749).主要操作如下：
```
sudo apt-get install docker.io  # 14.04自带安装包，直接安装
docker version #安装成功 查看版本
#   https://wiki.openvz.org/Download/template/precreated
sudo cat ubuntu-14.04-x86-minimal.tar.gz | docker import - ubuntu:14.04
```

## win10 wsl安装ubuntu
为了学习https://www.datascienceatthecommandline.com/
以及后续继续学些C语言，试着使用下WSL。安装了ubuntu
参照链接https://jingyan.baidu.com/article/49711c61a1a025fa441b7cf2.html
和http://www.manongjc.com/article/36621.html 但是pip还是安装的有问题，后面学习python再注意把

### 遇到的问题
配置了清华的源后直接sudo apt-get install gcc时报错无法找到package。原因时修改完源后没有更新源。这个时候需要执行
`sudo apt-get update`





```
class Solution5 {
public:
	int slution = 0;
	int biggest = 0;


	int divide(int n) {
		if (n <= 0) {
			return 0;
		}
		slution = 0;
		biggest = n;

		DFS(n, 0);
		return slution;
	}
	
	void DFS(int remain, int lastPos) {
		if (!remain) {
			slution++;
			return;
		}
		lastPos++;
		if (lastPos > biggest)
		{
			return;
		}
		DFS(remain - lastPos, lastPos);
		DFS(remain, lastPos);
	}
};
```
