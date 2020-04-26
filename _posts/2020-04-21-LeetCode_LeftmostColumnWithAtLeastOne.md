---
title: "LeetCode Leftmost Column with at least One"
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-21T23:06:00+09:00

classes: wide
toc: false
---



## Problem

*(This problem is an **interactive problem**.)*

A binary matrix means that all elements are `0` or `1`. For each **individual** row of the matrix, this row is sorted in non-decreasing order.

Given a row-sorted binary matrix binaryMatrix, return leftmost column index(0-indexed) with at least a `1` in it. If such index doesn't exist, return `-1`.

**You can't access the Binary Matrix directly.** You may only access the matrix using a `BinaryMatrix` interface:

- `BinaryMatrix.get(x, y)` returns the element of the matrix at index `(x, y)` (0-indexed).
- `BinaryMatrix.dimensions()` returns a list of 2 elements `[n, m]`, which means the matrix is `n * m`.

Submissions making more than `1000` calls to `BinaryMatrix.get` will be judged *Wrong Answer*. Also, any solutions that attempt to circumvent the judge will result in disqualification.

For custom testing purposes you're given the binary matrix `mat` as input in the following four examples. You will not have access the binary matrix directly.

 

**Example 1:**

**![img](https://assets.leetcode.com/uploads/2019/10/25/untitled-diagram-5.jpg)**

```
Input: mat = [[0,0],[1,1]]
Output: 0
```

**Example 2:**

**![img](https://assets.leetcode.com/uploads/2019/10/25/untitled-diagram-4.jpg)**

```
Input: mat = [[0,0],[0,1]]
Output: 1
```

**Example 3:**

**![img](https://assets.leetcode.com/uploads/2019/10/25/untitled-diagram-3.jpg)**

```
Input: mat = [[0,0],[0,0]]
Output: -1
```

**Example 4:**

**![img](https://assets.leetcode.com/uploads/2019/10/25/untitled-diagram-6.jpg)**

```
Input: mat = [[0,0,0,1],[0,0,1,1],[0,1,1,1]]
Output: 1
```

 

**Constraints:**

- `1 <= mat.length, mat[i].length <= 100`

- `mat[i][j]` is either `0` or `1`.

- `mat[i]` is sorted in a non-decreasing way.

  

## Solution

뭐지 블로그 들킨 거 같은 이 기분... Binary Search 문제 좀 더 풀어야지라고 쓰니까 Binary Search만 나오는 이 기분ㅋㅋㅋ

각 row에 대해 binary search를 하면서 제일 왼쪽에 있는 1을 찾는다. 찾으면서 이미 1을 찾은 줄에서 나온 column 열보다 뒷부분을 검색하는 줄은 다 스킵한다. 최대 100 x 100이므로 모든 줄을 끝까지 검사한다 해도
$$
100*log_2(100)=663.3856
$$
번을 넘지 않는다.

```c#
/**
 * // This is BinaryMatrix's API interface.
 * // You should not implement it, or speculate about its implementation
 * class BinaryMatrix {
 *     public int Get(int x, int y) {}
 *     public IList<int> Dimensions() {}
 * }
 */

class Solution {
    public int LeftMostColumnWithOne(BinaryMatrix binaryMatrix) {
        var dimensions = binaryMatrix.Dimensions();
        var maxLen = dimensions[1];
        int[] left = new int[dimensions[0]];
        int[] right = new int[dimensions[0]];
        
        for (int i=0; i<dimensions[0]; ++i) {
            right[i] = maxLen-1;
        }
        
        var minRight = maxLen;
        var leftmost = maxLen;
        var notLeftMost = 0;
        
        while (notLeftMost != dimensions[0]) {
            for (int i=0; i<dimensions[0]; ++i) {
                // Console.WriteLine("Examine " + i.ToString() + ": " + left[i].ToString() + ", " + right[i].ToString());
                if (left[i] == -1) continue;
                
                if (left[i] > minRight || left[i] > leftmost || left[i]==right[i]) {
                    // Console.WriteLine("Skip");
                    left[i] = -1;
                    notLeftMost++;
                    continue;
                }
                
                if (left[i]+1==right[i]) {
                    if (binaryMatrix.Get(i, left[i]) == 1) {
                        if (leftmost > left[i]) leftmost = left[i];
                    }
                    else if (binaryMatrix.Get(i, right[i]) == 1) {
                        if (leftmost > right[i]) leftmost = right[i];
                    }
                    left[i] = -1;
                    notLeftMost++;
                    continue;
                }
                
                var mid = (left[i]+right[i])/2;
                if (binaryMatrix.Get(i, mid) == 0) {
                    // Console.WriteLine("discovered 0");
                    left[i] = mid;
                }
                else {
                    // Console.WriteLine("discovered 1");
                    right[i] = mid;
                    if (mid < leftmost) leftmost = mid;
                    if (right[i] < minRight) minRight = right[i];
                }
            }
            // Console.WriteLine("-----------------");
            // Console.WriteLine("leftmost: " + leftmost.ToString() + ", skipCount: " + notLeftMost);
            // Console.WriteLine("-----------------");
        }
        
        if (leftmost == maxLen) return -1;
        return leftmost;
    }
}
```