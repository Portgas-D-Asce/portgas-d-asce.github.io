---
layout: page
tags: 设计模式
image: /assets/image/27.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 抽象工厂模式（Abstract Factory）
categories: [计算机, 设计模式]
---

# 概述

## 动机

在软件系统中，经常面临着“一系列相互依赖的对象”的创建工 作;同时，由于需求的变化，往往存在更多系列对象的创建工作。

如何应对这种变化?如何绕过常规的对象创建方法(new)，提供一 种“封装机制”来避免客户程序和这种“多系列具体对象创建工作” 的紧耦合?



## 定义

提供一个接口，让该接口负责创建一系列“相关或者相互依赖的对象”，无需指定它们具体的类



## 类图

![/assets/content/5.png](/assets/content/5.png)



# 实现

```cpp
#include <iostream>
using namespace std;

// 产品 A 工厂模式
class IProductA {
public:
    virtual ~IProductA() = default;
    virtual void func() = 0;
};

class ProductA1 : public IProductA {
public:
    void func() override {
        cout << "ProductA1" << endl;
    }
};

class ProductA2 : public IProductA {
public:
    void func() override {
        cout << "ProductA2" << endl;
    }
};

class IFactoryA {
public:
    virtual ~IFactoryA() = default;
    virtual IProductA* create() = 0;
};

class FactoryProductA1 : public IFactoryA {
public:
    IProductA* create() override {
        return new ProductA1();
    }
};

class FactoryProductA2 : public IFactoryA {
public:
    IProductA* create() override {
        return new ProductA2();
    }
};


// 产品 B 工厂模式
class IProductB {
public:
    virtual ~IProductB() = default;
    virtual void func() = 0;
};

class ProductB1 : public IProductB {
public:
    void func() override {
        cout << "ProductB1" << endl;
    }
};

class ProductB2 : public IProductB {
public:
    void func() override {
        cout << "ProductB2" << endl;
    }
};

class IFactoryB {
public:
    virtual ~IFactoryB() = default;
    virtual IProductB* create() = 0;
};

class FactoryProductB1 : public IFactoryB {
public:
    IProductB* create() override {
        return new ProductB1();
    }
};

class FactoryProductB2 : public IFactoryB {
public:
    IProductB* create() override {
        return new ProductB2();
    }
};

// 生产一个产品 a 和一个产品 b
void test(IFactoryA* fa, IFactoryB* fb) {
    IProductA* a = fa->create();
    a->func();

    IProductB* b = fb->create();
    b->func();
}


int main() {
    IFactoryA* fa = new FactoryProductA1();
    IFactoryB* fb = new FactoryProductB2();
    test(fa, fb);
    return 0;
}
```

涉及两个产品，分别用工厂模式创建出它们，没有问题。但有时会有一种情况：

- FactoryProductA1 和 FactoryProductB1 是 Factory1 的生产线
- FactoryProductA2 和 FactoryProductB2 是 Factory2 的生产线
- A、B 两个产品需要搭配使用，但最好是同一家公司的产品，也就是不允许 ProductA1 和 ProductB2 （ProductA2 和 ProductB1）搭配使用
- 此时该怎么办？？？？



```cpp
#include <iostream>
using namespace std;

class IProductA {
public:
    virtual ~IProductA() = default;
    virtual void func() = 0;
};

class ProductA1 : public IProductA {
public:
    void func() override {
        cout << "ProductA1" << endl;
    }
};

class ProductA2 : public IProductA {
public:
    void func() override {
        cout << "ProductA2" << endl;
    }
};

class IProductB {
public:
    virtual ~IProductB() = default;
    virtual void func() = 0;
};

class ProductB1 : public IProductB {
public:
    void func() override {
        cout << "ProductB1" << endl;
    }
};

class ProductB2 : public IProductB {
public:
    void func() override {
        cout << "ProductB2" << endl;
    }
};

// 抽象工厂
class IFactory {
public:
    virtual ~IFactory() = default;
    virtual IProductA* create_a() = 0;
    virtual IProductB* create_b() = 0;
};

class Factory1 : public IFactory {
public:
    IProductA* create_a() override {
        return new ProductA1();
    }

    IProductB* create_b() override {
        return new ProductB1();
    }
};

class Factory2 : public IFactory {
public:
    IProductA* create_a() override {
        return new ProductA2();
    }

    IProductB* create_b() override {
        return new ProductB2();
    }
};

// 生产一个产品 a 和一个产品 b
void test(IFactory* f) {
    IProductA* a = f->create_a();
    a->func();

    IProductB* b = f->create_b();
    b->func();
}


int main() {
    IFactory* f = new Factory2();
    test(f);
    return 0;
}
```

此时 test 创建出的搭配只有 ProductA1、ProductB1 和 ProductA2、ProductB2 两种情况。



工厂模式和抽象工厂模式的区别：

- 工厂模式用于创建单个产品（苹果手机、小米手机）
- 抽象工厂模式用于创建某个系列的所有产品（苹果系列：iphone、mac、iwatch、ipods）



# 应用

如果没有应对“多系列对象构建”的需求变化，则没有必要使用 Abstract Factory模式，这时候使用简单的工厂完全可以。

“系列对象”指的是在某一特定系列下的对象之间有相互依赖、 或作用的关系。不同系列的对象之间不能相互依赖。

Abstract Factory模式主要在于应对“新系列”的需求变动。其缺 点在于难以应对“新对象”的需求变动
