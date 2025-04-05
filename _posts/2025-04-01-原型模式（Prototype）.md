---
layout: page
tags: 设计模式
image: /assets/image/32.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 原型模式（Prototype）
categories: [计算机, 设计模式]
---

# 概述

## 动机

经常面临 “某些结构复杂的对象” 的创建工作，由于需求的变化，这些对象经常面临着剧烈的变化，但是它们却拥有比较稳定一致的接口。



如何应对这种变化？如何想 “客户程序” 隔离这些易变对象，从而使得 “依赖这些易变对象的客户程序” 不随着需求改变而改变？



## 定义

使用原型实例指定创建对象的种类，然后通过拷贝这些原型来创建新的对象。



就是创建对象太麻烦了，搞出一个原型，下次要用的时候直接拷贝。

写作业不好写，抄作业好抄啊。



## 类图

![/assets/content/6.png](/assets/content/6.png)





# 实现

```cpp
class IProduct {
public:
    virtual ~IProduct() = default;
    virtual IProduct* clone() = 0;
    virtual void func() = 0;
};

class Product1 : public IProduct {
public:
    IProduct* clone() override {
        return new Product1(*this);
    }

    void func() override {
        cout << "Product1" << endl;
    }
};

class Product2 : public IProduct {
public:
    IProduct* clone() override {
        return new Product2(*this);
    }

    void func() override {
        cout << "Product2" << endl;
    }
};

int main() {
    IProduct* prototype = new Product1();

    IProduct* copy = prototype->clone();
    copy->func();
    return 0;
}
```





# 应用

prototype 模式同样用于隔离对象的使用者和具体类型（易变类）之间的耦合，它同样要求这些 “易变类” 拥有 “稳定的接口”。



Prototype 模式对于 “如何创建易变类的尸体对象” 采用 “原型克隆” 的方法来做，它是的我们可以分厂灵活地动态创建 “拥有某些稳定接口” 的新对象：所需工作仅仅是注册一个新类的对象（即原型），然后在任何需要的地方 clone



prototype 模式中的 clone 方法可以利用某些框架中的序列化来实现深拷贝。
