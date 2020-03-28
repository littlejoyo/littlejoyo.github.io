---
title: Coding:矩形覆盖
date: 2020-03-28 22:14:02
categories:
  - [算法题]
tag:
    - 算法
---

- 《剑指offer》算法题目10

- 关键词：递归、斐波那契数

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

# 1.题目

我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。
请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？

比如n=3时，2*3的矩形块有3种覆盖方法：

![例图](https://i.loli.net/2020/03/28/RjaEAgJn2IO1m9V.png)

# 2.解题思路

## 2.1 分析

![分析](https://i.loli.net/2020/03/28/jXGy9Sq8ZJMvH1z.png)

- 第一种情况等价于情形1中阴影部分的n-1块矩形有多少种覆盖方法，为f(n-1);

- 第二种情况等价于情形2中阴影部分的n-2块矩形有多少种覆盖方法，为f(n-2);

- 故f(n) = f(n-1) + f(n-2)，属于一个斐波那契数列


**总结:**

1.当n=0时，f(0) = 0

2.当n=1时，即只有一种放置的方法，f(1) = 1

3.当n=2时，可以竖着放和横着放两种方法，f(2) = 2

4.当n>2时，f(n) = f(n-1)+f(n)

## 2.2 代码实现

**递归**

```java
public class Solution {
    public int RectCover(int target) {
        if(target<=2){
            return target;
        }else {
            return RectCover(target-1)+RectCover(target-2);
        }
    }
}
```

**性能优化**

```java
public class Solution {
    public int RectCover(int target) {
        if(target<=2){
            return target;
        }
        int f=1,fn=2;
        int sum = 0;
        for(int i=3;i<=target;i++){
            sum = f + fn;
            f = fn;
            fn = sum;
        }
        return sum;
    }
}
```

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)

