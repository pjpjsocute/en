---
title: LeetCode-862
author: Ray
top: true
cover: false
date: 2023-05-12 16:14:10
categories: technology
mathjax: true
tags: 
  - LeetCode
  - java
  - persum
  - queue

---

### Q:

Given an integer array `nums` and an integer `k`, return *the length of the shortest non-empty **subarray** of* `nums` *with a sum of at least* `k`. If there is no such **subarray**, return `-1`.

A **subarray** is a **contiguous** part of an array.

<!-- more -->

### Input:

```
Input: nums = [2,-1,2], k = 3
Output: 3
```

### S:

​		To find the sum of consecutive subsequences, it is easy to think of using prefix sums.

​		Since there are positive and negative arrays, it is not possible to use bifurcation, and brute force cracking will time out.

\#### **Optimization:**

Because it is to find the shortest interval, it can be obvious to think of sliding windows, but this array does not satisfy monotonicity: the

There are **negative numbers** in the array, resulting in window values **not monotonic**, but because there are negative numbers so that is why when we find a certain window and for K, there may still be feasible solutions within the window, for the following reasons:

**For all indexes ${i_{0-j}}$ that satisfy ≥K before index $i_j$,**

**If $i_1$<$i_2$ and arr[$i_1$]>arr[$i_2$],**

 **then the feasible solution must be $i_2$,**

 **because $i_2$ is larger and arr[$i_2$] is smaller**

So  **we can maintain a monotonic queue to guarantee the monotonicity of the values in the window:**

### Idea:

Based on the fact that we always want the left pointer to be as close as possible for each right pointer j and the value to be as large as possible.

If there is a value at i-1 > i, then the value at i-1 must not be the correct solution, because the value at i is closer and can get a larger array sum, if i-1 is satisfied i must be satisfied, so as to reduce the amount of our judgment

If the value at the top of the queue satisfies the current value - the value at the top of the queue >= K, record the length and pop the top of the queue



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

