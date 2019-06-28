---
title: Swift基本语法
date: 2019-06-28 11:09:13
tags:
categories: Swift Summary
---

1.变量和常量可以使用 print（swift 2 将 print 替换了 println） 函数来输出。

在字符串中可以使用括号与反斜线来插入变量，如下实例：
```bash
import Cocoa

var name = "菜鸟教程"
var site = "http://www.runoob.com"

print("\(name)的官网地址为：\(site)")
以上程序执行结果为：

菜鸟教程的官网地址为：http://www.runoob.com
```

2.当声明一个可选类型的时候，要确保用括号给 ? 操作符一个合适的范围。例如，声明可选整数数组，应该写成 (Int[])? 写成 Int[]? 会报错。

3.如果一个可选类型的实例包含一个值，你可以用后缀操作符 ！来访问这个值，如下所示：

```bash
optionalInteger = 42
optionalInteger! // 42
```
注意：
使用!来获取一个不存在的可选值会导致运行时错误。使用!来强制解析值之前，一定要确定可选包含一个非nil的值。

4.你可以在声明可选变量时使用感叹号（!）替换问号（?）。这样可选变量在使用时就不需要再加一个感叹号（!）来获取值，它会自动解析。

5.使用可选绑定（optional binding）来判断可选类型是否包含值，如果包含就把值赋给一个临时常量或者变量。可选绑定可以用在if和while语句中来对可选类型的值进行判断并把值赋给一个常量或者变量。

让我们来看下一个简单的可选绑定实例：
```bash
import Cocoa

var myString:String?

myString = "Hello, Swift!"

if let yourString = myString {
   print("你的字符串值为 - \(yourString)")
}else{
   print("你的字符串没有值")
}
以上程序执行结果为：

你的字符串值为 - Hello, Swift!
```

6.整型字面量可以是一个十进制，二进制，八进制或十六进制常量。 二进制前缀为 0b，八进制前缀为 0o，十六进制前缀为 0x，十进制没有前缀：

以下为一些整型字面量的实例：
```bash
let decimalInteger = 17           // 17 - 十进制表示
let binaryInteger = 0b10001       // 17 - 二进制表示
let octalInteger = 0o21           // 17 - 八进制表示
let hexadecimalInteger = 0x11     // 17 - 十六进制表示
```

7.除非特别指定，浮点型字面量的默认推导类型为 Swift 标准库类型中的 Double，表示64位浮点数。

8.注意：swift3 中已经取消了++、--。

9.以下为区间运算的简单实例：

```bash
import Cocoa

print("闭区间运算符:")
for index in 1...5 {
    print("\(index) * 5 = \(index * 5)")
}

print("半开区间运算符:")
for index in 1..<5 {
    print("\(index) * 5 = \(index * 5)")
}
```

10.注意：在大多数语言中，switch 语句块中，case 要紧跟 break，否则 case 之后的语句会顺序运行，而在 Swift 语言中，默认是不会执行下去的，switch 也会终止。如果你想在 Swift 中让 case 之后的语句会按顺序继续运行，则需要使用 fallthrough 语句。

11.你可以使用 == 来比较两个字符串是否相等

12.Swift 中不能创建空的 Character（字符） 类型变量或常量

13.字符串连接字符
以下实例演示了使用 String 的 append() 方法来实现字符串连接字符：
```bash
import Cocoa

var varA:String = "Hello "
let varB:Character = "G"

varA.append( varB )

print("varC  =  \(varA)")
```

14.你可以使用 append() 方法或者赋值运算符 += 在数组末尾添加元素，如下所示，我们初始化一个数组，并向其添加元素

15.如果我们同时需要每个数据项的值和索引值，可以使用 String 的 enumerate() 方法来进行数组遍历

16.我们可以使用加法操作符（+）来合并两种已存在的相同类型数组。新数组的数据类型会从两个数组的数据类型中推断出来

17.我们可以使用 updateValue(forKey:) 增加或更新字典的内容。如果 key 不存在，则添加值，如果存在则修改 key 对应的值。updateValue(_:forKey:)方法返回Optional值.

18.我们可以使用 removeValueForKey() 方法来移除字典 key-value 对。如果 key 存在该方法返回移除的值，如果不存在返回 nil 