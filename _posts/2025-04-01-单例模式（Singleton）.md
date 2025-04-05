---
layout: page
tags: 设计模式
image: /assets/image/28.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 单例模式（Singleton）
---

# 概述

## 动机

在软件系统中，存在一种特殊的类，必须保证它们只存在一个实例，才能保证它们的逻辑正确性、良好的效率。



如何绕过常规构造器，提供一种机制来保证一个类只有一个实例？



这是类设计者的责任，而不是使用者的责任。



## 定义

保证一个类仅有一个实例，并提供一个该实例的全局访问点。



## 类图

![/assets/content/3.png](/assets/content/3.png)

## 实现

实现方式有两种，懒汉模式/饿汉模式



当要求类仅有一个实例化对象时，就需要使用单例模式。

- 构造函数、拷贝构造、拷贝赋值、析构函数：私有，防止存在多个对象，单例被意外释放
- 单例：静态，私有，指针形式。
- `get_instance`：静态公有，可以返回单例指针，单例引用。



# 饿汉模式

在一开始就进行单例的实例化：也是线程安全的，但当有很多单例，且比较大时，会浪费内存空间。

```cpp
class Singleton {
public:
    static Singleton& get_instance() {
        return *_instance;
    }
private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton&) = default;
    Singleton& operator=(const Singleton&) = default;
private:
    static Singleton* _instance;
};

Singleton* Singleton::_instance = new Singleton();
```



最简单模式！！！

```cpp
class Singleton {
public:
    static Singleton& instance() {
        static Singleton singleton;
        return singleton;
    }
    // ...
};
```



# 懒汉模式

只有用到的时候才进行实例化

## Baseline

```cpp
class Singleton {
public:
    static Singleton& get_instance() {
        if(!_instance) {
            _instance = new Singleton();
        }
        return *_instance;
    }
private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton&) = default;
    Singleton& operator=(const Singleton&) = default;
private:
    static Singleton* _instance;
};

Singleton* Singleton::_instance = nullptr;
```

显然以上实现，存在两点问题：

- 内存泄露问题：虽然是特殊的内存泄露，程序结束时系统会自动回收，但是泄漏就是泄漏，必须回收
- 不是线程安全的



## 无脑加锁

虽然保证了线程安全，但存在效率问题。

```cpp
class Singleton {
public:
    static Singleton& get_instance() {
        {
            static mutex mtx;
            lock_guard<mutex> lg(mtx);
            if(!_instance) {
                _instance = new Singleton();
            }
        }
        return *_instance;
    }
private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton&) = default;
    Singleton& operator=(const Singleton&) = default;
private:
    static Singleton* _instance;
};

Singleton* Singleton::_instance = nullptr;
```



## Double Check

主要是为了避免每次调用都加锁的开销，看上去是线程安全的，但实际上不是：内存读写 reorder 会导致线程不安全：

- 线程 1 取得互斥锁：new 表达式先申请内存，然后将起始地址赋值给 `_instance` ，但是还没有调用构造函数
- 线程 2 检查 `_instance` 已经初始化，直接返回，但此时 `_instance` 对象还没有构造好呢

```cpp
class Singleton {
public:
    static Singleton& get_instance() {
        if(!_instance) {
            static mutex mtx;
            lock_guard<mutex> lg(mtx);
            if(!_instance) {
                _instance = new Singleton();
            }
        }
        return *_instance;
    }
private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton&) = default;
    Singleton& operator=(const Singleton&) = default;
private:
    static Singleton* _instance;
};

Singleton* Singleton::_instance = nullptr;
```



## once init

```cpp
class Singleton {
public:
    static Singleton& get_instance() {
        static once_flag flag;
        call_once(flag, [&](){
            _instance = new Singleton();
        });
        return *_instance;
    }
private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton&) = default;
    Singleton& operator=(const Singleton&) = default;
private:
    static Singleton* _instance;
};

Singleton* Singleton::_instance = nullptr;
```



## 内存泄露

通过智能指针，来自动释放申请的内存空间。

```cpp
class Singleton {
public:
    friend class default_delete<Singleton>;
    static Singleton& get_instance() {
        if(!_instance) {
            _instance = unique_ptr<Singleton>(new Singleton());
        }
        return *_instance;
    }
private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton&) = default;
    Singleton& operator=(const Singleton&) = default;
private:
    static unique_ptr<Singleton> _instance;
};
unique_ptr<Singleton> Singleton::_instance = nullptr;
```



## final

```cpp
class Singleton {
public:
    friend class default_delete<Singleton>;
    static Singleton& get_instance() {
        static once_flag flag;
        call_once(flag, [&]() {
            _instance = unique_ptr<Singleton>(new Singleton());
        });
        return *_instance;
    }
private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton&) = default;
    Singleton& operator=(const Singleton&) = default;
private:
    static unique_ptr<Singleton> _instance;
};

unique_ptr<Singleton> Singleton::_instance = nullptr;
```



