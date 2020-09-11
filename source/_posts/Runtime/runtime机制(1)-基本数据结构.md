---
title: runtime机制(1)-基本数据结构
p: /Runtime/runtime机制(1)-基本数据结构
date: 2020-08-31 20:15:14
tags:
categories: Runtime
---

## 什么是runtime

OC是动态语言, 与静态语言不同, 静态语言的各种数据结构在编译期已经决定了, 不能被修改, 而动态语言可以在程序的运行期, 动态的修改一个类的结构, 如修改方法实现、绑定实例变量等.

> OC作为动态语言, 会想办法将编译期做的事推迟到运行期来做, 所以, 仅有编译器是不够的, 还需要一个运行时系统(runtime system), 它是OC运行框架的基石.

要想了解runtime，就要先了解runtime中定义的各种数据结构, 先从最基础的objc_object和objc_class开始。

## 缘起:NSObject

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
## objc_clas
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
```
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
这里我们第一次遇到uintptr_t类型（_key） 。在runtime中，uintptr_t定义为
```
#ifndef _UINTPTR_T
#define _UINTPTR_T
typedef unsigned long		uintptr_t;
#endif /* _UINTPTR_T */
```
可以理解为void *.

cache的核心是一个类型为bucket_t的指针, 指向一个cache_key_t和IMP的缓存节点.

runtime方法调用的流程: 当要调用一个方法时, 先不去Class的方法列表中查找, 而是先去找cache_t cache. 
> 理论上一个方法被调用过之后, 再次被调用的概率很大, 所以当系统调用过一个方法后, 会将其实现IMP和key存到cache中

* class_data_bits_t bits: Class的核心, 本质上是一个可以被Mask的指针, 不同的Mask, 可以取出不同的值.
```
struct class_data_bits_t {

    // Values are the FAST_ flags above.
    uintptr_t bits;
 
	public:
    class_rw_t* data() {
        return (class_rw_t *)(bits & FAST_DATA_MASK);
    }
    void setData(class_rw_t *newData)
    {
        assert(!data()  ||  (newData->flags & (RW_REALIZING | RW_FUTURE)));
        // Set during realization or construction only. No locking needed.
        // Use a store-release fence because there may be concurrent
        // readers of data and data's contents.
        uintptr_t newBits = (bits & ~FAST_DATA_MASK) | (uintptr_t)newData;
        atomic_thread_fence(memory_order_release);
        bits = newBits;
    }
    。。。
```

class_data_bits_t中仅含有一个成员uintptr_t, 可以理解为"复合指针".
> 复合指针: 不仅包含了指针, 还包含了Class的各种异或flag来说明Class的属性, 把这些信息复合在一起, 仅用一个指针来表示

当需要取出这些信息时, 需要用对应的以***FAST_***前缀开头的flag掩码对bits做按位与操作

例如，我们需要取出Classs的核心信息class_rw_t, 则需要调用方法：
```
class_rw_t* data() {
        return (class_rw_t *)(bits & FAST_DATA_MASK);
    }
```
该方法返回一个class_rw_t*，需要对bits进行FAST_DATA_MASK的与操作。

Class的核心结构class_rw_t:
```
struct class_rw_t {
    // Be warned that Symbolication knows the layout of this structure.
    uint32_t flags;
    uint32_t version;

    const class_ro_t *ro;         // 类不可修改的原始核心

    // 下面三个array，method,property, protocol，可以被runtime 扩展，如Category
    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;

    // 和继承相关的东西
    Class firstSubclass;
    Class nextSiblingClass;

    // Class对应的 符号名称
    char *demangledName;
	
	// 以下方法省略
	...
}

struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
#ifdef \__LP64\__
    uint32_t reserved;
#endif

    const uint8_t * ivarLayout;
    
    const char * name;
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;

    const uint8_t * weakIvarLayout;
    property_list_t *baseProperties;

    method_list_t *baseMethods() const {
        return baseMethodList;
    }
};
```
可以看到，在class_ro_t 中包含了类的名称，以及method_list_t， protocol_list_t， ivar_list_t， property_list_t 这些类的基本信息。 在class_ro_t 的信息是***不可修改和扩展***的。
在更外一层 class_rw_t 中，有三个数组method_array_t, property_array_t, protocol_array_t:
```
struct class_rw_t {

	...
    const class_ro_t *ro;         // 类不可修改的原始核心

    // 下面三个array，method,property, protocol，可以被runtime 扩展，如Category
    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;
	...
}
```
这三个数组是可以被runtime动态扩展的。

类抽象结构:
> objc_class
>> class_data_bits_t
>>> class_rw_t(通过FAST_DATA_MASK获取)
>>>> class_ro_t(类核心const信息)

## realizeClass
在objc_class的data()方法最初返回的是const class_ro_t * 类型，也就是类的基本信息。因为在调用realizeClass方法前，Category定义的各种方法，属性还没有附加到class上，因此只能够返回类的基本信息。

而当我们调用realizeClass时，会在函数内部将Category中定义的各种扩展附加到class上，同时改写data()的返回值为class_rw_t *类型，核心代码如下:
```
	const class_ro_t *ro;
    class_rw_t *rw;
	ro = (const class_ro_t *)cls->data();
    if (ro->flags & RO_FUTURE) {
        // This was a future class. rw data is already allocated.
        rw = cls->data();
        ro = cls->data()->ro;
        cls->changeInfo(RW_REALIZED|RW_REALIZING, RW_FUTURE);
    } else {
        // Normal class. Allocate writeable class data.
        rw = (class_rw_t *)calloc(sizeof(class_rw_t), 1);
        rw->ro = ro;
        rw->flags = RW_REALIZED|RW_REALIZING;
        cls->setData(rw);
    }
```
所以在class没有调用realizeClass之前，不是真正完整的类。

## objc_object
OC的底层实现是runtime，在runtime这一层，对象被定义为objc_object 结构体，类被定义为了objc_class 结构体。而objc_class继承于objc_object，因此，类可以看做是一类特殊的对象。
objc_object定义:
```
struct objc_object {
private:
    isa_t isa;

public:

    // ISA() assumes this is NOT a tagged pointer object
    Class ISA();

    // getIsa() allows this to be a tagged pointer object
    Class getIsa();

	// 省略其余方法
	...
}
```
objc_object的定义很简单，仅包含一个isa_t 类型。
```
union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;
	
	// 省略其余
	。。。
}
```
isa_t 是一个联合，可以表示Class cls或uintptr_t bits类型。