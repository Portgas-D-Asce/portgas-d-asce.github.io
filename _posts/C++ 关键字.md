# static

**隐藏：** 修饰全局变量/函数时，会将其作用域限定在当前文件。

```cpp
static int x = 10;

static void func() {}
```



**延长局部变量生命周期：** 修饰局部变量时，仅会初始化一次，可以延长局部变量的生命周期

```cpp
void func() {
    static int cnt = 0;
    cnt++;
    cout << cnt << endl;
}
```



**修饰类的成员变量/成员函数：** 修饰类成员时，静态成员变量/函数属于类，为类的所有对象共享

```cpp
class Base {
public:
    static int x;
    static void func() {
        cout << 123 << endl;
    }
};

int x::num = 10;
```



# inline

内联函数会在调用处展开，消除了函数调用开销，提高运行速度。inline 仅仅是对编译器的建议，最后能否真正内联，取决于编译器。



递归函数不能设置为内联

```cpp
inline void func() {
    func();
}
```



在类中定义的成员函数默认是内联的

```cpp
class A {
public:
    void func(int x, int y) {}
}

class A {
public:
    void func(int x, int y);
}
inline void A::func(int x, int y) {} 
```



被修饰的函数体较大时，会导致代码膨胀，甚至影响运行效率。



# const 

修饰变量，防止变量被修改。函数形参通常被声明为 const 引用，即避免了值传递，又可以保证参数不被修改

```cpp
const int x = 10;

void func(const vector<int> &nums) {
    for(int x : nums) {
        cout << x << endl;
    }
}
```



修饰指针：

- 当 const 位于 * 左侧时，指向常量的指针 （底层 const，指向的数据无法修改）
- 当 const 位于 * 右侧时，常指针（顶层 const），指向常量的常指针



```cpp
int x = 0, y = 0;

// 指向常量的指针，底层 const
const int *p1 = &x;
// *p1 = 10;
p1 = &y;


// 常指针，顶层 const
int * const p2 = &x;
*p2 = 20;
// p2 = &y;

// 指向常量的常指针
const int * const p3 = &x;
// *p3 = 10;
// p3 = &y;
```



修饰类的非静态成员函数，保证类的非静态成员变量不被修改，非 const 成员函数与 const 成员函数可以构成重载。

```cpp
class Base {
public:
    void func() {
        x++;
    }
    void func() const {
        // x++;
        y++;
    }
private:
    int x;
    static int y;
};
```



# volatile

volatile 关键字表示 **变量是易变的，编译器不要进行优化，每次访问时，都要从地址中获取** 。

```cpp
int x = 5;
// 编译器优化：两者之间没有对 x 修改，直接取一次 x 分别赋值给 y、z
// 但 x 可能在其它线程，或以编译器无法差别的方式被修改了
int y = x;
int z = x;
```



volatile 修饰指针时，与 const 类似，有顶层与底层之分。

```cpp
volatile int* ptr;
int* volatile ptr;
volatile int* volatile ptr;
```



volatile 使用场景

- 和信号处理相关的场合
- 和内存映射相关的场合
- 非本地跳转相关的场合



volatile 不能解决多线程中的问题时说不能用来同步线程？？？？？？？



# constexpr

const 的语义是 “保护数据不被修改”，constexpr 的语义是 “编译时确定”



编译时要确定值的变量，需要声明为常量表达式

```cpp
// const 常量可以，但语义 constexpr 更合适
// const int N = 10;
constexpr int N = 10;
array<int, N> arr{};
```



编译时需要调用的函数，需要使用 constexpr 修饰

```cpp
constexpr int N = 5;
constexpr int func(int x, int y) {
    return x + y;
}

array<int, func(N, N)> arr{};
```



# virtual

修饰成员函数，使其成为一个虚函数/纯虚函数。

```cpp
class Base {
public:
    virtual void func() = 0;
    virtual ~Base() {}
};
```



修饰基类，表示虚继承。

```cpp
class Derive : virtual public Base {};
```



# override

修饰重写函数，使重写的意图更加明确。

```cpp
class Base {
public:
    virtual void func() {
        cout << "Base" << endl;
    }
};

class Derived : public Base {
public:
    void func() override {
        cout << "Derived" << endl;
    }
};
```



# final 

修饰类，禁止类被继承

```cpp
class Base final {};
// ERROR: Base is marked 'final'
class Derived : public Base {};
```



修饰虚函数，禁止重写虚函数

```cpp
class Base {
public:
    virtual void func() final {
        cout << "Base" << endl;
    }
};

class Derived : public Base {
public:
    // ERROR: Declaration of 'func' overrides a 'final' function
    void func() override {
        cout << "Derived" << endl;
    }
};
```



# explicit

修饰转换构造函数，防止隐式类型转换

```cpp
class Base {
public:
    explicit Base(int num) : x(num) {}
private:
    int x;
};

// ERROR: explicit constructor is not a candidate
// Base b1 = 5;
Base b2 = Base(5);
```



修饰类型转换运算符，防止隐式类型转换

```cpp
class Base {
public:
    explicit operator int() const {
        return 5;
    }
};

// ERROR: explicit conversion function is not a candidate
// int x1 = Base();
int x2 = static_cast<int>(Base());
```



虽然 explicit 修饰了类型转换运算符，但此时它被忽略了。

```cpp
class Base {
public:
    explicit Base(int num) : x(num) {}
    explicit operator bool() const {
        return x > 2;
    }
private:
    int x;
};

if(Base(1)) {
    cout << 111 << endl;
}
```



除非有充分理由，否则转换构造函数、类型转换运算符应该声明为 explicit



# mutable

修饰成员变量，表示其总是可变的，无论是在 const 还是非 const 成员函数里面。

```cpp
class Base {
public:
    void func() const {
        cout << ++x << endl;
    }
private:
    mutable int x;
};
```



# noexcept

`noexcept/noexcept(true)` 异常发生时，不会抛出异常，程序立马结束。析构函数默认是 `noexcept(true)` ，可以显示指定为 `noexcept(false)` ，但千万不要那么做！！！

```cpp
void func() noexcept {}
void func() noexcept(true) {}
```

```cpp
class X {
public:
    // default is noexcept(true)
    ~X() {
        throw(exception());
    }
};

int main() {
    try {
        for(int i = 0; i < 1; ++i) {
            X x;
        }
    } catch (...) {
        cout << 1111 << endl;
    }

    return 0;
}
```



`noexcept(false)` 异常发生时，抛出异常，由调用者决定如何处理异常。构造函数和普通函数在异常发生时，是默认抛出异常的

```cpp
void func() {}
void func() noexcept(false) {}
```

```cpp
void myFunction() noexcept(false) {
    cout << "Function called: " << endl;
    throw(exception());
}

class X {
public:
    // default is noexcept(false)
    X() {
        throw(exception());
    }
};

int main() {
    try {
        for(int i = 0; i < 1; ++i) {
            X x;
        }
        myFunction();
    } catch (...) {
        cout << 1111 << endl;
    }

    return 0;
}
```



# typedef

为类型定义别名

```cpp
typedef int* Ptr;
Ptr x = &t;

typedef void (*FPtr)();
void func() {
    cout << 111 << endl;
}
FPtr fp = func;
fp();
```



C 语言中为结构体取别名

```cpp
typedef struct _Base {
    int x;
} Base;
```



# using

导入命名空间，或命名空间中的声明，但需注意可能会导致命名冲突。

```cpp
using namespace std;

using std::cout;
```



为类型定义别名，相当于 typedef

```cpp
using Ptr = int*;

using FPtr = void (*)();
void func() {
    cout << 111 << endl;
}
FPtr fp = func;
fp();
```



派生类中导入被覆盖的基类成员

```cpp
class Base {
public:
    void func() {
        cout << "empty func" << endl;
    }
    virtual void func (int x) {
        cout << "Base::func(int)" << endl;
    }
};

class Derive : public Base {
public:
    using Base::func;
    void func(int x) override {
        cout << "Derive::func(int)" << endl;
    }
};

Derive d;
d.func();	
```



# namespace

为了解决命名冲突，C++ 引入了 namespace 限制标识符的作用域。



命名空间只能在全局作用域中定义，可以嵌套，内部可以包含变量、函数、类等。

```cpp
namespace outer {
    namespace inner {
        int x = 10;
    }
    int x = 20;
    void func() {
        cout << 111 << endl;
    }
    class Base {
    public:
        static int x;
    };
}

int outer::Base::x = 30;

cout << outer::inner::x << endl;
cout << outer::x << endl;
cout << outer::Base::x << endl;
outer::func();
```



匿名命名空间，只能在本文件内访问，效果等同于 static 。

```cpp
namespace {
    int x = 100;
}

static int x = 1000;

// Reference to 'x' is ambiguous
// cout << x << endl;
```



命名空间取别名

```cpp
namespace outer{
    int x = 100;
}

namespace inner = outer;
cout << outer::x << endl;
```



# typename

typename 的语义是 “后续表达式是一个类型，而非其他东西”



用来声明一个类型模板参数

```cpp
template<typename T>
T add(const T& a, const T& b) {
    return a + b;
}
```



说明，表达式为类型，而非成员函数或变量

```cpp
typedef typename _Base::iterator iterator;
```



# new

使用 new 可以在堆上创建一个对象

```cpp 
auto *p = new vector<int>(5);
delete p;
```

使用 new[] 在堆上批量创建对象

```cpp
auto *p = new vector<int>[2] {{1, 3}, {2, 4}};
delete []p;
```



# delete

当不需要编译器生成默认构造函数、默认拷贝构造函数、默认赋值运算符时，可以将其声明为 delete

```cpp
class Base {
public:
    Base() = delete;
};

// ERROR
// Base b1;
```

delelte 释放 new 创建的单个对象

```cpp
auto *p = new vector<int>(5);
delete p;
```

delete[] 释放 new[] 批量创建的对象

```cpp
auto *p = new vector<int>[2] {{1, 3}, {2, 4}};
delete []p;
```



# default

当自定义了构造函数时，编译器不会生成默认构造函数，default 表示使用编译器生成的默认版本。

```cpp
class Base {
public:
    Base() = default;
    explicit Base(int num) : x(num) {}
private:
    int x;
};

Base b1{};
Base b2(5);
```



# extern

使用 extern 可以引用其它编译单元中定义的变量

```cpp
// a.cpp
int x = 10;

// b.cpp
// int x = 20; ERROR 重定义
extern int x;
```



引用 const 全局变量：在一个文件中声明一个 const 变量，并且没有使用 extern 关键字进行声明，那么该 const 变量的作用域将被限制在当前文件内，其他文件无法访问或使用该 const 变量。

```cpp
// a.cpp
// const int x = 10; ERROR
extern const int x = 10;
// 不同文件定义相同的 const 全局变量不会导致重定义
const int y = 20;

// b.cpp
extern const int x;
const int y = 20;
```



在头文件中 extern 引入，cpp 文件中定义，需要使用的地方包含头文件

```cpp
// a.h
extern int x;
extern const int y;

// a.cpp
int x = 10;
extern const int y = 20;

// b.cpp
#include "a.h"
cout << x << endl;
cout << y << endl;
```



C++ 中使用 extern "C"  表示使用 C 的方式进行编译

```cpp
extern "C" int add(int x, int y);
extern "C" int multi(int x, int y) {
    return x * y;
}

extern "C" {
    void func1();
    void func2();
}
```





# friend

友元的语义是 “允许其它人访问自己的私有成员”



声明一个全局函数为友元

```cpp
class Base {
public:
    friend ostream& operator<<(ostream&, const Base&);
    explicit Base(int num) : x(num) {}
private:
    int x;
};

ostream& operator<<(ostream &os, const Base &b) {
    os << "Base: " << b.x;
    return os;
}

Base b(5);
cout << b << endl;
```



声明一个类为友元

```cpp
class Friend;

class Base {
public:
    friend Friend;
    explicit Base(int num) : x(num) {}
private:
    int x;
};

class Friend {
public:
    explicit Friend(int num) : b(num) {}
    void func() const {
        cout << "Friend func: " << b.x << endl;
    }
private:
    Base b;
};

Friend f(5);
f.func();
```



声明一个类的成员函数为友元

```cpp
class Base;

class Friend {
public:
    void func1(const Base& b) const;
    void func2(const Base& b) const;
};

class Base {
public:
    friend void Friend::func1(const Base &b) const;
    explicit Base(int num) : x(num) {}
private:
    int x;
};

void Friend::func1(const Base &b) const {
    cout << "Friend func: " << b.x << endl;
}

void Friend::func2(const Base &b) const {
    //cout << "Friend func: " << b.x << endl;
}
```



# auto

auto 简化变量的定义，让编译器去推导变量的类型。当不太好确定返回值类型时，也可以让编译器去推导。

```cpp
for(vector<int>::iterator it = a.begin(); it != a.end(); ++it);
for(auto it = a.begin(); it != a.end(); ++it);
```



auto 会忽略掉顶层 const，保留底层 const;

```cpp
const int* const p1 = &t;
auto p2 = p1; // auto 被推导为 const int*
```



auto 会忽略引用，保留指针（如上）

```cpp
vector<int> a(10);
auto b1 = a; // auto 被推导为 vector<int>, b1 是 a 的副本
auto &b2 = a; // auto 被推导为 vector<int>, b2 是 a 的引用
```



# decltype

auto 的语义是 “简化变量声明，类型让编译器去推导”，decltype 的语义是 “获取变量/表达式的类型”



decltype(x) 获取变量的类型：cv 属性、指针、引用都会被保留。

```cpp
const int* const p1 = &t;
// Ptr 被推导为 const int* const
using Ptr = decltype(p1);
```

```cpp
int w = 10;
int &x1 = w;
const int &x2 = 20;
int &&y1 = 30;
const int &&y2 = 40;
// decltype(w) 推导结果为 int
// decltype(x1) 推导结果为 int&
// decltype(x2) 推导结果为 int const&
// decltype(y1) 推导结果为 int&&
// decltype(y2) 推导结果为 int const&&
```



decltype((x)) 不同于 decltype(x)，用于获取表达式的类型：

- x 为 lvalue 时，推导为 type&
- x 为 pvalue 时，推导为 type
- x 为 xvalue 时，推导为 type&&

```cpp
int w = 10;
// decltype((w)) 推导结果为 int&
// decltype((10)) 推导结果为 int
// decltype((move(w))) 推导结果为 int&&
```



decltype(auto) 声明变量，以下两者等价

```cpp
decltype(*pos) element = *pos;
decltype(auto) element = *pos;
```





