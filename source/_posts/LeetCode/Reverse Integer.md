---
title: Reverse Integer
categories: LeetCode(Swift版)
date: 2019-06-28 15:13:50
---


### Reverse Integer

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

示例 1:

>输入: 123
输出: 321
 
示例 2:

>输入: -123
输出: -321

示例 3:

>输入: 120
输出: 21
注意:

假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−2^31,  2^31 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。

解法:
将整数 % 10,取到的值放入数组中,再将整数 / 10, 得到的将是减少一位后的值,循环重复这个过程,则将每位数字都存入了数组中,若整数位负数,则取反,若超出数值范围,则返回0,代码如下:

```bash
func reverseInteger(num : Int) -> Int {
        
        var temp = abs(num);
        var i = 0;
        var array = [Int]();
        while (temp != 0) {
            i = temp % 10;
            temp = temp / 10;
            array.append(i);
        }
        
        var res = 0;
        for (index, item) in array.enumerated() {
            let t : Double = pow(10.0, Double(array.count - index - 1));
            res = res + item * Int(t);
        }
        
        if (num < 0) {
            res = -res;
        }
        
        if (res < Int32.min || res > Int32.max) {
            res = 0;
        }
        
        return res;
    }
```