---
title: "LeetCode Last Stone Weight"
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-12T23:16:00+09:00

classes: wide
toc: false
---

## Problem

We have a collection of stones, each stone has a positive integer weight.

Each turn, we choose the two **heaviest** stones and smash them together. Suppose the stones have weights `x` and `y` with `x <= y`. The result of this smash is:

- If `x == y`, both stones are totally destroyed;
- If `x != y`, the stone of weight `x` is totally destroyed, and the stone of weight `y` has new weight `y-x`.

At the end, there is at most 1 stone left. Return the weight of this stone (or 0 if there are no stones left.)

1. `1 <= stones.length <= 30`
2. `1 <= stones[i] <= 1000`





## Solution

생각 많이 안 하고 그냥 문제 그대로 풀었다. 일단 처음에 정렬해놓고 제일 큰 돌의 무게 두 개를 비교해서 같으면 둘 다 부시고 돌 개수 두 개 줄이고 다르면 더 큰 애 무게를 남겨놓고 뒤에서 앞으로 가면서 정렬된 자리에 갈 때까지 swap해줬다. 지금 풀이 쓰다가 보니까 swap할 때 for 쓰지 말고 while써서 break걸걸... 그래서 시간이 제일 잘 나오진 않았던 거겠구나. 그래도 뭐... 총 배열 길이가 30이하니까ㅎㅎㅎㅎㅎ

```c#
public class Solution {
    public int LastStoneWeight(int[] stones) {
        int stoneCount = stones.Length;
        Array.Sort(stones);
        while (stoneCount>1)
        {
            if (stones[stoneCount-1] == stones[stoneCount-2])
            {
                stones[stoneCount-1] = 0;
                stones[stoneCount-2] = 0;
                stoneCount -= 2;
            }
            else
            {
                stones[stoneCount-2] = stones[stoneCount-1] - stones[stoneCount-2];
                stones[stoneCount-1] = 0;
                stoneCount -= 1;
                for (int i=stoneCount-1; i>0; i--)
                {
                    if (stones[i] < stones[i-1])
                    {
                        int tmp = stones[i-1];
                        stones[i-1] = stones[i];
                        stones[i] = tmp;
                    }
                }
            }
        }
        return stones[0];
    }
}
```

![image-20200413231319948](C:\Users\hyeyo\AppData\Roaming\Typora\typora-user-images\image-20200413231319948.png)





 흠 뭐지...? else break 추가하니까 갑자기 실행시간 90ms가 넘었다. 미스테리군...