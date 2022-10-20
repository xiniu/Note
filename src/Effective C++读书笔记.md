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

- 条款理解01：如果你自己没声明，编译器就会生成一个默认的copy构造、copy assignment操作符和一个析构函数。如果没有声明任何构造函数，编译器会生成一个default构造函数（有声明拷贝构造，会不会有default构造呢？ no）。所有这些函数是public且inline。因此如果你写下`class Empty { }`就好比写下如下代码：
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

- 条款理解01： polymorphic base class应该声明一个virtual析构函数。如果class带有任何一个virtual函数，它就应该拥有一个virtual。C++明确指出当derived class经由一个base class指针删除，而该base class带着一个non-virtual的析构函数，其行为式未定义的
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
- 条款理解02： 如果一个类不含其他virtual函数，意味着它并不意图被用作一个base class， 千万不要为其声明virtual析构，带来的影响是：对象会由很客观膨胀，且于其他语言交互会出更多问题。同时，不要试图从不带virtual的类派生，尤其是标准容器等

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

- 条款理解02：有时候方便期间，可以提供隐式转换的能力，从资源管理隐式转化为原始资源
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

    Font f(getFont());
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

    Font f(getFont());
    changeFontSize(f, someSize); // Font隐式转化为FontHandle
    ```


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
    | protected | 在private的基础上+derive类字类的成员使用                   |
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
