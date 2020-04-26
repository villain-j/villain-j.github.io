---
title: LeetCode Construct Binary Search Tree from Preorder Traversal
categories:
  - leetcode
author_profile: true
read_time: true
tags:
  - leetcode
  - csharp
last_modified_at: 2020-04-20T23:04:00+09:00

classes: wide
toc: false
---



## Problem

Return the root node of a binary **search** tree that matches the given `preorder` traversal.

*(Recall that a binary search tree is a binary tree where for every node, any descendant of `node.left` has a value `<` `node.val`, and any descendant of `node.right` has a value `>` `node.val`. Also recall that a preorder traversal displays the value of the `node` first, then traverses `node.left`, then traverses `node.right`.)*

 

**Example 1:**

```
Input: [8,5,1,7,10,12]
Output: [8,5,10,1,7,null,12]
```

 

**Note:** 

1. `1 <= preorder.length <= 100`
2. The values of `preorder` are distinct.





## Solution

Binary Tree니까 preorder traversal 순서대로 binary tree에 그대로 넣어줘서 반환하면 된다.

```c#
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     public int val;
 *     public TreeNode left;
 *     public TreeNode right;
 *     public TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    private void InsertTreeNode(TreeNode root, int val) {
        if (val < root.val) {
            if (root.left == null) {
                TreeNode newNode = new TreeNode(val);
                root.left = newNode;
                return;
            }
            else {
                InsertTreeNode(root.left, val);
                return;
            }
        }
        else {
            if (root.right == null) {
                TreeNode newNode = new TreeNode(val);
                root.right = newNode;
                return;
            }
            else {
                InsertTreeNode(root.right, val);
                return;
            }
        }
    }
    
    public TreeNode BstFromPreorder(int[] preorder) {
        TreeNode root = new TreeNode(preorder[0]);
        for (int i=1; i<preorder.Length; i++) {
            InsertTreeNode(root, preorder[i]);
        }
        return root;
    }
}
```


