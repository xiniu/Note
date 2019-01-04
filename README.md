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

## *Effective C++*读书笔记

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
extern int x;   					 //对象声明式
std::size_t numDigits(int number);   //函数声明式
class Widget;    					 //类声明式

templete<typename T>
class GraphNode；    				//模版类声明式
```

- 签名式：签名可以理解为标示函数的类型，在本书中我们指定参数类型（参数签名）和返回类型。例如`numDigits`的签名是`std::size_t (int)`.需要额外注意的是，C++签名式的官方定义不包含返回类型，这也是重载函数的定义，就是只有参数签名不同
- 定义式：补充声明式遗漏的一些细节。对对象而言，定义式是划分内存的地方；对`function`和`functionc templete`而言，是实现代码逻辑本体的地方；对于`class`和`class templete`而言，是指明他们成员的地方

```C++
int x;   							 //对象定义式
std::size_t numDigits(int number)    //函数定义式
{
	//some logic code.
}
class Widget
{
public:
	Widget();
    ~Widget();
	...
};				 					//类定义式

templete<typename T>
class GraphNode；    			   //模版类定义式
{
public:
	GraphNode();
    
    ~GraphNode();
    ...
}
```  
  
  
