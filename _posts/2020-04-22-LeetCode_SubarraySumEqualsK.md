---
title: "LeetCode Subarray Sum Equals K"
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-22T23:47:00+09:00

classes: wide
toc: false
---



## Problem

Given an array of integers and an integer **k**, you need to find the total number of continuous subarrays whose sum equals to **k**.

**Example 1:**

```
Input:nums = [1,1,1], k = 2
Output: 2
```



**Note:**

1. The length of the array is in range [1, 20,000].
2. The range of numbers in the array is [-1000, 1000] and the range of the integer **k** is [-1e7, 1e7].



## Solution

```c#
public class Solution {
    public int SubarraySum(int[] nums, int k) {
        var tot_count = 0;
        var sum = 0;
        Dictionary<int, int> map = new Dictionary<int, int>();
        map.Add(0, 1);
        
        for (int i=0; i<nums.Length; i++) {
            sum += nums[i];
            var count = 0;
            if (map.TryGetValue(sum-k, out count)) {
                tot_count += count;
            }
            if (map.TryGetValue(sum, out count)) {
                map[sum] = count+1;
            }
            else {
                map.Add(sum, 1);
            }
        }
        
        return tot_count;
    }
}
```