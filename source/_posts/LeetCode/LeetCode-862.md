---
title: LeetCode-862
author: Ray
top: true
cover: false
date: 2023-05-12 16:14:10
categories: 技术
mathjax: true
tags: 
  - 力扣之旅
  - java
  - 前缀和
  - 单调队列
---

### Q:

给你一个整数数组 `nums` 和一个整数 `k` ，找出 `nums` 中和至少为 `k` 的 **最短非空子数组** ，并返回该子数组的长度。如果不存在这样的 **子数组** ，返回 `-1` 。

**子数组** 是数组中 **连续** 的一部分。

<!-- more -->

### Input:

```
输入：nums = [2,-1,2], k = 3
输出：3
```

### S:

​	求连续子序列的和，很容易想到使用前缀和。

​	因为数组是存在正负的，所以无法使用二分，而暴力破解会超时。

#### **优化：**

​    因为是求区间最短，可以很显然可以想到滑动窗，但是这个数组并不满足单调性：

​		数组中存在**负数**，导致窗口值**不单调**，但是因为有负数所以才会导致当我们找到某个窗口和为K，窗内依然可能存在可行解，原因如下：

​    **对于索引$i_j$前面满足≥K的所有索引${i_{0-j}}$，**

​	**如果$i_1$<$i_2$，arr[$i_1$]>arr[$i_2$],**

​	**那么可行解一定是$i_2$，**

​	**因为$i_2$更大且arr[$i_2$]更小**

所以**我们可以维护一个单调队列保证窗口内值的单调性：**

#### 思路：

​	基于我们总是希望对于每个右指针j，左指针能够尽可能的靠近，并且值尽可能地大。

​	如果有一个i-1的值 >i 处的值，那么i-1处的值就一定不是正确解，因为i处的值更近并且能够得到的数组和更大，如果i-1满足i一定满足，以此来减少我们的判断量

​    如果队首的值满足当前值-队首值>=K,记录长度并弹出队首

​    如果当前值<队列尾，那么弹出队尾保持队列单调

```java
class Solution {
    public int shortestSubarray(int[] A, int K) {
        long [] arr = new long [A.length+1];
        for(int i=0;i<A.length;i++){
            arr[i+1] = arr[i]+A[i];
            if(A[i]>=K) return 1;
        }//得到前缀和数组/ get pre sum
        int res = Integer.MAX_VALUE;
        // for(int i=0;i<=A.length-1;i++){  //暴力破解 N^2 超时/O(N^2) out time
        //     for(int j = i+1;j<=A.length;j++){
        //         if(arr[j]-arr[i]>=K){
        //             res = Math.min(j-i,res);
        //         }
        //     }
        // }
      	
        Deque<Integer> queue = new ArrayDeque<>();
        for(int i=0;i<arr.length;i++){
            while(!queue.isEmpty()&&arr[i]<=arr[queue.getLast()])   queue.removeLast();
            while(!queue.isEmpty()&&arr[i]-arr[queue.peek()]>=K)    res = Math.min(res,i-queue.poll());
            queue.add(i);
        }
        return res==Integer.MAX_VALUE?-1:res;
    }
}
```

