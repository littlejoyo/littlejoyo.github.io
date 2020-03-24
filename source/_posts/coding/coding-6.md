---
title: Coding:旋转数组的最小数字
date: 2020-03-24 22:25:44
categories:
  - [算法题]
tag:
    - 算法
---

- 《剑指offer》算法题目6

- 关键词：数组、查找

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

# 1.题目

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。

输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。

例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。

> NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。

# 2.解题思路

## 2.1 分析

单纯使用排序后获取最小值的话很简单，但是时间复杂度是O(n),一般都会要求更低的时间复杂度。

所以可以使用二分查找的思想，用中间值和高低位进行比较，看处于递增还是递减序列，进行操作缩小范围。

- 处于递增：low上移

- 处于递减：high下移（如果是high-1，则可能会错过最小值，因为找的就是最小值）

- 其余情况：low++缩小范围

![index](https://i.loli.net/2020/03/24/fyiVKAt6QEG9nWu.png)

- 特殊情况

![index](https://i.loli.net/2020/03/24/kK4p7WHGtM9cmvl.png)

## 2.2 代码实现

```java
import java.util.ArrayList;
public class Solution {
    public int minNumberInRotateArray(int [] array) {
        if(array.length==0)
            return 0;
 
        int low = 0;
        int high = array.length - 1;
        int mid = 0;
 
        while(low < high){
            // 子数组是非递减的数组，10111
            if (array[low] < array[high])
                return array[low];
            // 防溢出
            mid = low + (high - low) / 2; 
            if(array[mid] > array[low])
                low = mid + 1;
            else if(array[mid] < array[high])
                high = mid;
            else low++;
        }
        return array[low];
    }
}
```

## 2.3 时间复杂度

- O(logn)

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)
