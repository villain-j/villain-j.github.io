---
title: "LeetCode Valid Parenthesis String"
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-19T14:31:00+09:00

classes: wide
toc: false
---



## Problem

Given a string containing only three types of characters: '(', ')' and '*', write a function to check whether this string is valid. We define the validity of a string by these rules:

1. Any left parenthesis `'('` must have a corresponding right parenthesis `')'`.
2. Any right parenthesis `')'` must have a corresponding left parenthesis `'('`.
3. Left parenthesis `'('` must go before the corresponding right parenthesis `')'`.
4. `'*'` could be treated as a single right parenthesis `')'` or a single left parenthesis `'('` or an empty string.
5. An empty string is also valid.



**Example 1:**

```
Input: "()"
Output: True
```



**Example 2:**

```
Input: "(*)"
Output: True
```



**Example 3:**

```
Input: "(*))"
Output: True
```



**Note:**

1. The string size will be in the range [1, 100].



## Solution

여기저기서 항상 꾸준히 다시 나오는 Parenthesis 문제.

`lo`가 지금 열려있는 괄호의 최소 개수이고 `hi`가 지금 열려있는 괄호의 최대 개수이다. 지금 글자가 `'('`이면 열린 괄호의 최소 개수는 무조건 1개 증가하므로 `lo`에 1을 더하고, 아닌 경우 괄호가 닫힐 수 있으므로 `lo`에 1을 뺀다. 반대로 지금 글자가 `')'`라면 괄호가 무조건 1개 닫히므로 `hi`에서 1을 뺀다. 아닌 경우에는 `hi`에 1을 더한다.

이렇게 했을 때 열린 괄호의 최대 개수(`hi`)가 음수가 되면 괄호가 이상하게 된 것이므로 false를 반환하고, 열린 괄호의 최소 개수는 음수가 될 수는 없으므로 음수인 경우 0으로 바꿔준다. 이렇게 문자열을 모두 훑었을 때 열린 괄호의 최소 개수가 0이어야 모든 괄호가 제대로 닫힌 것이다.

```c#
public class Solution {
    public bool CheckValidString(string s) {
        int lo = 0;
        int hi = 0;
        for (int i=0; i<s.Length; i++) {
            if (s[i]=='(') lo++;
            else lo--;
            if (s[i]!=')') hi++;
            else hi--;
            if (hi<0) return lo==0;
            lo = lo>0 ? lo:0;
        }
        return lo==0;
    }
}
```