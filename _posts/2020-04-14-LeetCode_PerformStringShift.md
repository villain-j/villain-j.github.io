---
title: "LeetCode Perform String Shifts"
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-14T23:47:00+09:00

classes: wide
toc: false
---

## Problem

You are given a string `s` containing lowercase English letters, and a matrix `shift`, where `shift[i] = [direction, amount]`:

- `direction` can be `0` (for left shift) or `1` (for right shift). 
- `amount` is the amount by which string `s` is to be shifted.
- A left shift by 1 means remove the first character of `s` and append it to the end.
- Similarly, a right shift by 1 means remove the last character of `s` and add it to the beginning.

Return the final string after all operations.

 

**Example 1:**

```
Input: s = "abc", shift = [[0,1],[1,2]]
Output: "cab"
Explanation: 
[0,1] means shift to left by 1. "abc" -> "bca"
[1,2] means shift to right by 2. "bca" -> "cab"
```

**Example 2:**

```
Input: s = "abcdefg", shift = [[1,1],[1,1],[0,2],[1,3]]
Output: "efgabcd"
Explanation:  
[1,1] means shift to right by 1. "abcdefg" -> "gabcdef"
[1,1] means shift to right by 1. "gabcdef" -> "fgabcde"
[0,2] means shift to left by 2. "fgabcde" -> "abcdefg"
[1,3] means shift to right by 3. "abcdefg" -> "efgabcd"
```

 

**Constraints:**

- `1 <= s.length <= 100`
- `s` only contains lower case English letters.
- `1 <= shift.length <= 100`
- `shift[i].length == 2`
- `0 <= shift[i][0] <= 1`
- `0 <= shift[i][1] <= 100`



## Solution

**첫 번째 풀이**

그냥 C# `System.Collections.Generic`에서 제공하는 `LinkedList\<char>`에 한 글자씩 넣고 하라는 대로 했다... 생각해보니 shift 다 합쳐서 결론적으로 몇 글자를 얼마나 옮겨야 하는지 세는 방법도 있겠구나.

```c#
public class Solution {
    public string StringShift(string s, int[][] shift) {       
        LinkedList<char> str = new LinkedList<char>();
        for (int i=0; i<s.Length; i++){
            str.AddLast((char)s[i]);
        }
        int index = 0;
        while (index < shift.Length)
        {
            if (shift[index][0] == 0)
            {
                while (shift[index][1] != 0) {
                    char c = str.First.Value;
                    str.RemoveFirst();
                    shift[index][1]--;
                    str.AddLast(c);
                }
            }
            else
            {
                while (shift[index][1] != 0) {
                    char c = str.Last.Value;
                    str.RemoveLast();
                    str.AddFirst(c);
                    shift[index][1]--;
                }
            }
            index++;
        }
        
        string result = "";
        LinkedListNode<char> node = str.First;
        while (node != str.Last) {
            result += node.Value;
            node = node.Next;
        }
        result += node.Value;
        return result;
    }
}
```

![](../assets/images/LeetCode_PerformStringShifts_01.png)



**두 번째 풀이**

첫 번째 풀이 쓰다가 더 나은 방법 찾아서 고침. 처음에 total을 총 길이로 나눠서 나머지를 구했어야 했는데 그 부분을 빼먹어서 Substring 인자가 마이너스가 나왔었나 그랬다. 퇴근하고 문제풀기 쉽지 않아...

```c#
public class Solution {
    public string StringShift(string s, int[][] shift) {       
        LinkedList<char> str = new LinkedList<char>();
        for (int i=0; i<s.Length; i++){
            str.AddLast((char)s[i]);
        }
        
        int total = 0;
        for (int i=0; i<shift.Length; i++) {
            if (shift[i][0] == 0) total -= shift[i][1];
            else total += shift[i][1];
        }
        total = total % s.Length;
        
        if (total == 0) return s;
        else if (total < 0) {
            // 앞에서 total 길이만큼 가져와서 뒤에 붙임
            string str1 = s.Substring(-total);
            string str2 = s.Substring(0, -total);
            return str1+str2;
        }
        else {
            //뒤에서 total 길이만큼 가져와서 앞에 붙임
            string str1 = s.Substring(s.Length-total);
            string str2 = s.Substring(0, s.Length-total);
            return str1+str2;
        }
    }
}
```

![](../assets/images/LeetCode_PerformStringShifts_02.png)