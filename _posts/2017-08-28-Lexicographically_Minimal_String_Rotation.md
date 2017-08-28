---
layout:   single
title: "Lexicographically Minimal String Rotation"
date: 2017-07-11 15:57:00
categories: Algorithms
tags: Algorithms
---

{% include toc %}

## 简介
题目：使用一次字符串旋转，得到该字符串的最低词典序列的形式。

比如`lllllllabllll`和`lllllllballll`两个字符串，前者的词典序列更低，因为两者第一个不同的字符较小的字符串，其词典序列较低。

再比如`ccddaaccaabbklj`按照题目要求的结果应该为`aabbkljccddaacc`

## 思路
使用暴力解法，可以在$$O(n^2)$$时间复杂度内解决。但是我们可以通过数组来记录已匹配的字符串，来防止重复检查。`Booth Algorithm`就可以达到$$O(n)$$的时间复杂度。

## 实现
```python
def lcs(S):
    S += S      # 连接字符串，以防尾头相接的情况
    f = [-1] * len(S)     # 记录已匹配的字符串长度
    k = 0       # 最低词典序列的index
    for j in xrange(1,len(S)):
        sj = S[j] # 提取当前字符
        i = f[j-k-1] # 提取已匹配字符串的长度
        while i != -1 and sj != S[k+i+1]:
            # 这个for循环来检查之前连续匹配的字符串中，是否包含比起始index对应的字符更小的字符
            if sj < S[k+i+1]:
                k = j-i-1
            i = f[i]
        if sj != S[k+i+1]: # 判断当前字符与已匹配的字符串后的字符是否相匹配
            if sj < S[k]: # k+i+1 = k
                k = j
            f[j-k] = -1
        else:
            f[j-k] = i+1 # 如果匹配，在上一个index的值的基础上加一
    return k
```
