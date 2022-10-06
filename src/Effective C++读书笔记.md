# *Effective* *C++* 读书笔记

## 0 导读 ##

1. 中英文对照（只列举不会的）

    | 中文                      | 英文         |
    | ------------------------- | ------------ |
    | inheritance               | 继承         |
    | composition               | 复合         |
    | assignment                | 赋值         |
    | constructor               | 构造         |
    | destructor                | 析构         |
    | declaration               | 声明式       |
    | signature                 | 签名式       |
    | definition                | 定义式       |
    | initialization            | 初始化       |
    | implicit type conversions | 隐式类型转化 |
    | explicit type conversions | 显式类型转化 |

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

- *copy*构造函数和*copy assignment*函数。*copy*构造函数用来用另一个对象初始化自我对象 ，*copy assignment*函数用来从另一个对象拷贝值到自我对象。一个是构造函数，一个是赋值操作。如果有新对象被创建，用的肯定是构造函数；如果没有新对象被创建，肯定是赋值操作。值传递方式传递参数时肯定会调用*copy*构造函数，往 STL 容器放值也应当是*copy*构造函数被调用。
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

- 对操作符的理解。如果声明了如下`operate*`, 则 `a * b`理解为`operate*(a, b)`:
    ```C++
    const Rational operate*(const Rational& lhs, const Rational& rhs);
    ```





## 1 让自己习惯C++ (Accustoming Yourself to C++) ##


## 条款 01：视 C++ 为一个语言联邦 ##
- 中英文对照

    | 中文                 | 英文     |
    | -------------------- | -------- |
    | muliparadigm         | 多重范型 |
    | procedure            | 过程形式 |
    | object-oriented      | 面向对象 |
    | generic              | 范型     |
    | metaprogramming      | 元编程   |
    | programming paradigm | 编程范型 |
    | encapsulation        | 封装     |
    | polymorphism         | 多态     |

- 条款内容：将 C++ 视为一个语言的联邦，而非单一语言。主要有以下4个次语言：C；Object-Oriented C++; Temeplete C++; STL.

## 条款 02：尽量以`const`, `enum`, `inline` 替换 `#define` (Prefer `consts`, `enums`, and `inline` to `#define`) ##
- 条款理解01：对于单纯常量，最好用`const`对象或者`enum`对象替换`#define`
    - why：对于用`#define` 定义常量的情况：`#define` 的名称未进入符号表，难以跟踪调试；使用常量会减少少量的码。
    - 代码讲解：
    ```C++
    #define ASPEC_RATIO 1.653;           //bad
    const double AspecRatio = 1.653;     //good
    
    const char* const authoName = "Scott Meyers";    //指针和所指之物都是const。
    const std::string authoName("Scott Meyers");  // string对象通常比其前辈char*-based更加合适
    
    // 为了将常量限定在class内，必须将常量定义为成员；但是为了保证常量只有一份，需要让其成为static成员
    class GamePlayer 
    {
    private:
        static const int NumTurns = 5;  //static class常量声明式
        int scores[NumTurns];
        //some code...
    
    };
    const int GamePlayer::NumTurns;     //如果要取常量地址或者编译器要求，你必须提供定义式。放进实现文件
    
    //旧式编译器不支持以上语法，不允许static常量声明时获得初值即in class初值设定。可以将初值放在定义式。
    class CostEstimate
    {
    private:
        static const double FudgeFactor;   //static class常量声明 放在头文件中
        //some code...
    }
    const static double CostEstimate::FudgeFactor = 1.35;   //static class常量定义 放在实现文件中
    
    //如果编译器不允许in class初值设定，但编译器又在编译期间必须知道初值，例如GamePlayer中的数组。此时
    //改用 the enum hack补偿做法，一个属于枚举类型的数值可充当int使用。   
    class GamePlayer 
    {
    private:
        enum { NumTurns = 5};  //enum hack补偿做法
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
        f(a > b ? a : b);   //类型控制const 用引用传参数 是一个真正的函数
    }                       //much better
    ```    

## 条款 03：尽可能使用`const` (Use const whenever possible)
- 条款理解01：对于变量，可以增加一个语义描述，描述该变量不可修改，用以获取编译器的帮助。const多才多艺。主要要注意跟指针使用时，到底是指定指针自身、指针所指之物以及两者都是常量。
    - why：明确告诉编译器你的诉求，用以获取编译器的帮助；另外，在函数参数中使用时，有助于调用者判断函数的行为，例如会不会改变参数等；
    - 代码讲解
    ```C++
    char greeting[] = "Hello"
    char* p = greeting;                 //non-const pointer, non-const data
    const char* p = greeting;           //non-const pointer, const data
    char const * p = greeting;          //non-const pointer, const data
    char* const p = greeting;            //const pointer, non-const data
    // 如果const出现在* 左边意味着所指指物是const。出现在* 右边代表本身是const。这块对于复杂表达式的声明，有通用的方法。
    // 具体可以参照 https://cdecl.org/ 或者  https://c-faq.com/decl/spiral.anderson.html  或者 https://stackoverflow.com/questions/15111526/complex-c-declaration 或者 https://www.codeproject.com/articles/7042/how-to-interpret-complex-c-c-declarations
    
    // 如果被指之物是const，将const置于type之前和之后都是可以的，具体参照习惯,我觉得f1更好
    void f1(const Widget* pw);
    void f2(Widget const * pw);
    
    // 关于迭代器，声明迭代器为常量就跟声明指针为场景是一样，不能指向其他对象；如果希望是不能指向所改之物，应该使用const_iterator
    std::vector<int> vec;
    const std::vect<int>::iterator itr = vec.begin();
    *itr = 42; // ok
    ++itr; // error!
    std::vect<int>:const_iterator citr = vec.begin();
    *citr = 42; // error!
    ++citr;
    
    // 令函数返回一个常量，往往可以降低客户错误导致的意外
    class  Rational {...}
    const Rational operator* (const Ration& lhs, const Ration& rhs){
    ...
    }
    Rationan a, b, c;
    (a * b) = c; // error!
    if (a * b = c){...} // error!
    ```
- 条款理解02：将const实施于成员函数可以：1、可以使得class接口更加容易理解，得知哪个函数会修改对象；2、使得“操作const对象成为可能”，按我的理解，就是可以使用的范围变大了，const对象也可以使用。
    - 两个成员函数如果只是常量性不同，是可以被重载
    ```C++
    class TextBlock {
    public:
        // ... something
        const char& operator[](std::size_t position) const
        { return text[position];}
        char& operator[](std::size_t position)
        { return text[position];}
    private:
        std::string text;
    }

    TextBlock tb("Hello");
    std::cout << tb[0];
    tb[0] = 'x';  // this is why operator[] return reference type

    const TextBlock ctb("Hello");
    std::cout << ctb[0];
    ctb[0] = 'x'; // error! write a conts char&;

    void print(const TextBlock& ctb)
    {
        std::cout << ctb[0];
        // ...
    }
    ```
    - bitwise constness(physical constness)：不更改对象内的任何一个bit，即不更改任何成员变量，这个也正是C++对常量性的定义。一个更改了“指针所指之物”的成员函数能否算作const呢？按照bitwise的定义，确实符合，但是很别扭
    ```C++
    class CTextBlock {
    public:
        // ...
        char& operator[](std::size_t position) const{
            return pText[position];
        }
    private:
        char* pText;
    }
    const CTextBlock cctb("Hello");
    char* pc = &cctb[0];
    *pc = 'J';
    ```
    - logical constness认为，一个const成员函数可以修改对象内的某些bit，但是只得在客户端侦测不出来的情况下如此。但是这种代码不满足C++对const的定义，会编译失败。解决办法很简单，使用*mutable*关键字。**这些成员变量总可能被修改，就算是在const成员函数内。**
    ```c++
    class CTextBlock {
    public:
        // ....
        std::size_t length() const;
    private:
        char* pText;
        std::size_t textLength; // 最近计算过的一次长度
        bool lengthIsValid; // 目前长度是否还有效
    };

    std::size_t length() const {
        if (!lengthIsValid) {
            textLength = std::strlen(pText);
            lengthIsValid = true;
         }
         return textLength;
    }
    ```
    - 在const和non-const成员函数中避免重复。假设TextBlock内的`operator[]`不单单是返回reference指向某个字符，还执行边界检擦、日志记录等，那么目前的两套重载的实现，不可避免的会增加大量的冗余代码。建议通过常量性擦除(casting away constness)的技术，让non-const operate[]调用其const兄弟的方法
    ```c++
    class TextBlock {
    public:
        // ... something
        const char& operator[](std::size_t position) const
        { 
            // ... 
            return text[position];
        }
        char& operator[](std::size_t position)
        { return const_cast<char&>(
            static_cast<const Textblock& >(*this)
            [position]
        );
    }
    ```
## 条款 04：确定对象被使用前已先被初始化 (Make suer that objects are initialized before they're used)

- 条款理解01： 关于“将对象初始化”这事C++反复无常。最佳的处理办法就是在使用对象之前先将它初始化。对于无任何成员函数的内置类型需要手动初始化，对于类对象，初始化的责任在构造函数。但是别混淆了赋值和初始化。C++规定，对象的成员变量的初始化动作发生在进入构造函数本体之前。构造函数最好使用**成员初值列**且总是在初值列列出所有变量。代码讲解如下：
```c++
class PhoneNumber{...}
class ABEntry {
public:
    ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones);
private:
    std::string theName
    std::string theAddress
    std::list<PhoneNumber> thePhones;
    int numTimesConsulted;
}

// bad example : 使用赋值而不是初始化
ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones){
    theName = name;
    theAddress = address;
    thePhones = phones;
    numTimesConsulted = 0;
}

// good example : 在进入本体前完成初始化， 效率更高
ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones)
:theName(name),
theAddress(address),
thePhones(phones),
numTimesConsulted(0){
}

ABEntry::ABEntry()
:theName(),
theAddress(),
thePhones(),
numTimesConsulted(0){
}




```
- 

## 附录 C运算符优先级以及备注点

| 运算符                          | 结合性     |
| ------------------------------- | ---------- |
| () [] -> .                      | 从左至右   |
| ! ~ ++ -- + - * & (type) sizeof | 从右至左   |
| \*  \/  %                       | 从左至右   |
| \+  -                           | 从左至右   |
| <<  >>                          | 从左至右   |
| <  <=  >  >=                    | 从左至右   |
| == !=                           | 从左至右   |
| &                               | 从左至右   |
| ^                               | 从左至右   |
| \|                              | 从左至右   |
| &&                              | 从左至右   |
| \|\|                            | 从左至右   |
| ?:                              | 从左至右   |
| =  +=  -+  \*=  \/= %= &=       | 从左至右   |
| ^=                              | =  <<= >>= | 从左至右 |
| ,                               | 从左至右   |

注意第二行几个一元运算符的结合顺序是从右到左,下面的括号不能去掉
```
// 将ip指向的对象+1
*ip += 1
++*ip
(*ip)++
```
