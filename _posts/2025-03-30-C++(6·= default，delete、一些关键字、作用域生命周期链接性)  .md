---
title: c++(6·= default、关键字、作用域、定义、数据类型、初始化)
date: 2025-03-30 12:00:00 +0800
categories: [C++]
tags: []     # TAG names should always be lowercase
math: true
---
# std::chrono

一个用于处理时间和日期的库

```c++
#include <chrono>

auto start = std::chrono::steady_clock::now();
//……
auto end = std::chrono::steady_clock::now();
std::chrono::duration<double> diff = end - start;

```

steady_clock：单调时钟，时间无法后退

* steady_clock::now()返回现在的时间
* steady_clock::time_point()返回更精确的时间点

high_resolution_clock：提供最高精度的时钟

# *this

this指针:

指向调用该成员函数的对象实例，因此指针类型是className * ，它通常可以省略

作用：

* 解决名称冲突：比如函数的形参和成员变量的名称一致，用this->成员，表示此为类成员
* 返回当前对象，return *this

# inline

内联函数：避免频繁调用的简单函数消耗栈内存的问题

特点：

* 如果包含复杂的控制语句例如 while、switch，或复杂结构递归，编译器可能会忽略 inline 建议
* 成员函数如果定义在类中，默认是内联的
* 如果在类中未给出成员函数定义，而又想内联该函数的话，那在类外定义的开头加上 inline
* 优点：
  * 在编译阶段，将函数在调用位置进行替换展开，这样节省了函数调用的开销（保存寄存器、传递参数、跳转到函数地址、返回结果），从而提高执行效率
* 缺点：
  * 消耗内存空间：导致可执行文件增大，由于CPU缓存的空间是有限的，超出后不得不从内存中读取代码，从而降低程序的运行效率
  * 编译时间增加：由于在构建阶段处理内联函数，因此会增加构建的时间
* 使用场景：所以说应该保证inline不出现复杂的控制语句/结构，以免消耗大量内存反而降低了运行效率

# = default 、= delete

类中有几种特殊的默认成员函数：默认构造，拷贝构造，拷贝赋值，移动构造，移动赋值，析构函数，它们分别有构造，初始化，赋值，销毁的作用

```c++
PilotEngine(const PilotEngine&) = delete;
PilotEngine& operator=(const PilotEngine&) = delete;
```

= default 、= delete在这些特殊的成员函数中的应用：

* = default 创建默认函数
* = delete  删除默认函数（形参名是可以省略的）

构造：

* 一旦自定义了构造函数，编译器不会隐式的生成一个默认的构造函数，如果调用了默认构造函数会在编译时报错，
* 为了避免这种情况，我们的思路就是显示的重载一个默认的构造函数，但是如果使用= default，可以让编译器隐式的生成一个默认的构造函数，它的执行效率更高

拷贝构造，拷贝赋值，移动构造，移动赋值

* 无论是否自定义，编译器始终会隐式生成默认版本
* 但有时不希望类实例进行拷贝构造或拷贝赋值，那么将它们使用关键字 =delete 标记，相当于删除了它们

析构函数

* 一个类中有且仅有一个析构函数，所以=delete不能用于析构函数，它可以被重载，且会调用重载的版本

# 作用域、生命周期、链接性

**变量：**

作用域（5类）：可被访问的范围，它是由对象的定义位置和关键字决定的

* 全局：在命名空间、类、函数外，在整个程序可见，不过其他文件要包含#include/extern才能访问
* 文件：在命名空间、类、函数外定义，用static限制，从而其他文件不可以通过#include/extern访问它
* 命名空间：对整个命名空间内直接访问，外部可以通过作用域解析运算符Namespace::name 访问
* 类：在类(class/struct)内可以直接访问，类外通过 类对象/指针/引用/作用域解析运算符 :: 访问（具体是否可以被类外访问取决于成员访问修饰符）
* 局部：在函数/代码块({})中定义，仅在内部访问,且会隐藏外部的同名变量

生命周期（3类）：在内存中存在的时间，它是由对象的定义位置和关键字决定的

* 自动（栈）：局部作用域的在离开作用域自动销毁
* 静态（静态存储区）：全局作用域的，被static修饰的，在程序结束被销毁
* 动态（堆）：程序员手动分配和销毁

链接性（3类）：能否跨文件访问，它是由对象的定义位置和关键字决定的（构建过程的链接阶段）

* 外部链接：可被其他文件访问
* 内部链接：仅当前文件可见
* 无链接：仅当前作用域可见

# 定义、声明、初始化、赋值：

**变量：**

定义 + 声明：类型 + 自定义名称，一定会初始化，如果没有显示初始化就会执行默认初始化，分配内存空间

声明：告诉编译器标识符的存在，不分配内存空间，声明可以多次，定义只能一次

初始化：首次为内存块设置值

赋值：在定义之后修改值都叫做赋值

# static

static静态：用来修改变量/函数的作用域和生命周期

```c++
#include <iostream>
using namespace std;

int n = 1; //全局变量

void func()
{
	static int a = 2; // 静态局部变量
	int b = 5; // 局部变量
	a += 2;
	n += 12;
	b += 5;
	cout << "a:" << a
		<< " b:" << b
		<< " n:" << n << endl;
}

void main()
{
	static int a; // 静态局部变量
	int b = -10; // 局部变量
	cout << "a:" << a
		<< " b:" << b
		<< " n:" << n << endl;
	b += 4;
	func();
	cout << "a:" << a
		<< " b:" << b
		<< " n:" << n << endl;
	n += 10;
	func();

	system("pause");
}
```

* a初始为0，
  * func中内部作用域a隐藏了外部作用域a，输出4，
  * 回到外部局部作用域结束，返回0，
  * func，由于生命周期为静态，局部作用域再次可见，定义并非再次创建，而是返回静态内存中的a，返回6
* b初始为-10，
  * 进入内部隐藏外部，返回10，
  * 返回外部，由于+4返回-6
  * func，由于生命周期自动，上一次的销毁重新创建，返回10
* n初始为1，
  * func没有定义同名的变量，不会隐藏，直接修改，返回13
  * 返回外部，由于被修改返回13，
  * +10，进入func，再次+12，返回35

全局静态变量：全局作用域用 static 修饰的变量，作用域限制为文件，外部不可访问，生命周期静态

局部静态变量：局部作用域用 static 修饰的变量，作用域局部，生命周期静态, 如果再次进入函数它不会因为定义再次创建，而会从静态区查找对象

```c++
class Example {
public:
    static int count;  
    static const double PI; 
    static void printCount();
};
//类外定义
int Example::count = 0;   
const double Example::PI = 3.14; 
void Example::printCount() {
    std::cout << "Count: " << count << std::endl;
}
```

静态成员变量：类中用 static 修饰的变量，类作用域（可以理解为从某个类对象提升为类），生命周期静态，保证了类内的唯一性

静态成员函数：类中用 static 修饰的函数，类作用域（可以理解为从某个类对象提升为类），生命周期静态，保证了类内的唯一性

静态成员特点：

* 是和类相关的，而不是和类的具体对象相关的
* 可以通过作用域解析运算符 :: 直接调用
* 必须在类外定义，定义时不添加static关键字
* 没有隐藏的this指针，不能访问任何非静态成员
* 遵守访问修饰符的原则

# extern

声明变量/函数为外部链接性

```c++

//1.cpp
int global_var = 42; 
void print_hello() {
  std::cout << "Hello!";
}
int a = 10;//定义a
//extern int a = 20;//定义a，重复定义错误

//2.cpp
extern int global_var; 
extern void print_hello(); 
//extern int a = 20;//定义a，重复定义链接错误

```

注意：

* extern仅可以修饰全局变量/函数，不能修饰局部变量
* extern可以放在任意的位置，但作用域是不同的
* static 限制作用域，extern 扩展作用域
* 如果在extern的时候给变量赋值 == 定义

问题：为什么要用extern 函数呢？直接#include相应的头文件不可以吗？

* 不会引入大量头文件，进而不会引入大量的无关函数,会加速程序的编译

# const

修饰对象/函数：

常量对象：如果不希望去修改某个对象的值，就声明为const，以防止程序员意外修改

常量成员函数：不可以修改类的成员数据

* 或者某个函数表明它不会修改类成员数据，就声明为const，以防止程序员意外修改
* 如果不希望修改类对象中的数据，将类对象声明为const，并用const修饰类成员函数，表明该函数不会修改类成员数据，这样这个函数可以被常量类对象调用

注意：

* const member function和non-const member function的同名函数不同形参，是属于函数重载
* 非常量对象可以调用常量成员函数，也可以调用非常量成员函数（因为非常量对象内部的数据是否被修改都可以），而常量对象则只能调用常量成员函数（因为常量对象内部的数据不能被修改）
* 当成员函数的const版本和non-const版本同时存在时，常量对象只能调用const版本，而非常量对象只能调用non-const版本
* 常量成员函数中，只能调用其他常量成员函数，为了确保不会间接修改对象的状态

const对作用域、生命周期、链接性的影响：

```c++
// file1.cpp
const int x = 10;  //定义时必须初始化
extern const int y = 10; 

// file2.cpp
extern const int x;  // ❌ 错误
extern const int y;  // ✅ 正确
```

只会影响全局const，修改为内部链接，文件作用域

如果想要被外部访问，必须在定义时加extern，这样表示这个全局const为外部链接，全局作用域

# constexpr

常量表达式（const expression）：用于修饰对象/函数，它比传统的 const 更严格，是指值不会改变且编译期可以得到结果的表达式

```c++
constexpr int max_size = 100; //定义时必须初始化
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}
```

注意：

* constexpr 变量一定是const的，但const变量不一定是constexpr
* 当我们在运行时不想改变它的值，并且希望能在编译期间就计算的确定值，通常声明为constexpr来保证为常量表达式, 以防止程序员意外修改
* constexpr 变量，应该保证初始值为字面值/字面值与constexpr 变量的运算/constexpr函数，否则会报错
* constexpr函数，返回类型和所有形参的类型必须是字面值类型（不可以是void）
* 并不是说使用了constexpr这个关键字就保证了被修饰的不会被修改和在编译器计算，而是通过编译器的报错提醒我们，进行了不符合预期的命令

# 数据类型

在编程时，需要用到各种变量来存储各种信息，不同的数据类型决定内存分配的存储空间的大小

```c++

数据类型分类（C++）
├── 内置类型（Built-in Types）: C++语言本身直接支持的数据类型
│   ├── 基本数据类型（Fundamental Types）: 不可再分解的最小数据类型
│   │   ├── 算术类型（Arithmetic Types）: 支持算术运算的基本数据类型
│   │   │   ├── 整型（Integral Types）
│   │   │   │   ├── 字符类型
│   │   │   │   │   ├── char 1字节，ASCII
│   │   │   │   │   ├── wchar_t 宽字符2字节
│   │   │   │   │   ├── char8_t 1 字节，明确表示 UTF-8 编码的 Unicode 字符
│   │   │   │   │   ├── char16_t 2 字节，表示 UTF-16 编码的 Unicode 字符
│   │   │   │   │   └── char32_t 4 字节，表示 UTF-32 编码的 Unicode 字符
│   │   │   │   ├── 布尔类型：bool
│   │   │   │   └── 整数类型
│   │   │   │       ├── short 2 字节
│   │   │   │       ├── int 4 字节
│   │   │   │       ├── long 4/8 字节 L 或 l 后缀
│   │   │   │       └── long long 8 字节 LL 或 ll后缀
│   │   │   └── 浮点类型（Floating-point Types）
│   │   │       ├── float 4字节
│   │   │       ├── double 8字节
│   │   │       └── long double 8字节
│   │   └── void 类型：表示无类型或空类型
│   └── 复合类型（Compound Types）: 可再分解的最小数据类型
│       ├── 指针（Pointer）: T*
│       ├── 数组（Array）: T[N]
│       ├── 引用（Reference）
│       │   ├── 左值引用：T&
│       │   └── 右值引用：T&& 
│       ├── 函数类型（Function Types）
│       └── 限定类型（Qualified Types）
│           ├── const限定：const T
│           └── volatile限定：volatile T
│
├── 标准库类型（Standard Library Types）: C++标准库提供的数据类型
│   ├── 容器类（Containers）
│   │   ├── 序列容器
│   │   │   ├── vector
│   │   │   ├── array (C++11)
│   │   │   ├── deque
│   │   │   ├── forward_list (C++11)
│   │   │   └── list
│   │   ├── 关联容器
│   │   │   ├── set
│   │   │   ├── map
│   │   │   ├── multiset
│   │   │   └── multimap
│   │   └── 无序关联容器 (C++11)
│   │       ├── unordered_set
│   │       ├── unordered_map
│   │       ├── unordered_multiset
│   │       └── unordered_multimap
│   ├── 字符串类：string, wstring
│   ├── 智能指针 (C++11)
│   │   ├── unique_ptr
│   │   ├── shared_ptr
│   │   └── weak_ptr
│   ├── 工具类
│   │   ├── pair
│   │   ├── tuple (C++11)
│   │   └── variant (C++17)
│   └── 其他
│       ├── optional (C++17)
│       ├── any (C++17)
│       └── function (C++11)
│
└── 自定义数据类型（User-defined Types）
    ├── 类类型（Class Types）：自定义数据类型
    │   ├── class
    │   └── struct
    ├── 枚举类型（Enumeration Types）：自定义整数常量集合
    │   ├── 无作用域枚举：enum
    │   └── 有作用域枚举：enum class (C++11)
    └── 联合体：union：多个数据共享同一内存
```

![1743320103564](/assets/img/blog/c++/内置类型.png)

![1743320107935](/assets/img/blog/c++/修饰符.png)

![1743320112363](/assets/img/blog/c++/新增类型.png)

![1743320117682](/assets/img/blog/c++/派生数据类型.png)

![1743320122189](/assets/img/blog/c++/标准库类型.png)

# 初始化方式

```c++
int x; 
```

默认初始化：定义时未显示初始化，全局作用域内置类型初始化为0，局部作用域内置类型值未定义, 类对象调用默认构造函数

```c++
int x{}; 
std::vector<int> v(10); //10个元素
```

值初始化：使用空括号或空花括号进行的初始化，内置类型初始化为0，类对象调用默认构造函数

```c++
std::string s1 = "hello"; 
```

拷贝初始化：使用等号进行的初始化，类调用拷贝构造函数或移动构造函数

```c++
std::string s2(5, 'a');
```

直接初始化：使用有参圆括号或有参花括号初始化，类调用匹配的构造函数

```c++
std::vector<int> v{1, 2, 3};
```

列表初始化：使用有参花括号进行的初始化，调用std::initializer_list构造函数
