---
title: "LeetCode Minimum Path Sum"
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-19T14:13:00+09:00

classes: wide
toc: false
---



## Problem

Given a *m* x *n* grid filled with non-negative numbers, find a path from top left to bottom right which *minimizes* the sum of all numbers along its path.

**Note:** You can only move either down or right at any point in time.

**Example:**

```
Input:
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
Output: 7
Explanation: Because the path 1→3→1→1→1 minimizes the sum.
```

 

## Solution

그냥 간단한 DP... 아래나 오른쪽으로만 움직이면 되서 생각 많이 안 하고 간단하게 풀었다.

```c#
public class Solution {
    public int MinPathSum(int[][] grid) {
        int x_len = grid.Length;
        int y_len = grid[0].Length;
        for (int i=1; i<x_len; i++) {
            grid[i][0] = grid[i-1][0] + grid[i][0];
        }
        for (int j=1; j<y_len; j++) {
            grid[0][j] = grid[0][j-1] + grid[0][j];
        }
        
        for (int i=1; i<x_len; i++) {
            for (int j=1; j<y_len; j++) {
                grid[i][j] += grid[i-1][j]<grid[i][j-1] ? grid[i-1][j] : grid[i][j-1];
            }
        }
        return grid[x_len-1][y_len-1];
    }
}
```


