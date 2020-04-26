---
title: "LeetCode Search In Rotated Array"
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-20T00:48:00+09:00

classes: wide
toc: false
---



## Problem

Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.

(i.e., `[0,1,2,4,5,6,7]` might become `[4,5,6,7,0,1,2]`).

You are given a target value to search. If found in the array return its index, otherwise return `-1`.

You may assume no duplicate exists in the array.

Your algorithm's runtime complexity must be in the order of *O*(log *n*).

**Example 1:**

```
Input: nums = [4,5,6,7,0,1,2], target = 0
Output: 4
```

**Example 2:**

```
Input: nums = [4,5,6,7,0,1,2], target = 3
Output: -1
```



## Solution

Binary Search 문제는 몇 번을 풀어도 한 번만에 깔끔하게 해결을 못하는 것 같다ㅠㅠ 30일 챌린지 끝나면 Binary Search 문제만 주구장창 풀어봐야지...

`O(log n)` 으로 해결해야 되기 때문에 우선 얼마만큼 돌아갔는지 찾는 것도 Binary search로 찾아야 한다. 그렇게 시작점을 찾고 나서 Binary Search를 한다. 나는 일단 `left, right`를 회전되지 않은 배열에서의 인덱스로 생각하고 (안그러면 binary search 구현하는 게 너무 생각하기 귀찮아진다...) binary search 하면서 배열에 접근할 때만 GetSkewedIndex로 회전된 배열 상의 인덱스로 바꿔서 생각했다.

```c#
public class Solution {
    private int GetSkewedIndex(int index, int skewed, int size) {
        index += skewed;
        if (index >= size) index -= size;
        return index;
    }
    
    private int SkewedBinarySearch(int[] nums, int target, int left, int right, int skewed) {
        if (left == right) {
            int index = GetSkewedIndex(left, skewed, nums.Length);
            if (nums[index] == target) return index;
            else return -1;
        }
        else if (right-left == 1) {
            int index = GetSkewedIndex(left, skewed, nums.Length);
            if (nums[index] == target) return index;
            index = GetSkewedIndex(right, skewed, nums.Length);
            if (nums[index] == target) return index;
            else return -1;
        }
        
        int mid = GetSkewedIndex((left+right)/2, skewed, nums.Length);
        if (nums[mid] == target) {
            return mid;
        }
        else if (nums[mid] < target) {
            left = (left+right)/2;
        }
        else {
            right = (left+right)/2;
        }
        return SkewedBinarySearch(nums, target, left, right, skewed);
    }
    
    public int Search(int[] nums, int target) {
        if (nums.Length == 0) return -1;
        if (nums.Length == 1) {
            if (target == nums[0]) return 0;
            else return -1;
        }
        if (nums.Length == 2) {
            if (target == nums[0]) return 0;
            else if (target == nums[1]) return 1;
            else return -1;
        }
        
        var left = 0;
        var right = nums.Length - 1;
        var mid = nums.Length / 2;
        var start = -1;
        // if nums[left] > nums[mid], start is in left side
        // else if nums[right] < nums[mid], start is in right side
        while (start == -1) {
            if (nums[mid] < nums[left]) {
                right = mid;
            }
            else if (nums[mid] > nums[right]) {
                left = mid;
            }
            else start = 0;
            mid = (left+right)/2;
            if (left == mid) {
                if (nums[left] > nums[right]) start = right;
                else start = left;
            }
        }
        
        return SkewedBinarySearch(nums, target, 0, nums.Length-1, start);
    }
}
```