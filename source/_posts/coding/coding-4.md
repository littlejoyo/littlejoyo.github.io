---
title: Coding:重建二叉树
date: 2020-03-22 22:56:51
categories:
  - [算法题]
tag:
    - 算法
---

- 《剑指offer》算法题目4

- 关键词：树、二叉树

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

# 1.题目

输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。
假设输入的前序遍历和中序遍历的结果中都不含重复的数字。
例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

二叉树的定义：

```java
 public class TreeNode {
      int val;
      TreeNode left;
      TreeNode right;
      TreeNode(int x) { 
          val = x; 
    }
 }
```

# 2.解题思路

## 2.1 思路分析

根据`中序遍历`和`前序遍历`可以确定二叉树，具体过程为：

1.根据前序序列第一个结点确定根结点
2.根据根结点在中序序列中的位置分割出左右两个子序列
3.对左子树和右子树分别递归使用同样的方法继续分解

例如：
前序序列{1,2,4,7,3,5,6,8} = pre
中序序列{4,7,2,1,5,3,8,6} = in

## 2.2 分解过程

1.根据当前前序序列的第一个结点确定根结点，为 1

2.找到 1 在中序遍历序列中的位置，为 in[3]

3.切割左右子树，则 in[3] 前面的为左子树， in[3] 后面的为右子树

4.所以切割后的左子树前序序列为：{2,4,7}，切割后的左子树中序序列为：{4,7,2}；

5.切割后的右子树前序序列为：{3,5,6,8}，切割后的右子树中序序列为：{5,3,8,6}

6.对分割后的子树分别使用同样的方法分解，符合递归的思路。

## 2.3 实现代码 

```java
import java.util.Arrays;
public class Solution {
    public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
        if (pre.length == 0 || in.length == 0) {
            return null;
        }
        TreeNode root = new TreeNode(pre[0]);
        // 在中序中找到前序的根
        for (int i = 0; i < in.length; i++) {
            if (in[i] == pre[0]) {
                // 左子树，注意 copyOfRange 函数，左闭右开
                root.left = reConstructBinaryTree(Arrays.copyOfRange(pre, 1, i + 1), Arrays.copyOfRange(in, 0, i));
                // 右子树，注意 copyOfRange 函数，左闭右开
                root.right = reConstructBinaryTree(Arrays.copyOfRange(pre, i + 1, pre.length), Arrays.copyOfRange(in, i + 1, in.length));
                break;
            }
        }
        return root;
    }
}
```

注意：

- `copyOfRange`函数的范围是：左闭右开,[a,b)

- 区分好子树的分布范围，然后通过递归的思想来构建二叉树

## 2.4 复杂度

时间复杂度：O(n)
空间复杂度：O(n)

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)





