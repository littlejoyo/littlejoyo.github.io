---
title: Coding:变态跳台阶
date: 2020-03-27 22:14:28
categories:
  - [算法题]
tag:
    - 算法
---

- 《剑指offer》算法题目9

- 关键词：递归、贪心算法

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

# 1.题目

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。
求该青蛙跳上一个n级的台阶总共有多少种跳法。

1级：1种
2级：2种
3级：4种
4级：8种
......

# 2.解题思路

## 2.1 分析

因为n级台阶，第一步有n种跳法：跳1级、跳2级、到跳n级

- 跳1级，剩下n-1级，则剩下跳法是f(n-1)

- 跳2级，剩下n-2级，则剩下跳法是f(n-2)

- 所以f(n)=`f(n-1)`+f(n-2)+...+f(1)

- 因为`f(n-1)`=f(n-2)+f(n-3)+...+f(1)

- 所以f(n)=2*f(n-1)

- 当n=0时，f(n) = 0

- 最后总结规律，通过递归的思想进行解答。

## 2.2 实现代码

### 递归

```java
public class Solution {
    public int JumpFloorII(int target) {
        if (target <= 0) {
            return 0;
        } else if (target == 1) {
            return 1;
        } else {
            return 2 * JumpFloorII(target - 1);
        }
    }
}
```

### 正向循环

```java
public class Solution {
    public int JumpFloorII(int target) {
        int f=1,fn=1;
        for(int i=2;i<=target;i++){
            fn=2*f;
            f=fn;
        }
        return fn;
    }
}
```

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)


