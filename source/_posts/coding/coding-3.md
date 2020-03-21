---
title: Coding:从尾到头打印链表
date: 2020-03-21 12:22:50
categories:
  - [算法题]
tag:
    - 算法
---

- 《剑指offer》算法题目3

- 关键词：链表、倒序输出

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

# 1.题目

输入一个链表，按链表从尾到头的顺序返回一个ArrayList。

- 链表结构：

```java
public class ListNode {
        int val;
        ListNode next = null;

        ListNode(int val) {
            this.val = val;
        }
    }
```

# 2.解题思路

## 2.1 递归

- 因为要对链表进行从尾部开始输出val的值，所以可以直接使用递归来解决。

```java
import java.util.*;
public class Solution {
    ArrayList<Integer> list = new ArrayList();
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        if(listNode!=null){
            printListFromTailToHead(listNode.next);
            list.add(listNode.val);
        }
        return list;
    }
}
```

- 时间复杂度：O(n)

- 空间复杂度：O(n)

## 2.2 利用栈

- listNode 是链表，只能从头遍历到尾，但是输出却要求从尾到头

- 这是典型的"先进后出"，可以利用栈的数据结构进行解答

```java
import java.util.*;
public class Solution {
    ArrayList<Integer> list = new ArrayList<>();
    Stack<Integer> stack = new Stack<>();
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        while(listNode!=null){
            stack.push(listNode.val);
            listNode = listNode.next;
        }
        
        while(!stack.empty()){
            list.add(stack.pop());
        }
        return list;
    }
}
```

- 时间复杂度：O(n)

- 空间复杂度：O(n)

## 2.3 利用ArrayList的内置方法

- ArrayList 中有个方法是 add(index,value)，可以指定 index 位置插入 value 值

- 所以我们在遍历 listNode 的同时将每个遇到的值插入到 list 的 0 位置，最后输出 listNode 即可得到逆序链表

```java
import java.util.*;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ArrayList<Integer> list = new ArrayList<>();
        ListNode tmp = listNode;
        while(tmp!=null){
            list.add(0,tmp.val);
            tmp = tmp.next;
        }
        return list;
    }
}
```

- 时间复杂度：O(n)

- 空间复杂度：O(n)

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)
