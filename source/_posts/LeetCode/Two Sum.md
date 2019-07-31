---
title: Two Sum
date: 2019-05-14 20:35:17
categories: LeetCode(Swift)
---

### Two Sum

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

示例:

> 给定 nums = [2, 7, 11, 15], target = 9  
因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]

暴力解法可以使用两个循环嵌套,但是时间复杂度过高
可以牺牲一定空间,使用查表的方式提升效率

解法一:

两遍遍历
```bash
func twoSum( nums : [Int], target: Int) -> [Int] {
        
        var dict = [Int : Int]();
        for (index, item) in nums.enumerated() {
            dict.updateValue(index, forKey: item);
        }

        var result = [Int]();
        for (index, item) in nums.enumerated() {
            if(dict.keys.contains(target - item) && (dict[target - item] != index)) {
                result.append(index);
                result.append(dict[target - item]!);
                break;
            }
        }
        
        return result;
    }
```

解法二:

一遍遍历
```bash
func twoSum( nums : [Int], target: Int) -> [Int] {
        
        var dict = [Int : Int]();
        var result = [Int]();
        for (index, item) in nums.enumerated() {
            if(dict.keys.contains(target - item) && (dict[target - item] != index)) {
                result.append(index);
                result.append(dict[target - item]!);
                break;
            }
            dict.updateValue(index, forKey: item);
        }
        
        return result;
    }
```