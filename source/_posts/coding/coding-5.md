---
title: Coding:用两个栈实现队列
date: 2020-03-23 22:19:02
categories:
  - [算法题]
tag:
    - 算法
---

- 《剑指offer》算法题目5

- 关键词：栈、队列

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

# 1.题目

- 用两个栈来实现一个队列，完成队列的Push和Pop操作。 

- 队列中的元素为int类型。

# 2.解题思路

## 2.1 分析

- 队列的特性是：“先入先出”，栈的特性是：“先入后出”

- 当我们向模拟的队列插入数 a,b,c 时，假设插入的是 stack1，此时的栈情况为：

```
栈 stack1：{a,b,c}
栈 stack2：{}
```

- 当需要弹出一个数，根据队列的"先进先出"原则，a 先进入，则 a 应该先弹出。

- 但是此时 a 在 stack1 的最下面，将 stack1 中全部元素逐个弹出压入 stack2，现在可以正确的从 stack2 中弹出 a，此时的栈情况为：

```
栈 stack1：{}
栈 stack2：{c,b,a}  ==> stack2: {c,b}
```

- 继续弹出一个数，b 比 c 先进入"队列"，b 弹出，注意此时 b 在 stack2 的栈顶，可直接弹出，此时的栈情况为：

```
栈 stack1：{}
栈 stack2：{c,b}  ==> stack2: {c}
```

- 此时向模拟队列插入一个数 d，还是插入 stack1，此时的栈情况为：

```
栈 stack1：{d}
栈 stack2：{c}
```

- 弹出一个数，c 比 d 先进入，c 弹出，注意此时 c 在 stack2 的栈顶，可直接弹出，此时的栈情况为：

```
栈 stack1：{d}
栈 stack2：{c}  ==> stack2: {}
```
根据上述过程可得出结论：

- 当插入时，直接插入 stack1

- 当弹出时，当 stack2 不为空，弹出 stack2 栈顶元素，如果 stack2 为空，将 stack1 中的全部数逐个出栈入栈 stack2，再弹出 stack2 栈顶元素

## 2.2 代码实现

```java
import java.util.Stack;
 
public class Solution {
    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();
 
    public void push(int node) {
        stack1.push(node);
    }
 
    public int pop() {
        if (stack2.isEmpty()) {
            while (!stack1.isEmpty()) {
                stack2.push(stack1.pop());
            }
        }
        return stack2.pop();
    }
}
```

## 2.3 复杂度

时间复杂度：

- pop:O(n)
- push:O(1)

空间复杂度：

- pop:O(n)
- push:O(1)

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)







