---
title: LeetCode Product of Array Except Self
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-19T14:19:00+09:00

classes: wide
toc: false
---



## Problem

Given an array `nums` of *n* integers where *n* > 1,  return an array `output` such that `output[i]` is equal to the product of all the elements of `nums` except `nums[i]`.

**Example:**

```
Input:  [1,2,3,4]
Output: [24,12,8,6]
```

**Constraint:** It's guaranteed that the product of the elements of any prefix or suffix of the array (including the whole array) fits in a 32 bit integer.

**Note:** Please solve it **without division** and in O(*n*).

**Follow up:**
Could you solve it with constant space complexity? (The output array **does not** count as extra space for the purpose of space complexity analysis.)



## Solution

나누기 없이 풀라니... 방법이 생각 안 나서 일단 나누기로 풀고 Solution 보고 풀었다ㅠㅠ

이거 풀었던 날 집에 와서 너무 피곤했어서 `O(1)` 로 푸는 방법은 적당히 보고 넘어갔었는데 지금 보니까 좀 알 것 같다.



### 첫 번째 풀이

`left[i]`는 `nums[i]` 왼쪽에 있는 원소들의 곱이고 `right[i]`는 `nums[i]` 오른쪽에 있는 원소들의 곱이다.

```c#
public class Solution {
    public int[] ProductExceptSelf(int[] nums) {
        int N = nums.Length;
        int[] left = new int[N];
        int[] right = new int[N];
        
        int product = 1;
        left[0] = 1;
        
        for (int i=1; i<N; i++) {
            product *= nums[i-1];
            left[i] = product;
        }
        
        product = 1;
        right[N-1] = 1;
        for (int i=N-2; i>=0; i--) {
            product *= nums[i+1];
            right[i] = product;
        }
        
        nums[0] = right[0];
        nums[N-1] = left[N-1];
        for (int i=1; i<N-1; i++) {
            nums[i] = left[i] * right[i];
        }
        
        return nums;
    }
}
```



## 두 번째 풀이

첫 번째 풀이랑 거의 비슷한데 `answer[]`에 기존의 `left[]`를 저장하고, `right[]`를 쓰는 대신 `answer[]`를 끝에서부터 채우면서 `R`에다가 오른쪽 원소들의 곱을 저장하면서 답을 구하면 된다.

```c#
public class Solution {
    public int[] ProductExceptSelf(int[] nums) {
        int N = nums.Length;
        int[] answer = new int[N];
        
        answer[0] = 1;
        
        for (int i=1; i<N; i++) {
            answer[i] = answer[i-1] * nums[i-1];
        }
        
        int R = 1;
        for (int i=N-1; i>=0; i--) {
            answer[i] = answer[i] * R;
            R *= nums[i];
        }
        
        return answer;
    }
}
```

