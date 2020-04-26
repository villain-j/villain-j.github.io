---
title: "LeetCode Maximum Subarray"
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-19T16:07:00+09:00

classes: wide
toc: false

---

## Problem

Given an integer array `nums`, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

**Example:**

```
Input: [-2,1,-3,4,-1,2,1,-5,4],
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
```

**Follow up:**

If you have figured out the O(*n*) solution, try coding another solution using the divide and conquer approach, which is more subtle.



## Solution

분명히 더 빠른 풀이랑 내 풀이랑 알고리즘 상으로는 별 차이가 없어서 왜 이렇게 시간 차이가 나지 의문이었는데 `int sum, int max, int i`를 다 `var sum, var max, var i`로 바꾸니까 빨라졌다. 무슨 차이지...? C#의 세계는 참 어렵군ㅠㅠ C# 형식을 다시 한 번 봐야 할 것 같다.

```c#
public class Solution {
    public int MaxSubArray(int[] nums) {
        if(nums.Length == 0) return 0;
        
        var sum = 0;
        var max = int.MinValue;
        for(var i = 0; i < nums.Length; i++){
            sum += nums[i];
            
            if(sum > max) max = sum;
            if(sum < 0) sum = 0;
        }
        
        return max;
    }
}
```
