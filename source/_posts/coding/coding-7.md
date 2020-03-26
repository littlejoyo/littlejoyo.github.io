---
title: Coding:斐波那契数列
date: 2020-03-25 23:12:25
categories:
  - [算法题]
tag:
    - 算法
---

- 《剑指offer》算法题目7

- 关键词：递归、数列、栈

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

# 1.题目

大家都知道斐波那契数列，现在要求输入一个整数n。
请你输出斐波那契数列的第n项（从0开始，第0项为0）。

斐波那契数实例：{0,1,1,2,3,5,8,...}

> n<=39

# 2.解题思路

## 2.1 递归法

斐波那契数列的标准公式为：

> F(1)=1，F(2)=1, F(n)=F(n-1)+F(n-2)（n>=3，n∈N*）

很明显，根据公式可以使用递归的思想来开始实现。

```java
public class Solution {
    public int Fibonacci(int n) {
        if(n<=1){
            return n;
        }
        return Fibonacci(n-1) + Fibonacci(n-2);
    }
}
```

- 递归底层的原理使用到了栈的数据结果，如果n足够大的时候，很可能会出现栈溢出的现象，最终导致程序异常退出。

- 时间复杂度也是比较高，随着n的增大呈现指数递增

复杂度：

- 时间复杂度：O(2^n)
- 空间复杂度：O(1)

## 2.2 利用数组存储优化递归

- 递归会重复计算大量相同数据，我们用可以声明一个数组把计算获得的结果存起来

- 然后再通过循环进行公式的计算。

```java
public class Solution {
    public int Fibonacci(int n) {
        int ans[] = new int[40];
        ans[0] = 0;
        ans[1] = 1;
        for(int i=2;i<=n;i++){
            ans[i] = ans[i-1] + ans[i-2];
        }
        return ans[n];
    }
}
```

复杂度

- 时间复杂度：O(n)
- 空间复杂度：O(n) 


## 2.3 优化存储空间

上面的方法额外申请了长度为n的数组进行数据的存储

但是实际上我们每次只用到了最近的两个数，所以我们可以只存储最近的两个数

- sum 存储第 n 项的值
- one 存储第 n-1 项的值
- two 存储第 n-2 项的值

```java
public class Solution {
    public int Fibonacci(int n) {
        if(n <= 1){
            return n;
        }
        int sum = 0;
        int two = 0;
        int one = 1;
        for(int i=2;i<=n;i++){
            sum = two + one;
            two = one;
            one = sum;
        }
        return sum;
    }
}
```

复杂度

- 时间复杂度：O(n)
- 空间复杂度：O(1)

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)

