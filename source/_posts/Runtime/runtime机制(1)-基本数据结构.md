---
title: runtime机制(1)-基本数据结构
p: /Runtime/runtime机制(1)-基本数据结构
date: 2020-08-31 20:15:14
tags:
categories: Runtime
---

### 什么是runtime

OC是动态语言, 与静态语言不同, 静态语言的各种数据结构在编译期已经决定了, 不能被修改, 而动态语言可以在程序的运行期, 动态的修改一个类的结构, 如修改方法实现、绑定实例变量等.

> OC作为动态语言, 会想办法将编译期做的事推迟到运行期来做, 所以, 仅有编译器是不够的, 还需要一个运行时系统(runtime system), 它是OC运行框架的基石.

要想了解runtime，就要先了解runtime中定义的各种数据结构, 先从最基础的objc_object和objc_class开始。

### 缘起:NSObject

OC中，基本上所有的类的基类，都是NSObject。因此要深入了解OC中的类的结构，就要从NSObject这个类说起。

XCode中NSObject的实现:

```
@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}
```

NSObject仅有一个实例变量Class isa：
```
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;
```

Class实质上是指向objc_class的指针, 在runtime源码的objc-runtime-new.h中，可以看到objc_class在OC 2.0中的定义:
```
struct objc_class : objc_object {
    // Class ISA;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags

    class_rw_t *data() { 
        return bits.data();
    }
    void setData(class_rw_t *newData) {
        bits.setData(newData);
    }
	。。。。。。
}

objc_class继承自objc_object, 所以在runtime中, class也被看做一种对象, class中, 有三个数据:
* Class superclass: 同样是Class类型，表明当前类的父类。
* cache_t cache: cache用于优化方法调用，其对应的数据结构如是：
```
struct cache_t {
    struct bucket_t *_buckets;
    mask_t _mask;
    mask_t _occupied;
    
	// 省略其余方法
	。。。   
}

typedef uintptr_t cache_key_t;

struct bucket_t {
private:
    cache_key_t _key;
    IMP _imp;

public:
    inline cache_key_t key() const { return _key; }
    inline IMP imp() const { return (IMP)_imp; }
    inline void setKey(cache_key_t newKey) { _key = newKey; }
    inline void setImp(IMP newImp) { _imp = newImp; }

    void set(cache_key_t newKey, IMP newImp);
};
```