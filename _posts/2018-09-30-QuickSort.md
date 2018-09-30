---
layout:     post
title:      QuickSort & QuickSelect
subtitle:   
date:       2018-09-30
author:     LiaoBo
header-img: img/header_02.jpg
catalog: true
tags:
    - 算法
---
## 复习快排

- 优势 **对cache友好**

- 思路

  - 找一个基准点p，左边是小于p的，右边是大于p的，左右递归

- 找基准点有三种实现方式

  - 1.左右指针

    - 上课教的，左右迭代至一个大于p一个小于p再交换

    ```python
    def qsort(nums, l, r):
        if l > r: return
        p = partition(nums, l, r)
        qsort(nums, l, p-1)
        qsort(nums, p+1, r)
        pass
    
    def partition(nums, l, r):
        key = r
        keyval = nums[r]
        while l < r:
            while l < r and nums[l] <= keyval:
                l += 1
            while l < r and nums[r] >= keyval:
                r -= 1
            nums[l], nums[r] = nums[r], nums[l]
        nums[l], nums[key] = nums[key], nums[l]
        return l
    ```

  - 2.挖坑法

    - 差不多

  - 3.前后指针 **可用于链表排序**

    - 在没找到大于key值前，pre永远紧跟cur，遇到大的两者之间机会拉开差距，中间差的肯定是连续的大于key的值，当再次遇到小于key的值时，交换两个下标对应的值

    ![image-20180930164802129](/Users/zyb/Library/Application Support/typora-user-images/image-20180930164802129.png)

- **优化**

  - 三数取中法
    - 直接拿数组最后或第一个值，在数组正序或逆序时，划分没有意义，效率退化
    - 在数组开始，中间，末尾选中间值来作为枢纽
  - 直接插入
    - 当递归到序列长度不大时使用直接插入，避免过多的递归调用带来的栈开销



## Find Kth Largest Number

Leetcode 215 - 应该是比较经典的题

- 排序 **O(NlogN)**

- ```python
  def findKthLargest1(self, nums, k):
      # 暴力出奇迹
      return sorted(nums)[-k]
  ```

- 维护一个len(k)的优先队列 **O(NlogK)**

  ```python
  def findKthLargest2(self, nums, k):
      q = Queue.PriorityQueue()
      for num in nums:
          q.put(num)
          if q.qsize() > k:
              q.get()
  
      return q.get()
  ```

- Quick Select 快排的变种 

  - **worstcase O(N^2) bestcase O(N)** 

  ```python
  def partition(self,nums, l, r):
      key = r
      keyval = nums[r]
      while l < r:
          while l < r and nums[l] <= keyval:
              l += 1
          while l < r and nums[r] >= keyval:
              r -= 1
          nums[l], nums[r] = nums[r], nums[l]
      nums[l], nums[key] = nums[key], nums[l]
      return l
  
  def findKthLargest(self, nums, k):
      l,r = 0,len(nums)-1
      # 第k大数逆序index=k-1 正序index就是len(nums)-k
      k = len(nums)-k
      while l <= r:
          p = self.partition(nums, l, r)
          if p > k:
              r = p-1
          elif p < k:
              l = p+1
          else:
              break
      return nums[p]
  ```

- 在运行之前shuffle就可以保证**O(N)**