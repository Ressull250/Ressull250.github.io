---
layout:     post
title:      Substring类问题求解
subtitle:   
date:       2018-04-18
author:     LiaoBo
header-img: img/header_02.jpg
catalog: true
tags:
    - 算法
---
# 求解substring相关问题

**范式**：

- 1、初始化map

- 2、用start和end表示当前子串，遍历

- 3、根据条件改变状态

- 4、与当前最大（小）子串比较

**伪码**：[参考](https://leetcode.com/problems/minimum-window-substring/discuss/26808/Here-is-a-10-line-template-that-can-solve-most-'substring'-problems)

```c++
int findSubstring(string s){
        vector<int> map(128,0);
        int counter; // check whether the substring is valid
        int begin=0, end=0; //two pointers, one point to tail and one  head
        int d; //the length of substring

        for() { /* initialize the hash map here */ }

        while(end<s.size()){

            if(map[s[end++]]-- ?){  /* modify counter here */ }

            while(/* counter condition */){ 
                 
                 /* update d here if finding minimum*/

                //increase begin to make it invalid/valid again
                
                if(map[s[begin++]]++ ?){ /*modify counter here*/ }
            }  

            /* update d here if finding maximum*/
        }
        return d;
  }
```

**实例**：

- 题目：Given a string S and a string T, find the minimum window in S which will contain all the characters in T in complexity O(n).

  **S** = `"ADOBECODEBANC"`

  **T** = `"ABC"`

  Minimum window is `"BANC"`

- 代码：

```Python
import sys
class Solution:
    def minWindow(self, s, t):
        map = [0 for _ in range(128)]
        begin,end,minbegin,minlen = 0,0,0,sys.maxsize
        counter = len(t)

        # 初始化map
        for c in t:
            map[ord(c)]+=1

        while end<len(s):
            # 改变条件
            if map[ord(s[end])] > 0:
                counter-=1
            map[ord(s[end])]-=1
            end+=1

            while counter == 0:
                # 判断是否是最小
                if end-begin<minlen:
                    minlen = end-begin
                    minbegin = begin
                
                # 移动begin
                map[ord(s[begin])]+=1
                if map[ord(s[begin])]>0:
                    counter+=1
                begin+=1

        if minlen == sys.maxsize:
            return ""
        return s[minbegin:minbegin+minlen]
```

**总结**：

大多数substring的问题都应该是能在o(n)内解决的，通过map来处理string类是极好的