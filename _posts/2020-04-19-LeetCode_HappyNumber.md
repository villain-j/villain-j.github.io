---
title: LeetCode Happy Number
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-19T14:09:00+09:00

classes: wide
toc: false
---



## Problem

Write an algorithm to determine if a number `n` is "happy".

A happy number is a number defined by the following process: Starting with any positive integer, replace the number by the sum of the squares of its digits, and repeat the process until the number equals 1 (where it will stay), or it **loops endlessly in a cycle** which does not include 1. Those numbers for which this process **ends in 1** are happy numbers.

Return True if `n` is a happy number, and False if not.

**Example:** 

```
Input: 19
Output: true
Explanation: 
12 + 92 = 82
82 + 22 = 68
62 + 82 = 100
12 + 02 + 02 = 1
```

 

## Solution

한 자리 숫자의 경우 1과 7을 제외하면 Happy Number가 될 수 있는 숫자가 없어서 한 자리 숫자일 때는 1과 7이 아니면 false를 반환한다. 한 자리 숫자가 아닌 경우 각 자리의 숫자의 제곱 합을 구해 HashSet에 넣고 이미 있으면 무한 루프를 도는 숫자이므로 false를 반환한다.



처음에 시간 효율이 너무 처참하게 나와서 몇 번 고쳤었는데 같은 코드 제출했는데도 실행 시간이 다르게 나오는 걸 보니까 음...

```c#
public class Solution {
    private int GetSquareSum(int n) {
        string str = n.ToString();
        int result = 0;
        for (int i=0; i<str.Length; i++) {
            int tmp = str[i]-'0';
            result += tmp*tmp;
        }
        return result;
    }
    
    public bool IsHappy(int n) {
        HashSet<int> sums = new HashSet<int>();
        sums.Add(n);
        
        while (true) {
            if (n>9) {
                int result = GetSquareSum(n);
                if (sums.Contains(result)) return false;
                sums.Add(result);
                n = result;
            } else {
                if (n==1 || n==7) return true;
                else return false;
            }
        }
    }
}
```


