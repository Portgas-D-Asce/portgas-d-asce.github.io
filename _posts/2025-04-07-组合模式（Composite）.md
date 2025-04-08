---
layout: page
image: /assets/image/51.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 组合模式（Composite）
categories: [计算机, 设计模式]
---

# 概述

老板下达一条指令，不需要传递给公司每个人，只需要传递给各个部门负责人，部门负责人再传递给子部门负责人 ...... 直到传递给每个人

## 动机

在软件的某些情况下，客户代码过多地依赖于对象容器复杂的内部实现结构，对象容器内部实现结构（而非抽象接口）的变化将引起客户代码的频繁变化，带来了代码的维护性、扩展性等弊端



如何将 “客户代码与复杂的对象容器结构” 解耦？让对象容器自己来实现自身的复杂结构，从而使得客户代码就像处理简单对象一样来处理复杂的对象容器？



## 定义

将对象组合成树形结构以表示 “部分-整体” 的层次结构。Composite 使得用户对单个对象和组合对象的使用具有一致性（稳定）



## 类图

![/assets/content/13.png](/assets/content/13.png)



# 实现

```cpp
#include <iostream>
#include <string>
#include <set>
using namespace std;

class IComposite {
public:
    virtual ~IComposite() = default;
    virtual void process() = 0;
};

class Composite : public IComposite {
private:
    set<IComposite*> _children;
    string _name;
public:
    explicit Composite(const string& name) : _name(name) {}

    void add(IComposite* child) {
        _children.insert(child);
    }

    void rm(IComposite* child) {
        _children.erase(child);
    }

    void process() override {
        cout << "Composite Node: " << _name << endl;
        for(auto child : _children) {
            child->process();
        }
    }

};

class Leaf : public IComposite {
private:
    string _name;
public:
    explicit Leaf(const string& name) : _name(name) {}

    void process() override {
        cout << "Leaf Node: " << _name << endl;
    }
};

int main() {
    Composite root("root");
    Composite node11("node11");
    Composite node12("node12");
    Leaf leaf13("leaf13");
    Leaf leaf21("leaf21");
    Leaf leaf22("leaf22");

    root.add(&node11);
    root.add(&node12);
    root.add(&leaf13);
    node11.add(&leaf21);
    node11.add(&leaf22);

    root.process();

    cout << endl;

    node11.process();

    cout << endl;

    leaf21.process();

    return 0;
}
```



# 应用

