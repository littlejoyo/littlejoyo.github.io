---
title: Coding:替换空格
date: 2020-03-20 10:01:10
categories:
  - [算法题]
tag:
    - 算法
---

- 《剑指offer》算法题目2

- 关键词：字符串、替换

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

# 1.题目

请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

- 输入：We Are Happy

- 输出：We%20Are%20Happy

# 2.解题思路

## 2.1 利用Java自带的函数

- 用Java自带的函数replace进行字符的替换。

```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
        return str.toString().replace(" ", "%20");
    }
}
```

## 2.2 用新的数组存

- 当遇到 " "，就追加 "%20"，否则遇到什么追加什么

```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
        StringBuilder sb = new StringBuilder();
        for(int i=0;i<str.length();i++){
            char c = str.charAt(i);
            if(c == ' '){
                sb.append("%20");
            }else{
                sb.append(c);
            }
        }
        return sb.toString();
    }
}
```

## 2.3 在当前字符串上进行替换

- 先计算替换后的字符串需要多大的空间，并对原字符串空间进行扩容；

- 从后往前替换字符串的话，每个字符串只需要移动一次；

- 如果从前往后，每个字符串需要多次移动，效率较低。

```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
        int spacenum = 0;
        for(int i = 0; i < str.length(); i++){
            if(str.charAt(i) == ' '){
                spacenum++;
            }
        }
        int oldLength = str.length();
        int oldIndex = oldLength - 1;
        int newLength = oldLength + spacenum*2;
        str.setLength(newLength);
        int newIndex = newLength - 1;
        for(; oldIndex >= 0 && oldLength < newLength; oldIndex--){
            if(str.charAt(oldIndex) == ' '){
                str.setCharAt(newIndex--, '0');
                str.setCharAt(newIndex--, '2');
                str.setCharAt(newIndex--, '%');
            }else{
                str.setCharAt(newIndex--, str.charAt(oldIndex));
            }
        }
        return str.toString();
    }
}
```

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)




