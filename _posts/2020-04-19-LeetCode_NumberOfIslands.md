---
title: "LeetCode Number Of Islands"
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-19T14:17:00+09:00

classes: wide
toc: false
---



## Problem

Given a 2d grid map of `'1'`s (land) and `'0'`s (water), count the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.

**Example 1:**

```
Input:
11110
11010
11000
00000

Output: 1
```

**Example 2:**

```
Input:
11000
11000
00100
00011

Output: 3
```



## Solution

BFS/DFS로 풀면 된다.

```c#
public class Solution {
    private bool in_range(int x_len, int y_len, int x, int y) {
        return x>=0 && x<x_len && y>=0 && y<y_len;
    }
    
    private bool bfs_search(char[][] grid, char[,] searched, int x_len, int y_len, int x, int y) {
        if (searched[x,y] == '1' || grid[x][y] == '0') return false;
        
        searched[x,y] = '1';
        
        int[,] adjacent = new int[4,2] { {x-1, y}, {x+1, y}, {x, y-1}, {x, y+1} };
        
        for (int i=0; i<4; i++) {
            if (in_range(x_len, y_len, adjacent[i,0], adjacent[i,1])) {
                bfs_search(grid, searched, x_len, y_len, adjacent[i,0], adjacent[i,1]);
            }
        }
        
        return true;
    }
    
    public int NumIslands(char[][] grid) {
        int x_len = grid.Length;
        if (x_len == 0) return 0;
        int y_len = grid[0].Length;
        char[,] search_grid = new char[x_len, y_len];
        for (int i=0; i<x_len; i++) {
            for (int j=0; j<y_len; j++) {
                search_grid[i,j] = '0';
            }
        }
        
        int count = 0;
        for (int i=0; i<x_len; i++) {
            for (int j=0; j<y_len; j++) {
                if (bfs_search(grid, search_grid, x_len, y_len, i, j)) {
                    count++;
                }
            }
        }
        return count;
    }
}
```

