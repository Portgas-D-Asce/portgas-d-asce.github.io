---
layout: page
tags: 设计模式
image: /assets/image/24.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 装饰器模式（Decorator）
---

# 概述

## 定义

动态地给一个对象增加一些额外的职责。



# 实现

## 派生方式

```cpp
#include <iostream>
using namespace std;

// 组件接口
class IComponent {
public:
    virtual void func() = 0;
    virtual ~IComponent() = default;
};

// 组件 1 实现
class Component1 : public IComponent {
public:
    void func() override {
        cout << "Component1" << endl;
    }
};

// 组件 2 实现
class Component2 : public IComponent {
public:
    void func() override {
        cout << "Component2" << endl;
    }
};

// 装饰器 1 修饰组件 1
class Decorator1Comp1 : public Component1 {
public:
    void func() override {
        // 装饰
        cout << "Decorator1" << endl;
        Component1::func();
    }
};

// 装饰器 1 修饰组件 2
class Decorator1Comp2 : public Component2 {
public:
    void func() override {
        // 装饰
        cout << "Decorator1" << endl;
        Component2::func();
    }
};

// 装饰器 2 修饰组件 1
class Decorator2Comp1 : public Component1 {
public:
    void func() override {
        // 装饰
        cout << "Decorator2" << endl;
        Component1::func();
    }
};

// 装饰器 2 修饰组件 2
class Decorator2Comp2 : public Component2 {
public:
    void func() override {
        // 装饰
        cout << "Decorator2" << endl;
        Component2::func();
    }
};

// 装饰器 1 修饰 “装饰器 2 修饰组件 1”
class Decorator1Decorator2Comp1 : public Decorator2Comp1 {
public:
    void func() override {
        // 装饰
        cout << "Decorator1" << endl;
        Decorator2Comp1::func();
    }
};

// 装饰器 1 修饰 “装饰器 2 修饰组件 2”
class Decorator1Decorator2Comp2 : public Decorator2Comp2 {
public:
    void func() override {
        // 装饰
        cout << "Decorator1" << endl;
        Decorator2Comp2::func();
    }
};


int main() {
    // 静态装配
    Decorator1Decorator2Comp1 d12c1;
    d12c1.func();
    return 0;
}
```

可以实现 “装饰” 任务，但是不够优雅：

- 同一修饰器修饰组件时，会导致重复代码

- 当 `IComponent` 实现过多，修饰场景变得复杂时，将会变为地狱！



## 装饰器模式

```cpp
#include <iostream>
using namespace std;

// 组件接口
class IComponent {
public:
    virtual void func() = 0;
    virtual ~IComponent() = default;
};

// 组件 1 实现
class Component1 : public IComponent {
public:
    void func() override {
        cout << "Component1" << endl;
    }
};

// 组件 2 实现
class Component2 : public IComponent {
public:
    void func() override {
        cout << "Component2" << endl;
    }
};

// 装饰器接口，之所以继承组件接口：装饰器只起装饰作用
class IDecorator : public IComponent {
protected:
    // 要被装饰的组件
    IComponent* _comp;
public:
    explicit IDecorator(IComponent* comp) : _comp(comp) {}
};

class Decorator1 : public IDecorator {
public:
    explicit Decorator1(IComponent* comp) : IDecorator(comp) {}

    void func() override {
        // 装饰行为
        cout << "Decorator1" << endl;
        // 真正行为
        _comp->func();
    }
};

class Decorator2 : public IDecorator {
public:
    explicit Decorator2(IComponent* comp) : IDecorator(comp) {}

    void func() override {
        // 装饰行为
        cout << "Decorator2" << endl;
        // 真正行为
        _comp->func();
    }
};


int main() {
    // 动态装配
    Component1 comp1;
    Decorator1 dec1(&comp1);
    dec1.func();

    cout << endl;

    // 神奇的是，可以多次装饰
    //Decorator1 dec11(&dec1);
    Decorator2 dec2(&dec1);
    dec2.func();
    return 0;
}
```

实现优雅，要说有什么缺点的话，多态调用效率低。

