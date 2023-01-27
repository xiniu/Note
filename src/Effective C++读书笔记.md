# *Effective* *C++* 读书笔记

## 0 导读 (Introduction) ##

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

- *copy*构造函数和*copy assignment*函数。*copy*构造函数用来用另一个对象初始化自我对象 ，*copy assignment*函数用来从另一个对象拷贝值到自我对象。一个是构造函数，一个是赋值操作。**如果有新对象被创建，用的肯定是构造函数；如果没有新对象被创建，肯定是赋值操作**。值传递方式传递参数时肯定会调用*copy*构造函数，往 STL 容器放值也应当是*copy*构造函数被调用。
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


## 条款 01：视 C++ 为一个语言联邦 (Vew C++ as a federation of langueges) ##
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
    - why：对于用`#define` 定义常量的情况：`#define` 的名称未进入符号表，难以跟踪调试(但是使用常量会减少少量的码)。
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
    CALL_WITH_MAX(++a, b+10);   //a++ for 1 times
    
    
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
- 条款理解02：将const实施于成员函数可以：1、可以使得class接口更加容易理解，得知哪个函数会修改对象；2、使得“操作const对象成为可能”，按我的理解，就是对应的方法可以被使用的范围变大了，const对象也可以使用。
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

- 条款理解01： 关于“将对象初始化”这事C++反复无常。最佳的处理办法就是在使用对象之前先将它初始化。对于无任何成员函数的内置类型需要手动初始化，对于类对象，初始化的责任在构造函数。但是别混淆了赋值和初始化。C++规定，对象的成员变量的初始化动作发生在进入构造函数本体之前。构造函数最好使用**成员初值列**且总是在初值列列出所有变量（当然，有时存在多个构造函数、或者成员变量的初始化值需要根据文件读入等场景有例外）。成员变量初始化的顺序是跟声明的顺序一致的，而不管在初始值列中的顺序，为避免迷惑建议初始值中的列顺序跟声明的顺序保持一致。代码讲解如下：
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
numTimesConsulted(0){ }

ABEntry::ABEntry()
:theName(),
theAddress(),
thePhones(),
numTimesConsulted(0){ }
```
- 条款理解02：static对象代表其生命周期从被构造出来，直到程序结束，这种对象包括global对象、定义于namespace作用域中的对象、在class内、在函数内、以及在file作用域内被声明为static的对象。函数内的static对象称为local static对象。为避免“跨编译单元之初始化次序”问题，请以local static对象替换non-local static对象。C++对“定义于不同编译单元内的non-local static对象”的初始化次序无明确定义，因为决定该顺序非常困难，几乎无解。
    -  反面代码案例：如何保证tfs在tempDir之前构造呢？没有办法！
        ```C++
        class FileSystem {
        public:
            //...
            std::size_t numDisks() const;
            //...
        };

        extern FileSystem tfs; // 预留给客户的使用的对象

        class Directory {
        public:
            Directory( param );
            // ...
        };

        Directory::Directory( param ) {
            // ...
            std::size_t disks = tfs.numDisks(); // 使用tfs
            // ...
        }

        Directory tempDir( param );
        ```
    - 正面代码案例：将每个non-local static对象搬运到自己专属的函数内，在函数内声明为static，这个函数返回一个reference指向该对象，这是*Singleton*模式的常见实现方法，C++保证函数内的local static对象会在函数被调用期间首次遇到该对象之定义式时被初始化
        ```C++
        class FileSystem {
        public:
            //...
            std::size_t numDisks() const;
            //...
        };

        FileSystem& tfs() {
            static FileSystem fs;
            return fs;
        }

        class Directory {
        public:
            Directory( param );
            // ...
        };

        Directory::Directory( param ) {
            // ...
            std::size_t disks = tfs().numDisks(); // 使用tfs
            // ...
        }

        Directory tempDir( param );
        ```
        - 正面代码案例：从C++11开始，static变量是线程安全的，无需写锁保护。因此可以使用最简单的方式实现单例即Singleton，不用加锁、不用doublecheck
        ```C++
        class Singleton {
        public:
            Singleton& getInstance() {
                static Singleton inst;
                return inst;
            }
            Singleton(const Singleton&) = delete;
            Singleton& operator=(const Singleton&) = delete;
            // other member...
        private:
            Singleton() {...}
        }
        ```

## 2 构造/析构/赋值运算 (Constructor, Destructor, and Assignment Operators) ##


## 条款 05: 了解C++默默编写并调用哪些函数 (Know what functions C++ silently writes and calls)

- 条款理解01：如果你自己没声明，编译器就会生成一个默认的copy构造、copy assignment操作符和一个析构函数。如果没有声明任何构造函数，编译器会生成一个default构造函数（如果声明了拷贝构造，会不会有default构造呢？ no）。所有这些函数是public且inline。因此如果你写下`class Empty { }`就好比写下如下代码：
    ```c++
    class Empty {
    public:
        Empty() {...}
        Empty(const Empty& rhs) {...}
        ~Empty() {} // 默认式non-virtual的，除非base class自身由声明virtual
        
        Empty& operator=(const Empty& rhs){...}
    };
    ```
- 条款理解02：对于copy构造函数、copy assignment操作符，编译器创建的版本只想单纯的将来源对象的每一个non-satic成员变量拷贝到目标对象。且只有确实用的到才会生成：
    ```c++
    template<typename T>
    class NameObject {
    public:
        NameObject(const std::string& name, const T& value):nameValue(name), objectValue(value){}
        void Print(){cout << "name:" << nameValue << "," << "value:" << objectValue << ",address:" << this << endl;}
    private:
    std::string nameValue;
    T objectValue;
    };

    int main()
    {
        NameObject<int> no1("Smallest Prime Number", 2);
        no1.Print();
        NameObject<int> no2(no1);
        no2.Print();
        no2 = no1;
    }
    ```
- 条款理解03：对于copy构造函数、copy assignment操作符，只有当默认生成的函数代码合法且有意义，编译器才会生成。将上面代码中的成员变量改为`const std::string nameValue;`或者`T &objectValue;`将无法编译通过
    ```txt
    source>:21:11: error: use of deleted function 'NameObject<int>& NameObject<int>::operator=(const NameObject<int>&)'
    21 |     no2 = no1;
        |           ^~~
    <source>:6:7: note: 'NameObject<int>& NameObject<int>::operator=(const NameObject<int>&)' is implicitly deleted because the default definition would be ill-formed:
        6 | class NameObject {
    ```

## 条款 06: 若不想使用编译器自动生成的函数，就该明确拒绝 (Explicitly disallow the use of compile-generated functions you do not want.)

- 条款理解01：为驳回编译器自动产生copy和copy assignment,可以使用将函数声明为private且故意不实现他们。
    ```C++
    class HomeForScale {
    public:
        ...
    private:
        ...
        HomeForScale(const HomeForScale&);
        HomeForScale& operator=(const HomeForScale&); //只有声明
    }
    ```
- 条款理解02：可以使用base class的做法，专门阻止copying动作。
    ```C++
    class Uncopyable {
    protected:
        Uncopyable() {} // 一定要显式声明
        ~Uncopyable() {}
    private:
        Uncopyable(const Uncopyable&);
        Uncopyable& operator=(const Uncopyable&);
    };

    class HomeForScale: private Uncopyable {
    ...
    }
    ```
- 条款理解03：在C++11时代，引入了很多新特性，移动构造、移动赋值等出现使得这块相对更复杂了；也同时增加了default和deleted关键字，便于管理是否自动生成这些特殊的成员函数：
    - 如果有任何构造函数声明，则将不会生成default构造函数
    - 如果有虚析构函数声明，则不会声明默认的析构函数
    - 如果有移动构造或者移动赋值，则不会自动生成copy构造和copy assignment
    - 如果有copy构造或者copy assignment，则不会自动生成移动构造或者移动赋值
    
    在C++11之后，上述的Uncopyable代码可以简化成：
    ```C++
    class Uncopyable {
    public:
        Uncopyable() = default; // 效率更高
        Uncopyable(const Uncopyable&) = delete;
        Uncopyable& operator=(const Uncopyable&) = delete;
    };
    ```

## 条款 07: 为多态基类声明virtual析构函数 (Declare destruction virtual in polymorphic base classes)

- 条款理解01： polymorphic base class应该声明一个virtual析构函数。如果class带有任何一个virtual函数，它就应该拥有一个virtual析构函数。C++明确指出当derived class经由一个base class指针删除，而该base class带着一个non-virtual的析构函数，**其行为式未定义的**
    ```c++
    class TimeKeeper {
    public:
        TimeKeeper();
        virtual ~TimeKeeper();
    };
    class AtomicClock: public TimeKeeper{...};
    class WaterClock: public TimeKeeper{...};
    class WristClock: public TimeKeeper{...};
    
    TimeKeeper* ptk = getTimerKeeper(); // Factory函数
    ...
    delete ptk;
    ```
- 条款理解02： 如果一个类不含其他virtual函数，意味着它并不意图被用作一个base class， 千万不要为其声明virtual析构，带来的影响是：对象会有很客观膨胀，且于其他语言交互会出更多问题。同时，不要试图从不带virtual的类派生，尤其是标准容器等

## 条款 08: 别让异常逃离析构函数 (Prevent exceptions from leaving destructions.)

- 条款理解01： 析构函数绝对不要吐出异常，会导致程序过早退出或出现不明确行为。可以吞下他们或者结束程序
    ```c++
    class DBConnetion {
    public:
        ...
        void close();
    }

    // 使用RAII管理DBConnetion
    class DBConn {
    public:
        ...
        ~DBConn() {
            db.close();
        }
    private:
        DBConnetion db;    
    };

    // 如果有异常，结束异常
    class DBConnV1 {
    public:
        ...
        ~DBConn() {
            try {db.close();}
            catch (...) {
                Errlog(...);
                std::abort();
            }
        }
    private:
        DBConnetion db;    
    };

    // 如果有异常，吞异常
    class DBConnV2 {
    public:
        ...
        ~DBConn() {
            try {db.close();}
            catch (...) {
                Errlog(...);
            }
        }
    private:
        DBConnetion db;    
    };
    ```
- 条款理解02： 如果必须客户向异常做出反应，可以对外提供接口。DBConn对外提供close接口，让客户close。同时自己在析构时检查是否已关闭，没关闭时主动关闭
    ```C++
     // 对外提供接口，让客户关闭
    class DBConnV3 {
    public:
        ...
        void close() {
            if (!closed){
                db.close();
                closed = true;
            }
        }
        ~DBConn() {
            if (!closed){
                try {db.close();}
                catch (...) {
                    Errlog(...);
                    std::abort();
                }
            }
        }
    private:
        DBConnetion db;
        bool closed;  
    };

    ```

## 条款 09: 绝不在构造和析构过程中调用virtual函数 (Never call virtual functions during construction or descontruction.)

- 条款理解01： 简单理解在构造和析构期间，derived class根本不存在，virtual函数不是virtual函数。如果确实需要在构造时获取必要信息，请将derived class在构造时向上传递
  - 错误代码
    ```C++
    class Transaction {
    public:
        Transaction();
        virtual void logTransaction() const = 0;
    }

    Transaction::Transaction() {
        ...
        logTransaction(); // Error!  想在构造时记录日志
    }
    class BuyTransaction: public Transaction {
    public:
        virtual void logTransaction() const {
            ...
        }
        ...
    }
    class BellTransaction: public Transaction {
    public:
        virtual void logTransaction() const {
            ...
        }
        ...
    }
    ```
  - 正确代码
    ```C++
    class Transaction {
    public:
        explicit Transaction(const std::string& logInfo);
        void logTransaction(const std::string& logInfo) const;
    }

    Transaction::Transaction(const std::string& logInfo) {
        ...
        logTransaction(logInfo); 
    }
    class BuyTransaction: public Transaction {
    public:
        BuyTransaction(parameters) : Transaction(createLogString(parameters)){
            // 如果确实需要在构造时获取必要信息，请将derived class在构造时向上传递
            ....
        }
    private:
        static void createLogString() {
            ...
        }
        ...
    }
    ```

## 条款 10: 令operator= 返回一个reference to *this (Hava assignment operators return a reference to *this.)

- 条款理解01：为了“连锁赋值”，所有赋值相关运算符建议返回*this的引用。
    ```c++
    class Widget {
    public:
        ...
        Widget& operator=(const Widget& rhs){
            ...
            return *this;
        }
        Widget& operator+=(const Widget& rhs){
            ...
            return *this;
        }
        Widget& operator=(int rhs){
            ...
            return *this;
        }
        ...
    }
    ```

## 条款 11: 在operator= 中处理“自我赋值” (Handle assignment to self in operator=.)

- 条款理解01：确保对象自我赋值时有良好的行为。
  ```C++
    class BitMap {...};
    class Widget {
        ...
    private:
        BitMap *pb;
    } 
    // 一份不安全的实现，如果出现自我赋值，pb会被销毁
    Widget& Widget::operator=(const Widget& rhs) {
        delete pb;
        pb = new BitMap(*rhs.pb);
        return *this;
    }
  ```
  如下三类技术解决，推荐优先级由低到高：
    - 证同测试（identity test）,具备自我赋值安全性，不具备异常性。如果new失败，Widget持有的指针有害。
        ```c++
        Widget& Widget::operator=(const Widget& rhs) {
            if (this == &rhs) return *this;
            delete pb;
            pb = new BitMap(*rhs.pb);
            return *this;
        }
        ```
    - 精心安排语句：
        ```C++
        Widget& Widget::operator=(const Widget& rhs) {
            BitMap* pOrig = pb;
            pb = new BitMap(*rhs.pb);
            delete pOrig;
            return *this;
        }
        ```
    - copy and swap技术：
        ```C++
        class Widget {
        ...
            void swap(Widget &rhs);
        } 
        
        Widget& Widget::operator=(const Widget& rhs) {
            Widget temp(rhs);
            swap(temp);
            return *this;
        }

        Widget& Widget::operator=(const Widget rhs) { // creat copy temp when pass by value
            swap(rhs);
            return *this;
        }
        ```

## 条款 12: 复制对象时勿忘每一个成分 (Copy all parts of an object.)

- 条款理解01：Copying函数应该确保复制“对象内所有成员变量”以及“所有base class成分”（掉用base的copying函数）
    ```CPP
    class Customer {
    public:
        ...
        Customer(const Customer& rhs);
        Customer& operator=(const Customer&rhs);
        ...
    private:
        std::string name;
    }
    Customer::Customer(const Customer& rhs) : name(rhs.name){ }
    Customer::Customer& operator=(const Customer&rhs){
        name = rhs.name;
        return *this;
    }

    class PriorityCustomer: public Customer {
    public:
        ...
        PriorityCustomer(const PriorityCustomer& rhs);
        PriorityCustomer& operator=(const PriorityCustomer&rhs);
        ...
    private:
        int priority;
    }

    PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs) : Customer(rhs), priority(rhs.priority){ }
    PriorityCustomer::PriorityCustomer& operator=(const PriorityCustomer&rhs){
        Customer::operator=(rhs);
        priority = rhs.priority;
        return *this;
    }
    ```
- 条款理解02：不要尝试使用一个copying函数调用另一个copying函数。如果确实由很多重复代码，可以把重复代码封装到第三个函数，然后两个copying函数分别调用。

## 3 资源管理

## 条款 13: 以对象管理资源(Use objetcs to manage resource)

- 条款理解01：主要就是使用RAII惯用法来管理资源，所谓RAII（Resource Acquisition Is Initialization）：
  - 获得资源后立刻放到资源管理对象中
  - 管理对象运用析构函数确保资源释放
- 条款理解02：由于智能指针等在C++11之后由新的变化，所以不再赘述。RAII也是比较熟悉的特性，不再赘述。
  
## 条款 14: 在资源管理类中小心copying行为（Think carefully about copying bahavior in resource-namaging classes）

- 条款理解01： 如何处理RAII对象的copying行为，基本的处理思路：禁止拷贝（想想都有哪些手段）、所有权转移、使用引用计数。这实际就是C++11之后的unique_ptr和shared_ptr。 本文不再赘述


## 条款 15: 在资源管理类中提供对原始资源的访问（Provide access to raw resource in resource-managing classes）

- 条款理解01： 现实的代码世界存在很多APIs指涉资源，例如裸指针。资源管理类需要提供接口，直接返回资源，一般建议： 使用`get()`方法返回裸指针、重载指针取值操作符(`operator->`和`operator*`)直接转换至底部资源.C++11中的智能指针就是如此实现的。
  ```C++
  class NameObject {
    public:
        NameObject(const std::string& name, const int& value):nameValue(name), objectValue(value){}
    
    public:
    std::string nameValue;
    int objectValue;
    };

    void oldAPI(NameObject* ptr) {
        ptr->nameValue = ptr->nameValue + "+1";
        ptr->objectValue++;

    }
    int main()
    {
        auto up = make_unique<NameObject>("ONE",1);
        cout << up->nameValue << ":" << up->objectValue << endl;
        oldAPI(up.get());
        cout << (*up).nameValue << ":" << (*up).objectValue<< endl;
    }
  ```

- 条款理解02：有时候方便起见，可以提供隐式转换的能力，从资源管理隐式转化为原始资源
    ```CPP
    FontHandle getFont(); // 原生的获取handle的方法，C API
    void releaseFont(FontHandle fh); // 原生获取handle必须经由此释放，C API
    void changeFontSize(FontHanle f, int newSize); // C API

    // example 1: explicit转化
    class FontV1 {
    public:
        explicit Font(FontHandle fh) : f(fh) {}
        ~Font() { releaseFont(f);} // RAII
        FontHandle get() const {return f;}
    private:
        FontHandle f;
    }

    FontV1 f(getFont());
    changeFontSize(f.get(), someSize); // 每次调用都要调用get

    // example 1: implicit转化
    class FontV2 {
    public:
        explicit Font(FontHandle fh) : f(fh) {}
        ~Font() { releaseFont(f);} // RAII
        operator FontHandle() const { return f;} // 隐式转化函数
    private:
        FontHandle f;
    }

    FontV2 f(getFont());
    changeFontSize(f, someSize); // Font隐式转化为FontHandle
    ```

## 条款 16: 成对使用new和delete时要采取相同形式（Use the same form in corresponding use of new and delete）

- 条款理解01：`new`和`delete`， `new []`和`delete []`必须要配套使用，否则会导致未定义行为
    ```C++
    std::string* strPtr1 = new  std::string;
    std::string* strPtr2 = new  std::string[100];
    ...
    delete strPtr1;
    delete []strPtr2;
    ```
- 条款理解02：永远不要对数组形式`typedef`，尽量使用`string` 和 `vector`等stl容器
    ```C++
    typedef std::string AddressLine[4];

    AddressLine* pal = new AddressLine;
    ...
    delete pal;  // Dangerous
    ```
## 条款 17: 以独立语句将newed对象置入智能指针（Use newed objeces in smart pointers in standalone statement)

- 条款理解01：以独立语句将newed语句置入智能指针，如果不这样做，一旦异常被抛出，可能导致资源泄露。主要的原因时，我们希望newed的指针和智能指针的构造函数中间不能被打断，但是C++语言并不是以特定顺序计算参数。
    ```C++
    int priority();
    void processWidget(std::shared_ptr<Widget> pw, int priority);   
    ...
    // bad example. what if: 
    // step1 new Widget; 
    // step2 call priority();-----> may throw exception ? 
    // step 3 construct of shared_ptr
    processWidget(std::shared_ptr<Widget>(new Widget), priority());

    // good example. : set newed objeces in smart pointers in standalone statement
    std::shared_ptr<Widget> pt(new Widget);
    processWidget(pt, priority());
    ```
- 条款理解02：在C++11中，尽可能使用`make_unique`、`make_shared`也可以解决该类困境。
    ```C++
    // good example . : after c++11, prefer make_unique、 make_shared
    std::shared_ptr<Widget> pt(new Widget);
    processWidget(std::make_shared<Widget>(), priority());
    ```

## 设计与声明

## 条款 18：让接口容易被使用，不易被误用（Make interfaces easy to use correctly and hard to use incorrectly）

- 条款理解01： 可以通过引入新类型的方式提升接口的易用性。
    ```C++
    // Bad example:
    class Date {
    public:
        Date(int month, int day, int year);
        ...
    };
    // 如上的接口实现会导致各种各样的问题
    // 1 以错误次序传参; 2 传递无效的月份；
    Data d(30, 3, 1995);
    Date d(2, 30, 1995);

    // Good example: 引入新的类型、限制其值
    struct Day {
        explicit Day(int d) : val(d){}
        int val;
    };

    struct Month {
        explicit Month(int m) : val(m){}
        int val;
    };

    struct Year {
        explicit Year(int y) : val(y){}
        int val;
    };

    class Date {
    public:
        Date(const Month& m, const Day& d, const Year& y);
        ...
    };
    // 可以解决以错误次序传参的问题
    Date d(Month(3), Day(30), Year(1995));

    // 可以更进一步的，限定Month类型的所有合理的值
    class Month {
    public:
        static Month Jan() {return Month(1);}
        static Month Feb() {return Month(2);}
        ...
        static Month Dec() {return Month(12);}
        ...
    private:
        explicit Month(int m);
        ...
    }
  ```
- 条款理解02： 限制类型内什么操作可以做。参照条款3，让```const```修饰```operator*```的返回值，可以避免客户犯如下错误```if (a = b * c)```
    ```C++
    const Rational operator* (const Rational &lhs, const Rational &rhs)
    ```
- 条款理解03： 尽量让你的types的行为和内置类型一致。好的例子是C++ STL每个容器都有一个size成员函数。反例是java语言，针对数组，有一个length的成员变量；针对String， 有一个lengh()方法； 针对list， 有一个size()方法。

- 条款理解04：任何接口如果要求客户必须做某些事，就有着“不正确使用”的倾向。
    ```C++
    // Bad example: 如下的factory函数，开启了两个客户错误的机会：
    // 1 没有删除指针； 2 删除指针两次
    Invesment* createInvestMent(...);

    // Good Example:
    std::shared_ptr<Invesment> createInvestment(...);

    // 甚至可以定制删除器，并解决cross-DLL problem
    std::shared_ptr<Invesment> createInvestment(...){
        std::shared_ptr<Invesment> retVal(static_cast<Investment*>(0), getRidOfInvestment);
        retVal = ...; // 令retVal指向正确的对象
        return retVal; 
    }
    ```

## 条款 19：设计class犹如设计type（Treat class design as type design）

- 条款理解01： 你应该带着和“语言设计者当初设计语言内置类型时”一样谨慎的研讨class的设计：
    - 新type的对象应该如何被创建和销毁？ 这会影响你的class的构造函数、析构函数、内存分配函数、内存释放函数等
    - 对象的初始化和赋值该有什么样的行为
    - 新type的对象如果被passed by value，意味着什么？ 即copy构造如何实现
    - 有哪些合法值
    - 继承图系(inheritance graph)
    - 新的type需要什么样的转化？ 如果你希望类型T1之物被隐式转化为T2之物，那么必须在```class T1```中写一个类型转化函数```operator T2()```或在```class T2```中写一个non-explicit-one-arguement
    - 什么样的操作符和函数对此新type而言是合理的
    - 该如何取用成员
    - 什么是“未声明接口(undeclared interface)”
    - 新的type有多一般化？ 是否考虑应该定义一个class template
    - 你真的需要一新的type吗？如果只是为一个```derived class```增加技能，单纯定义non-member函数或者template，更能达到目标

## 条款 20：宁以pass-by-reference-to-const替换pass-by-value(Prefer pass-by-reference-to-const to pass-by-value)

- 条款理解01： 尽量以pass-by-reference-to-const替换pass-by-value。前者通常比较高效，并可以避免slicing problem
    ```C++
    class Person {
    public:
        Person();
        virtual ~Person();
    private:
        std::string name;
        std::string address;
    };

    class Student : public Person  {
    public:
        Student();
        ~Student();
    private:
        std::string schoolName;
        std::string schoolAddress;
    };

    // by value传参数会拷贝对象（包括内部的其他类型对象成员），造成严重的开销
    bool validateStudent(Student s);

    // pass-by-reference-to-const效率高的多，没有多余的构造或者析构函数调用；不必考虑会修改入参
    bool validateStudent(const Student& s);


    class Window {
    public:
        ...
        string name() const;
        virtual void desplay() const;
    };

    class WindowWithScrollBars: public Window {
    public:
        ...
        virtual void desplay() const;
    };

    void prinNameAndDisplay(Window w) {
        cout << w.name() << endl;
        w.display();
    }
    WindowWithScrollBars wwsb;
    prinNameAndDisplay(wwsb); // 会发生对象切割，w.display();调用的总是Window::desplay()


    void prinNameAndDisplayV2(const Window& w) {
        //  传进来是什么类型，实际表现就是什么类型
            cout << w.name() << endl;
            w.display();
        }
    WindowWithScrollBars wwsb;
    prinNameAndDisplay(wwsb); 
    ```

- 条款理解02： 内置类型、STL迭代器、函数对象，pass-by-value更合适。当然并不是绝对的，所有小型types都应该设计成pass-by-value

## 条款 21：必须返回对象时，别妄想返回其reference(Don't try to return a reference when you must return an object)

- 条款理解01：一个“必须返回新对象”的函数的正确写法是：就让那个函数返回一个新对象。一切通过reference、pointer返回栈对象、堆对象都是徒劳的
    ```C++
    const Rational operator* (const Rational &lhs, const Rational &rhs){
        return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
    }
    ```

## 条款 22：将成员变量声明为private(Declare data members private.)

- 条款理解01：将成员变量声明为private，可以赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供充分的弹性（例如，修改优化实现逻辑）
- protected并没有比public提供更多的封装性。 针对封装性的理解，**“当其内容改变时可能造成的代码破坏量”**，“愈多的函数可以访问它，数据的封装性越低”。


## 条款 23：宁以non-member、non-friend替换member函数(Prefer non-member non-friend functions to member function)

- 条款理解01：宁以non-member、non-friend替换member函数。这样做可以增加封装性、包裹弹性(packaging flexibility)和技能扩充性
    ```C++
    class WebBrowser {
    public:
        ...
        void clearCache();
        void clearHistory();
        void removeCookies();
        ...
    };
    
    // 如果要增加一个清理所有的操作，可以通过member函数方式
    class WebBrowser {
    public:
        ...
        void clearEverything(); // 分别调用clearCache、clearHistory、removeCookies
        ...
    };

    // 也可以通过non-member的方式
    void clearBrowser(WebBrowser &wb) {
        wb.clearCache();
        wb.clearHistory();
        wb.removeCookies();
    }

    // non-member更优的原因时：
    //  1 没有增加“能够访问class内部private成员成分的”数量
    //  2 更低编译相依度
    ```
- 条款理解02：在C++中，常见的一种处理方式是将便利函数放置在多个头文件中、但是隶属于同一个命名空间，客户可以更轻松的做扩充
    ```C++
    // 头文件webbrowser.h 这个头文件针对class WebBrowser自身以及核心功能
    namespace WebBrowserStuff {
    class WebBrowser{...}
    ... // 核心机能，所有客户都会用到的non-member函数
    }

    // 头文件webbrowserbookmarks.h 
    namespace WebBrowserStuff {
    ... // 与书签相关的便利函数
    }

    // 头文件webbrowsercookies.h 
    namespace WebBrowserStuff {
    ... // 与cookie有关的便利函数
    }
    ```

## 条款 24：若所有参数皆需要类型转化，请为此采用non-member函数(Declare non-member function when type conversions should apply to all parameters)

- 条款理解01：虽然令class支持隐式转化往往是个糟糕的主意，但是凡事均有例外。例如，当前项建立一个数值类型时，允许整数转换为有理数的要求颇为合理。当你想某个函数的所有类型都支持隐式转化，那么这个函数必须是个non-member。
    ```C++
    class Rational {
    public:
        Rational(int numerator = 0, int denominator = 1); // 无explicit，允许转化
        int numerator() const;
        int denominator() const;
    private:
        ...
    }

    // 当你想支持乘法时， 如果把operator* 实现为成员函数：
    class Rational {
    public:
        ...
        const Rational operator*(const Ration& rhs) const;
        ...
    };

    Rational oneHalf(1,2);
    Rational result = oneHalf * 2; // can work
    result = 2 * oneHalf ; // can not work

    // 如果把operator* 实现为non-member函数，将解决所有问题
    class Rational {
    public:
        ...
    };
    const Rational operator*(const Ration& lhs, const Ration& rhs) {
        return Rational(lhs.numerator() * rhs.numerator(),
        lhs.denominator() * rhs.denominator());
    }
    Rational oneHalf(1,2);
    Rational result = oneHalf * 2; // can work
    result = 2 * oneHalf ; // not work
    ```


## 条款 25：考虑写出一个不抛出异常的swap函数(Consider support for a non-throwing swap)

- 条款理解01：swap原本只是STL的一部分，但是后来成为异常安全性编程的脊柱，以及用来处理自我赋值问题。缺省的swap版本对某些类型的对象十分低效，典型的pimpl用法中使用默认的swap实现明显不合理。
    ```C++
    namespace std {
        template<typename T>
        swap(T& a, T& b) {
            T temp(a); 
            a = b;
            b = temp;
        }
    }

    // pimp手法
    class WidgetImpl {
    public:
    ...
    private:
        int a, b, c;
        std::vector<double> v; // so many data;
    }

    class Widget {
    public:
        Widget(const Widget& rhs);
        Widget& operator=(const Widget& rhs) {
            ...
            *pImpl = *(rhs.pImpl);
            ...
        }
    private:
        WidgetImpl* pImpl;
        }
    }
    ```
- 条款理解02：针对class类型，可以增加一个swap的成员函数，然后将std::swap特化，令他调用该成员函数。
    ```C++
    class Widget {
    public:
        ...
        void swap(Widget &other) {
            using std::swap;
            swap(pImpl, other.pImpl);
        }
    ...
    };

    namespace std {
        template<>
        void swap<Widget>(Widget& a, Widget& b) {
            a.swap(b);
        }
    }
    ```
- 条款理解03：如果考虑到模板类，情况比较复杂：在你的class或者template所在的命名空间提供一个non-member swap， 并令其调用swap成员函数，并且建议std::swap版本也要提供(为那些使用std::swap的迷途程序员提供帮助, 所谓迷途程序员指的是非要写std::swap(obj1, obj2)的人)。
    ```C++
    namespace WidgetStuff {
        ...
        template<typename T>
        class Widget {...};

        template<typename T> // non-member swap函数，但是这不属于std命名空间
        void swap(Widget<T>& a, Widget<T>& b) {
            a.swap(b);
        }

        namespace std {
            template<>
            void swap<Widget>(Widget& a, Widget& b) {
                a.swap(b);
            }
        }
    }

    template<typename T>
    void doSomething(T &obj1, T &obj2){
        using std::swap; // 令std::swap可见
        swap(obj1, obj2); // 为T型别找最佳的swap（global或者T所属命名空间中的--> std命名空间中的）
    }
    ```

## 5 实现 Implementations

## 条款26： 尽可能延后变量定义式的出现时间 (Postpone variable definations as long as possible)

- 条款理解01：尽可能延后变量定义式的出现。这样做可以增加程序的清晰度并改善程序效率。“尽可能延后的真正意义，不只应该延后变量的定义直到非得使用该变量的前一刻，甚至应该尝试延后这份定义直到能够给它处置实参为止，不仅能够避免构造和析构非必要对象，还可以避免无意义的default构造行为”。
    ```c++
    // bad example:
    std::string encryptPassword(const std::string& password) {
        using namespace std;
        string encrypted;
        // encrypted定义过早，如果抛出异常，它就没有真正被使用
        if (password.length() < MimimumPasswordLength) {
            throw logic_error("Password is too short");
        }
        ...
        return encrypted;
    }

    // good example:
        std::string encryptPassword(const std::string& password) {
        using namespace std;
        // encrypted定义过早，如果抛出异常，它就没有真正被使用
        if (password.length() < MimimumPasswordLength) {
            throw logic_error("Password is too short");
        }
        string encrypted;
        ...
        return encrypted;
    }
    ```
- 条款理解02：关于循环的处理：如果变量只在循环体内使用，把它定义在循环外并在每次循环时赋值比较好还是直接定义在循环提内比较好。A：定义在循环体外，需要 1个构造+1个析构+N个赋值； B：定义在循坏体内，需要N个构造和N个析构。除非明确知道赋值比构造和析构成本低、正在处理代码中对效率高度敏感的部分，否则建议使用B方法，这样对程序的可理解性和维护性更优。

## 条款27： 尽量少做转型操作 (Minimize casting)

### 几种转型类型

- C风格转型
  
  | 类型            | 描述                                      |
  | --------------- | ----------------------------------------- |
  | `(T)expression` | C风格旧式转型                             |
  | `T(expression)` | C风格旧式转型（函数风格），跟上面并无差别 |

- C++style转型  

  | 类型               | 描述                                                                                                                                                                           |
  | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | `const_cast`       | 常量性擦除(cast away the constness)，唯一有此功能的C++style转型                                                                                                                |
  | `dynamic_cast`     | 安全向下转型(safe downcasting),它是唯一无法由旧式转型完成的动作，它式唯一可能耗费重大运行成本的转型操作， 安全的意思是会判断某对象是否是归属于继承体现中的某个类，而不是直接转 |
  | `reinterpret_cast` | 执行低级转化，例如将point to int转化为int，除了在低级代码外很少见                                                                                                              |
  | `static_cast`      | 强迫隐式转化，功能全面：将non-const转为const， 将int转为double，将void*转为typed指针，将pointer-to-base转为pointer-to-derived， 但无法将consy转为non-const                     |

### 条款理解01
- 条款理解01：宁可使用C++style类型转化，不要使用旧式转型。前者很容易辨识出来，而且由作用越明确、编译器等越可以提供诊断信息。
   
- 条款理解02：尽量避免转型操作，有一种误解式认为转型其实什么都没做。但是其实任何一个类型的转化往往真的令编译器编译出在运行期间执行的码
    ```C++
    int x, yl
    ...
    double d = static_cast<double>(x)/y; //肯定会生成一些代码，因为在计算机体系结构中，int和double有完全不同的底层表达

    class Base {...};
    class Derived: public Base {};
    Derived d;
    Base* p = &d; // 有时候p和d正真的地址并不一样，会有个偏移 
    ```
- 条款理解03：如果要在derived class中virtual函数中调用父类的对应函数，不应该使用转型,而应该使用域操作符。因为这样，其实是在当前对象的base class成分的副本上执行对应的方法!
    ```C++
    // bad example
    class Window {
    public:
        virtual void onResize(){...}
        ...
    };
    class SpecialWindow : public Window {
    public:
        virtual void onResize(){
            static_cast<Window>(*this).onResize();
            ...
        }
    }

    // good example
    class SpecialWindow : public Window {
    public:
        virtual void onResize(){
            Window::onResize();
            ...
        }
    }
    ```
- 条款理解04：应当尽可能避免dynamic_cast，因为dynamic_cast的需要实现版本执行速度都相当慢。之所需需要dynamic_cast，是因为想在你认定是derived class对象身上执行derived class操作函数。 使用类型安全容器和将virtual函数往继承上方移动即让基类也增加相关的操作函数（该函数可以什么人都不做）可以避免这种场景。
- 条款05：绝对避免一连串的dynamic_cast!!!
    ```C++
        class Window { ... };
        class SpecialWindow1 : public Window{ ... };
        class SpecialWindow2 : public Window{ ... };
        ...
        typedef vector<shared_ptr<Window> > VPN;
        VPN winPtrs;

        for (VPN::iterator itr = winPtrs.begin(); itr != winPtrs.end(); ++itr) {
            if (SpecialWindow1* psw1 = dynamic_cast<SpecialWindow1*>(itr->get())) {
                ...
            } else if (SpecialWindow1* psw2 = dynamic_cast<SpecialWindow2*>(itr->get())) {
                ...
            }
            ...
        }
    ```

- 条款06：优秀的C++代码很少使用转型，但是如果说要完全拜托不切实际，但是如果必须要做，尽可能的隔离起来，隐藏在某个函数内。
    
## 条款28： 避免返回handles指向对象内部成分 (Avoid returning "handles" to object internals)

- 条款理解01：Reference、指针、迭代器等都是所谓的handles，返回一个代表内部数据的handle会导致对象封装性的降低，导致const成员函数不像成员函数。
    ```C++
    class Point {
    public:
        Point(int x, int y);
        ...
        void SetX(int x);
        void SetY(int y);
        ...
    };

    class RectData {
        Point ulhc;
        Point lrhc;
    }

    class Rectangle {
        Point& upperLeft() const {return pData->ulhc;} // error!
        Point& lowerRight() const {return pData->lrhc;}
    private:
        shard_ptr<RectData> pData; 
    }
    ...

    const Rectangle rec(xx, yy);
    rec.upperLeft.setY(50); 
    ```

- 条款理解02：即使对这些返回值加了const也不行，也会导致dangling handles。
    ```C++
    class GUIObject{...};
    const Rectangle boundingBox(const GUIObject& obj);

    GUIObject obo(...);
    const Point* pUpperLeft = &(boundingBox(obo).upperLeft()); // boundingBox(obo) creat a temp Rectangle and will destroy later.
    ```
- 条款理解03：避免返回handles指向对象内部。这样可以增加封装性，使得const成员函数的行为像个const，并将dangling handles的可能性降到最低。这并不意味这绝对不能这么做，有时候必须那么做，但这毕竟是例外，不是常态。
 
## 条款29： 为“异常安全”而努力是值得的(Strive for exception-safe code)

### 基本概念
- “异常安全性的两个条件”：
  - 不泄露任何资源，包括内存。mutex等.可以使用RAII解决资源泄露
  - 不允许数据被破坏。考虑如果new抛出异常，原始数据是否被破坏
    ```c++
    // bad example
    class PrettyMenu {
    public:
        ...
        void changeBackGroud(std::istream& imgSrc);
        ...
    private:
        Mutex mutex;
        Image* bgImage;
        int imageChanges;
    }

    void PrettyMenu::changeBackGroud(std::istream& imgSrc) {
        lock(&mutex); // This will lead resource leak!
        delete bgImage;
        ++imageChanges;
        bgImage = new Image(imgSrc); // What if new throw exception?
        unlock(&mutex);

    }
    ```
- “异常安全函数的三类保证”，一般而言我们想提供“强烈保证”：
  - 基本承诺：如果异常被抛出，程序内的任何事物都在有效状态下，没有任何对象或数据结构因此破坏。
  - 强烈保证：如果异常被抛出，程序状态不改变。如果函数成功，就是完全成功；如果函数失败，程序就会回到调用之前的状态
  - 不抛掷承诺（no throw），承诺绝不抛出异常，总是能够完成原先承诺的功能，作用于内置类型身上的所有操作都提供nothrow保证。注意，带着空白的异常明细的函数`int doSomething() throw()`并不是说绝不抛异常，它没有提供任何异常保证

### 条款理解

- 条款理解01：利用智能指针、函数语句重排等手段提供异常安全性。但是这也仅仅是提供了基本承诺，因为可能输入流的读取记号已被移走
    ```c++
    class PrettyMenu {
    public:
        ...
        void changeBackGroud(std::istream& imgSrc);
        ...
    private:
        Mutex mutex;
        shared_ptr<Image> bgImage;
        int imageChanges;
    };

    void PrettyMenu::changeBackGroud(std::istream& imgSrc) {
        Lock m1(&mutex);
        bgImage.reset(new Image(imgSrc));
        ++imageChanges;
    }
    ```
- 条款理解02：copy and swap这个一般化的策略会提供强烈保证：为你打算修改的对象做出一份副本，然后再那副本上做一切必要的修改。若任何修改动作抛出异常，原对象仍然保持未改变状态。待所有改变都成功后，再将修改过的副本和原对象在一个不抛出异常的swap函数中置换。实现上通常使用pimp惯用法
    ```C++
    struct PMImpl {
        shared_ptr<Image> bgImage;
        int imageChanges;
    };
    class PrettyMenu {
        ...
    private:
        Mutex mutex;
        shared_ptr<PMImpl> pImpl;
    };

    void PrettyMenu::changeBackGroud(std::istream& imgSrc) {
        using std::swap;
        Lock m1(&mutex);
        shared_ptr<PMImpl> pNew(new PMImpl(*pImpl));
        pNew->bgImage.reset(new Image(imgSrc));
        ++pNew->imageChanges;
        swap(pImpl, pNew);
    }
    ```
- 条款理解03：函数提供的“异常安全性保证”通常只等于其所调用之各个函数的“异常安全保证”中的最弱者

## 条款30： 透彻了解*inlining*的里里外外(Understand the ins and outs of inlining)

### 基本概念

- inline函数的好处：不蒙受函数调用所招致的额外开销，并且便于编译器优化
- 缺点：可能会增加object code的大小，进一步降低高速缓冲装置的击中率，进而影响效率
- inline只是对编译器的申请，不是强制命令。这项申请可以隐式提出（将函数定义在class定义式内，包括成员函数和friend），也可以显式提出（使用`inline`关键字）
    ```c++
    template<typename T>
    inline const T& std::max(const T& a, const T& b){
        return a < b ? b : a;
    }
    ```
- inline通常一定被置于头文件中，因为在编译过程中需要将函数调用替换为函数本体，编译器必须知道函数本体，大部分环境中inline式编译器行为
- template的具现化予inline无关。如果你认为所有根据此template具现出来的函数都应该式inline，请将此template声明为inline。二者没有什么必然的连接
- inline只是对编译器的申请，但是编译器可以忽略，例如对virtual的调用会使inlining落空。目前大部分编辑器如果无法将函数inline化，会给警告信息。同时，编译器通常不对通过函数指针而进行的调用实时inlining
- 构造函数和洗头函数往往不适合作为inline，会导致代码膨胀
- 必须评估将函数声明为inline的冲击，inline函数如果要变化，需要将所有客户端代码重编，而不只是重新连接

### 条款理解

- 将inlining限制到小型、且被频繁调用的函数身上，这使得日后的调试和二进制升级更容易，也使得潜在的代码膨胀问题最小化，使程序的速度提升机会最大化
- 不要只因为funtion template出现在头文件中，就将它声明为inline

## 条款31： 将文件间的编译依存关系降至最低

### 3个简单的策略：
- 如果使用object references或者object pointers就可以完成任务，就不要使用objects。
- 如果能够，尽量以class声明式替换class定义式：当你声明一个函数而它用到某个class时，并不需要该class的定义：纵使函数以by value方式传递参数
- 为声明式和定义式提供不同的头文件
    ```c++
    #include "datefwd.h" // 这个头文件中前置声明class Date，但不定义
    Data today();
    void clearAppointments(Date d);
    ```

### 条款理解

- 条款理解01：使用pimp或者叫Handle classes降低编译依存关系
    ```C++
    // Person的定义式，即Person.h
    #include <sring>
    #include <memory>

    class PersonImpl;
    class Date;
    class Address;
    class Person {
    public:
        Person(const string& name, const Date& birthday, const Address& addr);
        string name() const;
        string birthDate() const;
        string address() const;
        ...
    private:
        shared_ptr<PersonImpl> pImpl;
    }

    ....
    // 函数的实现
    #include "Person.h"
    #include "PersonImpl.h"

    Person::Person(const string& name, const Date& birthday, const Address& addr) : pImple(new PersonImpl(name, birthday, addr)) {}

    string Person::name() const {
        return pImple->name();
    }
    ```
- 条款理解02：另一种实现Handle classes的方式是Interfaces class。利用抽象基类和工厂factory函数（virtual构造函数），解除接口和实现之间的耦合关系。
    ```c++
    // Person.h
    #pragma once
    #include<string>
    #include<memory>
    using namespace std;
    class Person {
    public:
        Person() = default;
        virtual ~Person() {};
        virtual string name() const = 0;
        virtual string birthDate() const = 0;
        virtual string address() const = 0;
        static shared_ptr<Person> create(const string& name, const string& birthDay, const string& addr);
    };

    // RealPerson.h
    #pragma once
    #include"Person.h"
    class RealPerson : public Person {
    public:
        RealPerson(const string& name, const string& birthDay, const string& addr)
        : theName(name), theBirthDate(birthDay), theAddress(addr){}
        virtual ~RealPerson() {}
        string name() const { return theName; }
        string birthDate() const { return theBirthDate; }
        string address() const { return theAddress; }
    private:
        string theName, theBirthDate, theAddress;
    };

    // Person.cpp
    #include "Person.h"
    #include "RealPerson.h"
    shared_ptr<Person> Person::create(const string& name, const string& birthDay, const string& addr) {
        bool someCondition = true;
        //...
        if (someCondition) {
            return shared_ptr<Person>(new RealPerson(name, birthDay, addr));
        }
        return nullptr;
    }

    //业务代码
    #include "Person.h"
    #include <iostream>

    int main()
    {
        std::cout << "Hello World!\n";
        shared_ptr<Person> p = Person::create("Wang", "2022.1.1", "Hubei");
        cout << p->name() << endl;
    }
    ```

## 6 继承与面向对象设计 Inheritance and Object-Oriented Design

## 条款32： 确定你的public继承是is-a关系(Make sure public inheritance model "is a")

- 条款理解01：“public继承”意味这is-a。 适用于base classes身上的每件事也一定适用于derived classes（里氏替换原则），每一个derived class对象也是一个base class。反之并不成立。于是，在C++中，任何函数如果期望获得一个类型为Person的实参（或者pointer-to-Person或者reference-to-Person），都愿意接受一个Student对象（或pointer-to-Student或reference-to-Student）
    ```c++
    class Person {...};
    class Student : public Person {...};
    void eat(const Person& p);
    void study(const Student& s);
    Person P;
    Student s;
    eat(p);
    eat(s);
    study(s);
    study(p); // ERROR!
    ```
- 条款理解02：有时候你的生活直觉会误导你：
  - 典型的误导场景1：企鹅penguin是一种鸟，鸟都可以飞，但是鸟不会飞。解决思路是：1.承认有些鸟不会飞，有些鸟会飞，进行双classes设计； 或者2、通过抛出异常或者不实现成员函数的方法，在运行期或者编译器侦测到错误
  - 典型的误导场景2：正方形是矩形，那么`class Square`是否应该以public继承`class Rectangle` ?答案是否定的，可以独立的施加于矩形的操作（单独地调整长和宽）无法直接施加于正方形
- 条款理解03：世界上不存在“完美设计”，只存在“最佳设计”：取决于系统希望做什么事，包括现在和未来。如果你不打算关注鸟的飞行行为，那么不用区分会飞的鸟和不会飞的鸟。

## 条款33： 避免遮掩而来的名称(Avoid hiding inherited names)

### 基本概念

- C++名称遮掩：C++名称遮掩的唯一规则是：名称遮掩，不会关注名称是否是相同或者不同类型。如下例子中x是`SomeFunc`作用域内的`double x`。
    ```C++
    int x;
    void SomeFunc() {
        double x;
        std::cin >> x;
    }
    ```
    ```txt
    +-------------------------------------+
    |Global scope                         |
    |x          +-----------------------+ |
    |           |  SomeFunc's scope     | |
    |           |  x                    | |
    |           |                       | |
    |           |                       | |
    |           +-----------------------+ |
    +-------------------------------------+
    ```
- 在涉及到继承时，derived class的作用域嵌套在base class内，当寻找某个名称合适的类型，总是"从内到外"依次寻找
    ```C++
    class Base {
    private:
        int x;
    public:
        virtual void mf1() = 0;
        virtual void mf2();
        void mf3();
        ...
    };

    class Derived: public Base {
    public:
        virtual void mf1();
        void mf4();
        ...
    };
    ```
    ```txt
    +--------------------------------------------------------+
    |Base's scope:                                           |
    |x                                                       |
    |mf1                                                     |
    |mf2                                                     |
    |mf3                     +-------------------------------+
    |                        |Derived’s scope:              ||
    |                        |mf1                           ||
    |                        |mf2                           ||
    +--------------------------------------------------------+
    ```
- 在涉及到继承时并在存在函数重载并且又希望覆写其中一部分，会出现Base中的名称被遮掩的情况。这是C++对继承而来的名称的缺省遮掩行为
    ```c++
    class Base {
        public:
        virtual void mf1() = 0;
        virtual void mf1(int){}
        virtual void mf2(){}
        void mf3(){}
        void mf3(double){}
    };

    class Derived : public Base {
        public:
        virtual void mf1(){}
        void mf3(){}
        void mf4(){}
    };

    int main() {
        Derived d;
        d.mf1();
        int x = 5;
        d.mf1(5); // Compile Error
        d.mf3(5); // Compile Error
    }
    ```
### 条款理解

- 条款理解01：derived classes中的名称会遮掩base class中的名称。尤其在public继承下没人希望如此，可以通过`using`声明导入Base作用域内所有的名称来解决该问题
    ```c++
    class Base {
        public:
        virtual void mf1() = 0;
        virtual void mf1(int){}
        virtual void mf2(){}
        void mf3(){}
        void mf3(double){}
    };

    class Derived : public Base {
        public:
        using Base::mf1;
        using Base::mf3;
        virtual void mf1(){}
        void mf3(){}
        void mf4(){}
    };

    int main() {
        Derived d;
        d.mf1();
        int x = 5;
        d.mf1(5); // It's OK  now.
        d.mf3(5); // It's OK  now.
    }
    ```
- 条款理解02：在private继承等场景下，如果不想使得继承Base作用域内的所有名称都可见，可以适用转发函数（forwarding function）
    ```C++
    class Base {
        public:
        void mf3(){}
        void mf3(double){}
    };

    class Derived : private Base {
        public:
        void mf3(){
            Base::mf3();
        }

    };

    int main() {
        Derived d;
        d.mf3();
        d.mf3(5); // Compile Error!
    }
    ```

## 条款34： 区分接口继承和实现继承(Differentiate between inheritance of interface and inheritance of implementation.)

- 条款理解01：成员函数的接口总会被继承
    ```c++
    class Shape {
    public:
        virtual void draw() const = 0;
        virtual void error(const std::string &msg);
        int object( ) const;
        ...
    };
    class Rectangle: public Shape{...};
    class Ellipse: public Shape{...};
    ```
- 声明一个virtual函数的目的就是为了让derived classes只继承函数接口:
- 声明简朴的非纯impure virtual函数的目的就是让derived classe继承该函数的接口和**缺省实现**：你必须支持某接口，如果你不想自己实现可以使用缺省版本。但是这种实现可能造成危险，因为这里缺少“**只有在明确要求的情况下使用缺省版本**”的强制性，可能导致新派生的子类默认使用了父类的缺省实现而不自治。针对该痛点的解决办法又两个：
    - 以不同的函数分别提供接口和默认实现
      ```c++
      class Airplane {
      public:
            virtual void fly(const Airport& destination) = 0;
        ...
      protected:
            void defaultFly(const Airport& destination);
        ...
      };
      void Airplane::defaultFly(const Airport& destination){
        ...// default method
      };

      class ModelA : public Airplane {
      public:
            virtual void fly(const Airport& destination){
                defaultFly(destination);
            }
      };

      class ModelB : public Airplane {
      public:
            virtual void fly(const Airport& destination){
                defaultFly(destination);
            }
      };
      ```
    - pure virtual函数必须在derived classes中重新声明，但是他们自己也可以拥有自己的实现。跟第一种方式相比，只是避免了过度雷同的函数名称引起的命名空间的污染
      ```c++
      class Airplane {
      public:
            virtual void fly(const Airport& destination) = 0;
        ...
      };
      void Airplane::fly(const Airport& destination){
        ...// pure virtual可以拥有自己的实现
      };

      class ModelA : public Airplane {
      public:
            virtual void fly(const Airport& destination){
                Airplane::fly(destination);
            }
      };

      class ModelB : public Airplane {
      public:
            virtual void fly(const Airport& destination){
                Airplane::fly(destination);
            }
      };
      ```

- 声明non-virtual函数的目的是为了让derived class继承函数的接口以及一份强制性实现,即任何derived class都不应该尝试改变。

## 条款35： 考虑virtual函数以外的其他选择(Consider alnatives to virtual functions)

### 基本概念
- NVI手法：Non-Virtual Interface，就是令客户通过public non-virtual成员函数调用private virtual函数。它是所谓的**Template Method设计模式**的独特表现形式。NVI的优点是：可以在调用virtual函数之前做一些事前和事后的工作，即可以控制在合适的地方调用。它允许dirived class重新定义virtua函数，从而赋予“如何实现机能”的控制能力，但是base class保留“何时调用”的能力。如下的例子中，是一个游戏中人物的设计，每种人物可能有不同的计算健康值的诉求：
    ```C++
    class GameCharacter {
    public:
    int healthValue() const {
        // do some pre thing... 
        int retVal = doHealthValue();
        //do some post thing...
        return retVal;
    }
    private:
        virtual int doHealthValue() const{
            return 42;
        }
    };
    class EvilBadGuy:public GameCharacter {
        private:
            // Derived can re-define it if necessary
            virtual int doHealthValue() const {
                return 5;
            }
    };
    int main() {
        GameCharacter *p = new EvilBadGuy();
        cout << p-> healthValue() << endl;
        delete p;
    }
    ```
- Template Method设计模式：Template Method is a behavioral design pattern that defines the skeleton of an algorithm in the superclass but lets subclasses override specific steps of the algorithm without changing its structure.（定义一个操作中算法的骨架，而将一些操作延迟到子类中。TemplateMethod使得子类可以不改变一个算法的结构，即可重定义该算法的某些特定步骤。）
- Strategy设计模式：Strategy is a behavioral design pattern that turns a set of behaviors into objects and makes them interchangeable inside original context object.（在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。）
- 借由Funtion Pointers实现Strategy模式：还是刚才GameCharacter的例子，我们可以要求每个人物的构造函数接受一个指针，指向一个健康值计算函数，而我们可以调用该函数进行实际计算。这样可以达到更灵活的能力：同一人物类型的不同对象可以拥有不同的健康值计算函数，某已知人物对象的计算函数还可以动态调整：
    ```c++
    class GameCharacter;
    int defaultHealthCalc(const GameCharacter& gc );
    class GameCharacter {
    public:
        typedef int (*HealthCalcFunc)(const GameCharacter& );
        explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc): healthFunc(hcf) {}
        int healthValue() const {
            return healthFunc(*this);
        }
        void sethealthFunc(HealthCalcFunc hcf) {
            healthFunc = hcf;
        }
    private:
        HealthCalcFunc healthFunc;
    };

    class EvilBadGuy : public GameCharacter {
    public:
        explicit EvilBadGuy(HealthCalcFunc hcf = defaultHealthCalc) : GameCharacter(hcf) {}
        //...
    };

    int loseHealthQuickly(const GameCharacter& gc );
    int loseHealthSlowly(const GameCharacter& gc );
    // ...
    EvilBadGuy dbg1(loseHealthQuickly);
    EvilBadGuy dbg2(loseHealthSlowly);
    // ...
    dbg1.sethealthFunc(loseHealthSlowly);
    ```
- 借由std::function完成Strategy模式:就是将上述中的函数指针改为std::function对象，这样：完成健康值计算函数的就不一定必须是函数指针，还可以是仿函数对象、lambda表达式等。
- 传统的Strateg模式：传统的Strateg模式将会把健康值计算函数做成一个单独的继承体系中的virtual成员函数：GameCharacter是一个继承体系的根，HealthCalcFunc是另一个继承体系的根，每一个GameCharacter对象都含有一个指针指向一个HealthCalcFunc体系的对象
    ```txt
                    +-----------------------+        +----------------+
                    |   GameCharacter       |        |HealthCalcFunc  |
                    |                       <--------+                |
                    +----------^------------+        +------^---------+
                               |                            |
                               |                            |
                               |                            |
                               |                            |
   +-----------------------+---+              +----------+--+--------------------+
   |  EviBadGuy            |   |              SlowHealth |        |QuickHealthLoser
   |                       |   |              +----------+        +--------------+
   +-----------------------+   |
                               |
                               |
                     +---------+---------+
                     |EyeCandyCharacter  |
                     +-------------------+

    ```
    ```C++
    class GameCharacter;
    class HealthCalcFunc {
    public:
        virtual int calc(const GameCharacter& gc) const {
            //...
        }
    }
    HealthCalcFunc defultFunc;
    class GameCharacter {
    public:
        explicit GameCharacter(HealthCalcFunc* phcf = &defaultHealthCalc): pHealthFunc(phcf) {}
        int healthValue() const {
            return pHealthFunc->cals(*this);
        }
        void sethealthFunc(HealthCalcFunc* phcf) {
            pHealthFunc = phcf;
        }
    private:
        HealthCalcFunc *pHealthFunc;
    };
    ```
### 条款理解：

- 这个世界还有其他许多道路，值得我们花时间研究
- NVI是Template Method设计模式的一种特殊形式，有其独特的优势
- 将virtual函数替换为函数指针类型的成员变量，这是Strategy设计模式的一种表现形式；使用std::function替换函数指针类型将提升更多灵活性
- 传统的Strateg模式，将继承体系中的virtual函数替换为另一个继承体系的指针
  
## 条款36： 绝不重新定义继承而来的non-virtual函数(Never redefine an inherited non-vitrual function)

### 基本概念

- non-virtual函数是静态绑定，而virtual函数是动态绑定。所谓动态绑定，可以直接理解为：使用基类指针指向子类对象，当通过基类指针调用成员函数，会真正地的调用对应子类的成员函数，

### 条款理解

- 任何情况下都不要重新定义一个继承而来的non-virtual函数。

## 条款37： 绝不重新定义继承而来缺省参数值(Never redefine a function's inherited default parameter value)

### 基本概念

- 静态类型：所谓静态类型，就是程序中被声明时所采用的类型。如下面程序中，ps/pc/pr的静态类型都是`Shape*`。
    ```c++
    class Shape {
    public:
        enum ShapeColor {Red, Green, Blue};
        virtual void draw{ShapeColor color = Red} const = 0;
        ...
    };

    class Rectangle : public Shape {
        public:
        // draw has defferent default parameter value, damn.
        virtual void draw(ShapeColor color = Green) const;
        ...
    };

    class Circle : public Shape {
        public:
        virtual void draw(ShapeColor color) const;
        ...
    };

    Shape *ps;
    Shape *pc = new Circle();
    Shape *pr = new Rectangle();
    ```
- 动态类型：动态类型则是指“目前所指对象的类型”。pc的动态类型是`Circle*`,pr的动态类型是`Rectangle*`
- virtual函数系动态绑定，调用一个cirtual函数时究竟调用哪一份函数实现代码。取决于动态类型。 
- virtual函数是动态绑定，但是缺省参数却是静态绑定，如果调用`pr->draw()`时调用的是`Rectangle::draw()`但使用的参数却是`Red`即base class指定的默认参数

### 条款理解

- 绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，但是virtual函数却是动态绑定的
- 但是如果你遵循规则，又会出现重复代码并且相依性，如果base class的缺省值变了，所有“重复给定缺省参数值”的derived classes也必须改变
    ```c++
    class Shape {
    public:
        enum ShapeColor {Red, Green, Blue};
        virtual void draw{ShapeColor color = Red} const = 0;
        ...
    };

    class Rectangle : public Shape {
        public:
        // draw has defferent default parameter value, damn.
        virtual void draw(ShapeColor color = Red) const;
        ...
    };
    ```
- 如果遭遇这种麻烦，可以建议使用NVI进行优化设计
    ```C++
    class Shape {
    public:
        enum ShapeColor {Red, Green, Blue};
        void draw(ShapeColor color = Red) {
            doDraw(color);
        };
        ...
    private：
        virtual void doDraw(ShapeColor color) const = 0;
    };

    class Rectangle : public Shape {
        public:
        ...
        private：
        virtual void doDraw(ShapeColor color);
    };

    class Circle : public Shape {
        public:
        ...
        private：
        virtual void doDraw(ShapeColor color);
    };
    ```

## 条款38： 通过复合塑模出has-a或“根据某物实现出”(Model "has a" or "is-implemented-in-terms-of" through composition)

## 基本概念

- 复合： 复合（composition）是一种类型之间的关系。当某种类型的对象内部含其他种类型的对象，便是这种关系。复合有很多同义词，例如：layering（分层）、containment（内含）、aggregation（聚合）、embedding（内嵌）
    ```C++
    class Address{...};
    class PhoneNumber{...};

    class Person {
    public:
        ...
    private:
        std::string name; //合成之物
        Address address;
        PhoneNumber voiceNumber;
        PhoneNumber foxNumber;
    }
    ```
- 当复合发生于应用域对象之间，就是has-a关系，如果发生在实现域，则是表现is-implemented-in-terms-of。上述的`Person`和`Address`就是has-a的关系。
- 如果因为要节省空间，考虑从list实现一个set类型，应该如何实现呢。因为考虑条款32的要求，不适合使用继承。正确的做法是从list实现出Set对象
    ```C++
    template<class T>
    class Set {
    public:
        bool member(const T& item) const;
        void insert(const T& item);
        void remove(const T& item);
        std::size_t size() const;
    private:
        std::<list> rep;
    };

    template<class T>
    bool Set<T>::member(const T& item) const {
        return std::find(rep.begin(), rep.end(), item) !=  rep.end();
    }

    template<class T>
    void Set<T>::insert(const T& item) {
        if (!member(item)){
            rep.push_back(item);
        }
    }

    template<class T>
    void Set<T>::remove(const T& item) {
        auto it = std::find(rep.begin(), rep.end(), item);
        if (it != rep.end()){
            rep.erase(it);
        }
    }
    template<class T>
    size_t Set<T>::size() {
        return rep.size();
    }
    ```
## 条款理解 

- 当复合发生于应用域对象之间，就是has-a关系，如果发生在实现域，则是表现is-implemented-in-terms-of，即根据某物实现出。复合与public继承完全不同

## 条款39： 明智而审慎地使用private继承(Use private inheritance judiciously)

## 基本概念

- private继承并不意味着is-a关系。如果classes之间的继承关系是private，编译器不会自动将一个derived class对象转化为base class对象； 通过private继承而来的所有成员，在derived class中都会变为private属性。
- private继承纯粹只是一种实现技术，意味着implemented-in-terms-of（根据某物实现出）。如果你让class D以private继承class B，你的用意是为了使用class B中已经备妥的特性，不是因为B和D存在任何观念上的关系。复合的意义也是implemented-in-terms-of，那么如何取舍-----优先使用复合，必要时才使用private继承。
- 使用private继承的场景1：当两者之间的关系是implemented-in-terms-of，但是derived class想访问base class的protected成分或者是想重新定义virtual函数。例如，我们想设计一个类，这个类能周期性的审查自己的数据（例如成员函数被调用了多少次值类的）。我们发现有一个定时器类，其中的定时滴答函数是个虚函数，而我们刚好可以重新定义该虚函数。可以利用private继承实现
    ```C++
    class Timer {
    public:
        explicit Timer(int tickFrequency);
        virtual void onTick( )const; // 定时器没滴答一次，该函数自动被调用
    };

    class Widget : private Timer {
    private:
        virtual void onTick( )const; // 重新定义，使得定时审查自身相关数据
    };
    ```
- 使用private继承的场景1补充：解决一个设计问题的方法不只有一种，训练自己思考多种方法是值得的。上述的方法可以使用内嵌类并使用复合的方式解决，并且还有两个其他优势：1、防止子类重新定义虚函数；2、可以进一步将Widget的编译依存性降至最低，可以在Widget中使用指针指向WidgetTimer对象，这样Widget可以只带着一个简单的WidgetTimer声明式。
    ```C++
    class Widget {
    private:
        class WidgetTimer : public Timer {
            virtual onTick() const;
            ...
        };
        WidgetTimer timer;
    ```
- 使用private继承的场景2----EBO：所谓的EBO，空白基类最优化（Empty Base Optimization）。C++裁定凡是独立对象都必须有非零大小，而使用private继承可以保证空白基类最优化。
    ```C++
    class Empty {
    public:
        static void print(){
            cout << "hello" << endl;
        }
    };

    class Derived1 : private Empty {
        private:
        int m;
    };
    class Derived2 {
        private:
        Empty e;
        int m;
    };

    int main() {
        Empty e;
        Derived1 d1;
        Derived2 d2;
        cout << sizeof e << "-" << sizeof d1 << "-" << sizeof d2  << "-" << sizeof(int)<< endl;
    }
    // The output is:1-4-8-4
    ```

## 条款理解 

- private继承意味是is-implemented-in-terms-of, 通常优先私用复合，但是再两种场景下可能需要使用private继承：
  - 当derived class需要访问protected成员或者重新定义虚函数
  - 考虑EBO

## 条款40： 明智而审慎地使用多重继承(USe multiple inheritance judiciously)

## 基本概念：

- 歧义（ambiguity）：程序可能从多个base class继承相同的名称，会导致歧义（此时并不会检查可取用性）。解决办法是明白指出调用哪一个base中的名称。`obj.Base1::someFun()`.
- "钻石型多重继承"----某个继承体系中出现某个base class和derived class之间有一条以上的通路.在钻石型多重继承中
    ```C++
    class File {...};
    class InputFIle : public File {...};
    class OutputFIle : public File {...};
    class IOFile: public InputFIle,
                public OutputFIle
                {...};
    ```
- 需要考虑的是：是否要将base class中的每个成员变量经由每条路径复制？在C++中两种都支持，但是默认是“将base class中的每个成员变量经由每条路径复制”。如果这不是想要的行为，必须将base class成为一个`virtual base class`.但是virtual继承是有代价的，类的体积更大、访问成员变量的速度更慢、derived class不论多远都必须肩负初始化virtual base的责任。建议如非必要不要使用virtual base，如果必须使用尽可能避免在里面放置数据
    ```C++
    class File {...};
    class InputFIle : virtual public File {...};
    class OutputFIle : virtual public File {...};
    class IOFile: public InputFIle,
                public OutputFIle
                {...};
    ```
- 多重继承的一个合理的使用场景：将“public继承自某接口”和“private继承自某实现”（因为要利用已有的类且需要重新定义须函数）结合在一起：
    ```C++
    class IPerson {
    public:
        virtual ~IPerson();
        virtual std::string name() const = 0;
        virtual std::string birthDate() const = 0;
    };

    class DatabaseID {...};
    class PersonInfo {
    public:
        explicit PersonInfo(DatabaseID pid);
        virtual ~PersonInfo();
        virtual const char* theName() const;
        virtual const char* theBirthDate() const;
        virtual const char* valueDelimOpen() const;
        virtual const char* valueDelimClose() const;
    };

    class CPerson : public IPerson, private PersonInfo {
    public:
        explicit CPerson(DatabaseID pid) : PersonInfo(pid){}
        virtual std::string name() const {
            return PersonInfo::theName();
        }
        virtual std::string birthDate() const {
            return PersonInfo::thebirthDate();
        }
    private:
        const char* valueDelimOpen() const {
            return "";
        }
        const char* valueDelimClose() const {
            return "";
        }
    };
    ```

## 条款理解 

- 在多重继承下，需要考虑歧义；同时针对钻石型继承，需要关注virtual继承的必要性和限制
- 多重继承有其使用的正当场景----将“public继承自某接口”和“private继承自某实现”结合在一起


## 7 模板与泛型编程(Templates and Generic Programming)

## 条款41： 了解隐式接口和编译期多态(Understand implicit interfaces and compile-time polymorphism)

## 基本概念

- 面向对象编程的世界总是以显式接口（explicit interface）和运行期多态（runtime polymorphism）解决问题。所谓的显式接口，可以理解为基于函数的签名式。例如：w的类型被声明为Widget&， 所以w必须支持Widget接口；由于Widget的某些成员函数是virtual，所以w对那些函数的调用表现出运行期多态（runtime polymorphism）
    ```C++
    class Widget {
    public:
        Widget();
        virtual ~Widget();
        virtual std::size_t size() const;
        virtual void normallize();
        void swap(Widget& other);
        ...
    };

    void doProcessing(Widget& w) {
        if (w.size() > 10 && w != someNastyWidget) {
            Widget temp(w);
            temp.normallize();
            temp.swap(w);
        }
    }
    ```
- Templates及泛型编程的世界，隐式接口（implicit interfaces）和编译器多态（compile-time polymorphism）移到前头了。w必须支持哪一种接口，由template中执行于w身上的操作来决定； 在编译器出现“多态”行为，即“以不同的template参数具现化function templates”会导致调用不同的函数。隐式接口基于“有效表达式”
    ```C++
    template<typename T>
    void doProcessing(T& w){
        if (w.size() > 10 && w != someNastyWidget) {
        Widget temp(w);
        temp.normallize();
        temp.swap(w);
    }
    ```

## 条款理解 

- 面向对象和泛型编程都支持接口和多态
- 对面向对象而言，接口是显式的，以函数签名为中心；多态是同过virtual函数实现的
- 对泛型编程而言，接口是隐式的，基于“有效表达式”。多态则是通过template具现化和函数重载解析（function overloading resolution）发生于编译期

## 条款42： 了解typename的双重意义（Understand the tho meaning of the typename）

## 基本概念

- 当我们声明template参数时，class关键字和typename关键字意义完全相同
    ```C++
    template<class T>class Widget;
    template<typename T>class Widget;
    ```
- 然而class和typename并不总是等价的，typename有它独特的用武之地：
  - 嵌套从属名称：template中出现的名称如果相依于某个template参数，则该名称成为从属名称；如果从属名称在class中呈嵌套状，我们称它为嵌套从属名称。
  - 一般性的规则：当要在template中指涉一个嵌套从属类型名称，就必须在紧临它的前一位置使用typename关键字。这背后的原因是编译器默认嵌套从属名称不是一个名称，所以需要特别说明。
    ```C++
    template<typename C>
    void print2nd(const C& container){
        if (container.size() > 2) {
            typename C::const_iterator iter(container.begin()); // typename是必须的
            ++iter;
            int value = *itr;
            std::cout << value;
        }
    }
    ```
  - 一般性的规则的例外：不可以出现在base class list内的嵌套从属类型名称之前，也不可以在member initialization list中作为base class的修饰词。 
     ```C++
    template<typename T>
    class Derived : public Base<T>::Nested { // No typename
    public:
        explicit Derived(int x)
        : Base<T>::Nested(x) { // // No typename
            typename Base<T>::Nested temp; // Need typename
            ...
        }
    }
    ```
  - 一般性的规则的补充：使用`typedef typename `简化编码，事实上在现代C++，可以使用auto :)
    ```C++
    template<typename T>
    void plusOneAndPrint(const T& itr){
        typename std::iterator_traits<T>::value_type temp(*itr);
        cout << ++temp << endl;
    };

    template<typename T>
    void plusOneAndPrintV1(const T& itr){
        typedef typename std::iterator_traits<T>::value_type value_type;
        value_type temp(*itr);
        cout << ++temp << endl;
    };

    template<typename T>
    void plusOneAndPrintV2(const T& itr){
        auto temp(*itr);
        cout << ++temp << endl;
    };

    int main() {
        vector<int>v{0,1};
        plusOneAndPrint(v.begin()+1);
        plusOneAndPrintV1(v.begin()+1);
        plusOneAndPrintV2(v.begin()+1);
    }
    ```

## 条款理解

- 声明template参数时，typename和class是等价的
- 但是typename有独特的用武之地，主要是涉及到嵌套从属名称时。

## 条款43： 学习处理模板化基类内的名称(Know how to access names in templatized base classes)

## 基本概念

- 全特化：template MsgSender针对类型CompanyZ全特化了。
    ```C++
    class CompanyA {
    public:
        void sendClearText(const string& msg);
        void sendEncrypted(const string& msg);
    };

    class CompanyB {
    public:
        void sendClearText(const string& msg);
        void sendEncrypted(const string& msg);
    };

    class MsgInfo {...};

    template<typename Company>
    class MsgSender {
    public:
        void sendClear(const MsgInfo& Info){
            string msg;
            // convert Info into msg;
            Company c;
            c.sendClearText(msg);
        }
        void sendSecret(const MsgInfo& Info){
            ...
        }
    }

    class CompanyZ {
    public:
        void sendEncrypted(const string& msg); // CompanyZ only support  sendEncrypted
    };

    template<>
    class MsgSender<CompanyZ> { //针对CompanyZ的全特化版本
    public:
        void sendSecret(const MsgInfo& Info){
            ...
        }
    }
  ```
- 默认情况下，模板类的继承子类无法在derived class中直接调用父类的方法，原因就是上述的全特化版本可能存在，它无法提供一个和一般性template相同的接口，因此C++默认拒绝。下述代码将无法通过编译。
    ```C++
    template<typename Company>
    class LoggingMsgSender : public MsgSender<Company> {
    public:
        void sendClearMsg(const MsgInfo& Info){
            // do some logging work
            sendClear(Info); // Compile Error!
            // do some logging work
        }
        ...
    }
    ```
- 解决办法有三种：
  - 在base class调用之前加上`this->`
    ```C++
    template<typename Company>
    class LoggingMsgSender : public MsgSender<Company> {
    public:
        void sendClearMsg(const MsgInfo& Info){
            // do some logging work
            this->sendClear(Info);
            // do some logging work
        }
        ...
    }
  - 使用using声明
      ```C++
    template<typename Company>
    class LoggingMsgSender : public MsgSender<Company> {
    public:
        using MsgSender<Company>::sendClear;
        void sendClearMsg(const MsgInfo& Info){
            // do some logging work
            sendClear(Info);
            // do some logging work
        }
        ...
    }
  - 明白指出使用base class的方法.这往往是最不满意的解法，因为这会关闭virtual函数绑定
        ```C++
    template<typename Company>
    class LoggingMsgSender : public MsgSender<Company> {
    public:
        void sendClearMsg(const MsgInfo& Info){
            // do some logging work
            MsgSender<Company>::sendClear(Info);
            // do some logging work
        }
        ...
    }

## 条款理解

- 默认情况下，模板类的继承子类无法在derived class中直接调用父类的方法，原因就是上述的全特化版本可能存在，它无法提供一个和一般性template相同的接口，因此C++默认拒绝
- 有三种方法可以在derived class中调用父类的方法，在base class调用之前加上`this->`、使用using声明、明白指出使用base class的方法

## 条款44： 将参数无关的代码抽离templates（Factor paramter-independent code out of template）

## 附录 运算符顺序
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
```C++
// 将ip指向的对象+1
*ip += 1
++*ip
(*ip)++
```


## 附录 C++ Access
- A member of class's access

    | Access    | 说明                             |
    | --------- | -------------------------------- |
    | private   | 仅能够被该类其他成员函数使用     |
    | protected | 可以被该类其他成员函数和子类使用 |
    | public    | 可以被任意函数调用使用           |
- Base class's access

    | Access    | 说明                                                       |
    | --------- | ---------------------------------------------------------- |
    | private   | base类的public、protected成员仅能被derive类的成员使用      |
    | protected | 在private的基础上+derive类子类的成员使用                   |
    | public    | 在protected的基础上 + base类的public可以被任意函数调用使用 |

测试程序
```C++
// 将ip指向的对象+1
class Parent {
public:
    void publicMethodOfParent(){ 
        privateMethodOfParent();    // it's ok
        protectedMethodOfParent(); // it's ok
        cout << "publicMethodOfParent()" << endl; };
protected:
    void protectedMethodOfParent() { 
        privateMethodOfParent(); // it's ok
        cout << "protectedMethodOfParent()" << endl; 
    };
private:
    void privateMethodOfParent(){ cout << "privateMethodOfParent()" << endl; }
};
class PrivateDerived : private Parent {
protected:
    void protectedMethodOfPrivateDerived(){
        // privateMethodOfParent(); nok
        protectedMethodOfParent();
        publicMethodOfParent();
    }    
};
class ProtectedDerived : protected Parent {
protected:
    void protectedMethodOfPrivateDerived(){
        //privateMethodOfParent(); nok
        protectedMethodOfParent();
        publicMethodOfParent();
    }    
};
class PublicDerived : public Parent {
protected:
    void protectedMethodOfPublicDerived(){
        //privateMethodOfParent(); nok
        protectedMethodOfParent();
        publicMethodOfParent();
    }    
};

int main()
{
    Parent p;
    // p.privateMethodOfParent(); nok
    // p.protectedMethodOfParent(); nok
    p.publicMethodOfParent();
    ProtectedDerived p1;
    // p1.publicMethodOfParent(); // non error: 'Parent' is not an accessible base of 'ProtectedDerived'
    PublicDerived p2;
    p2.publicMethodOfParent(); 
    return 0;    
}
```

