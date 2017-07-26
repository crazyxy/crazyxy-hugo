---
title: "Interesting Problems from Coursera"
date: 2017-07-16T20:57:45+08:00
draft: false
tags: ["Algorithm", "Coursera"]
---

最近一直在Coursera上学习Princeton University的Algorithms。在学习过程中遇到了很多有趣的问题，在这里记录下来并分享给大家。

## 3-SUM $O(n^2)$ 时间复杂度的算法

3-SUM 问题如下，给定一个数组`a[N]`和`x`，问数组中存在几组`(i,j,k), i<j<k`使得`a[i]+a[j]+a[k]=x`。这道题早在Leetcode上就遇到了，具体的方法如下：

*（为简单起见，假设所有元素均不相同）*

```c
// Step 1: 将数组按照升序排序
// Step 2: 最外层循环一次从0遍历到N
// Step 3: 内层循环使用两个指针，分别指向i+1和N-1，问题变成了在一个数组中寻找一对数使它们的和为x-a[i]。
for(int i = 0; i < N; i ++){
    int j = i + 1, k = N - 1;
    while(j < k){
        int s = a[j]+a[k]+a[i];
        if(s==x){
            // find a new tuple
            break;
        }else if(s < x){
            j ++;
        }else{
            k --;
        }
    }
}
```

## 在双调数组（bitonic array）中找一个元素，要求时间复杂度$O(log n)$

### $\Theta(3 log n)$算法
- 先找出双调数组中的最大值 ===> $log n$次比较

```c++
int find_max(int* arr, int left, int right){
    while(left + 1 < right){
        int mid = (left + right) / 2;
        if(arr[mid] > arr[mid+1]){
            right = mid;
        }else{
            left = mid;
        }
    }
    return (arr[left] > arr[right] ? left : right);
}
```

- 分别在两边的子数组中使用二分查找 ===> $2log n$次比较

### $\Theta(2 log n)$算法

如果只允许使用$2logn$次比较，则不需要找最大值。具体算法如下：

0. 首先规定当前数组最左边为`left`，最右边为`right`，`mid=(left+right)/2`，需要找的元素为`x`；
1. 如果`arr[mid] = x`，程序结束；
2. 如果`arr[mid] < x`：
    - 如果`arr[mid] < arr[mid+1]`则在`mid+1`到`right`中递归查找`x`
    - 如果`arr[mid] < arr[mid-1]`则在`left`到`mid-1`中递归查找`x`
    - 否则，数组中不存在`x`
3. 如果`arr[mid] > x`，则在两边数组中使用二分查找找`x`。

为什么最后一步中子数组不是排好序的依旧可以使用二分查找？因为要查找的目标`x`均比没有排序的部分要小，因此不影响二分查找的结果。

## 高空扔鸡蛋

第一次见这个题目是在腾讯的笔试，当时的题目大意是：从高空中向下丢鸡蛋，在`T`层以下的鸡蛋都不会碎，而从`T`层以及`T`层向上往下抛鸡蛋就会碎。给2个鸡蛋，求抛鸡蛋的最优策略，使得在最坏情况下抛鸡蛋的次数最少。

首先什么是在最坏情况下抛鸡蛋的次数最少。假设现在有两个方案：

> 方案1: 按顺序从第一层开始抛鸡蛋。这种方案下，最坏情况抛鸡蛋的次数是`n`；

> 方案2: 先二分，如果鸡蛋碎了从第一层开始逐层向上，如果没有碎则继续二分。这种方案最坏情况抛鸡蛋的次数是`n/2`。

因此方案2优于方案1。那么最优方案是什么呢？

假设最优方案的最坏情况下需要抛`x`次，则第一个鸡蛋应该从第`x`层抛下。因为如果这个鸡蛋碎了，则最多只需要在抛`x-1`次就可以测出`T`；否则如果没有碎，问题变成了在`n-x`高的楼中抛鸡蛋，最多抛`x-1`次，因此第二次从第`x+x-1`楼向下抛，以此类推，第`x`次则是从`x+x-1+x-2+...+1`楼向下抛，所以只需要保证$\frac{x\times(x+1)}{2}\ge n$。

假设有`m`个鸡蛋，楼高`n`，最优方案是什么呢？

$dp[i][j]$表示`i`个鸡蛋从`j`层高的楼抛下最坏情况下需要的次数。递推方程如下：

$$
dp[i][j] = min(1+max(dp[i-1][k-1], dp[i][j-k])), \forall k \in [1,i]
$$

## Shuffling a linked list

链表中有`n`个元素，要求在满足空间复杂度`O(n)`，时间复杂度`O(n logn)`条件下shuffle链表。

Mergesort. 
```c++
// 1. a <- shuffled list
// 2. b <- shuffled list
// 3. r = random number % (len(a) + len(b))
// 4. if r < a then choose element from list a else choose element from b
// 5. go back to 3 until a or b is empty
```