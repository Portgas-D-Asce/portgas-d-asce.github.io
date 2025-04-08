---
layout: page
image: /assets/image/45.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 职责链模式（Chain of Responsibility）
categories: [计算机, 设计模式]
---

# 概述

## 动机

在软件构建过程中，一个请求可能被多个对象处理，但是每个请求在运行时只能有一个接收者，如果显示指定，将必不可少地带来请求发送者与接收者的紧耦合



如何使请求的发送者不需要指定具体的接收者？让请求的接收者自己在运行时决定来处理请求，从而使两者解耦。



## 定义

使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止。



将请求处理者连成一条链，让请求沿着这条链去处理，直到有一个对象处理它为止。



## 类图

![/assets/content/12.png](/assets/content/12.png)



# 实现

```cpp
#include <iostream>
#include <string>
using namespace std;

class IHandler {
private:
    IHandler* _next;
public:
    explicit IHandler(IHandler* next) : _next(next) {}
    virtual ~IHandler() = default;

    virtual void deal(const string& event) = 0;
    
    virtual bool pred(const string& event) = 0;

    void process(const string& event) {
        if(pred(event)) {
            deal(event);
        } else {
            _next->process(event);
        }
    }
};

class Handler1 : public IHandler {
public:
    explicit Handler1(IHandler* next) : IHandler(next) {}

    bool pred(const string& event) override {
        return event.empty();
    }

    void deal(const string& event) override {
        cout << "event is empty" << endl;
    }
};

class Handler2 : public IHandler {
public:
    explicit Handler2(IHandler* next) : IHandler(next) {}

    bool pred(const string& event) override {
        return event.size() <= 5;
    }

    void deal(const string& event) override {
        cout << event << ": small event" << endl;
    }
};

class Handler3 : public IHandler {
public:
    explicit Handler3() : IHandler(nullptr) {}

    bool pred(const string& event) override {
        return true;
    }

    void deal(const string& event) override {
        cout << event << ": big event" << endl;
    }
};

int main() {
    string event1;
    string event2 = "abc";
    string event3 = "abcdefgh";

    Handler3 handler3{};
    Handler2 handler2(&handler3);
    Handler1 handler1(&handler2);

    handler1.process(event1);
    handler1.process(event2);
    handler1.process(event3);

    return 0;
}
```





# 应用
