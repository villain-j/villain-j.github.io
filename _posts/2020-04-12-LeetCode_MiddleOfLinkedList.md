---
title: "LeetCode Middle of the Linked List"
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-12T23:35:00+09:00

classes: wide
toc: false
---

## Problem

Given a non-empty, singly linked list with head node `head`, return a middle node of linked list.

If there are two middle nodes, return the second middle node.

**Example 1:**

```
Input: [1,2,3,4,5]
Output: Node 3 from this list (Serialization: [3,4,5])
The returned node has value 3.  (The judge's serialization of this node is [3,4,5]).
Note that we returned a ListNode object ans, such that:
ans.val = 3, ans.next.val = 4, ans.next.next.val = 5, and ans.next.next.next = NULL.
```

**Example 2:**

```
Input: [1,2,3,4,5,6]
Output: Node 4 from this list (Serialization: [4,5,6])
Since the list has two middle nodes with values 3 and 4, we return the second one.
```

**Note:**

- The number of nodes in the given list will be between `1` and `100`.



## Solution

```c#
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     public int val;
 *     public ListNode next;
 *     public ListNode(int x) { val = x; }
 * }
 */
public class Solution {
    public ListNode MiddleNode(ListNode head) {
        int listLength = 1;
        ListNode midNode = head;
        ListNode node = head;
        while (node.next != null)
        {
            listLength++;
            if (listLength % 2 == 0)
            {
                midNode = midNode.next;
            }
            node = node.next;
        }
        return midNode;
    }
}
```

![image-20200412233849124](C:\Users\hyeyo\AppData\Roaming\Typora\typora-user-images\image-20200412233849124.png)

큐ㅠㅠㅠ 이정도 위치면 더 잘 풀 수 있는 방법이 있다는 건데... 끝까지 안 가고 중간 노드가 언제인지 알 수는 없을 것 같은데 중간 과정에 낭비가 있나?