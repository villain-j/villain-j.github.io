---
title: "LeetCode Single Number"
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-13T23:29:00+09:00

classes: wide
toc: false
---

## Problem

Given a **non-empty** array of integers, every element appears *twice* except for one. Find that single one.

**Note:**

Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

**Example 1:**

```
Input: [2,2,1]
Output: 1
```

**Example 2:**

```
Input: [4,1,2,1,2]
Output: 4
```



## Solution

오... 상상도 못한 방법이었다 알고리즘 문제 열심히 풀어야지ㅠㅠ

```c#
public class Solution {
    public int SingleNumber(int[] nums) {
        int result = nums[0];
        for (int i=1; i<nums.Length; i++)
        {
            result = result ^ nums[i];
        }
        return result;
    }
}
```

![image-20200413233015445](C:\Users\hyeyo\AppData\Roaming\Typora\typora-user-images\image-20200413233015445.png)