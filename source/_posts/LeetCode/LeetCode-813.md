---
title: According to LeetCode_813,How to use a recursion to dp
author: Ray
top: true
cover: false
date: 2023-05-13 16:14:10
categories: technology
mathjax: true
tags: 
  - LeetCode
  - java
  - Data structures
---

### Q:

You are given an integer array `nums` and an integer `k`. You can partition the array into **at most** `k` non-empty adjacent subarrays. The **score** of a partition is the sum of the averages of each subarray.

Note that the partition must use every integer in `nums`, and that the score is not necessarily an integer.

Return *the maximum **score** you can achieve of all the possible partitions*. Answers within `10-6` of the actual answer will be accepted.

<!-- more -->

### Input:

```
Input: nums = [9,1,2,3,9], k = 3
Output: 20.00000
Explanation: 
The best choice is to partition nums into [9], [1, 2, 3], [9]. The answer is 9 + (1 + 2 + 3) / 3 + 9 = 20.
We could have also partitioned nums into [9, 1], [2], [3, 9], for example.
That partition would lead to a score of 5 + 2 + 6 = 13, which is worse.
```

### S:

#### First

The most intuitive idea in solving this problem is that we can enumerate each case and finally get the optimal answer. So, we can solve the problem by recursive enumeration, by enumerating each partitioned case to get the final maximum value 
The recursive code is as follows:

```java
class Solution {
    public double largestSumOfAverages(int[] A, int K) {
        return dfs(A, 0, K);
    }
    private double dfs(int A[] ,int index,int K){
        if(K==0){
            return 0.0;
        }
        if(K==1){// k=1 return the total array sum
            int sum = 0;
            for(int i=index;i<A.length;i++){
                sum+=A[i];
            }
            return (double)sum/(A.length-index);
        }
        double sum = 0.0;
        double res = 0.0;
        for(int i=index;i<=A.length-K;i++){
            sum+=A[i];//Enumerate each separation point
            double avage = sum/(i-index+1);
            double temp = dfs(A,i+1,K-1);// next sub array sum 
            res = Math.max(res, avage+temp);//select max
        }
        return (double)res;
    }
}

```

#### Second

​		In general, using recursion for enumeration is not the optimal solution because it involves a lot of repetition, so we can use mnemonic recursion, which uses an array of recursive values that have already been obtained, so that when we enter the branch again, we can quickly obtain a solution. After understanding the above recursive code, it is easy to obtain the mnemonic recursive code with a little modification

```java
class Solution {
  	//use a array to record the value which has been caculate
    double [][] memo ;
    public double largestSumOfAverages(int[] A, int K) {
        this.memo = new double [A.length+1][K+1];
        return dfs(A, 0, K);
    }
    private double dfs(int A[] ,int index,int K){
        if(K==0){
            return 0.0;
        }
        if(memo[index][K]!=0.0)   return memo[index][K];
        if(K==1){
            int sum = 0;
            for(int i=index;i<A.length;i++){
                sum+=A[i];
            }
            memo[index][K] = (double)sum/(A.length-index);
            return memo[index][K];
        }
        double sum = 0.0;
        double res = 0.0;
        for(int i=index;i<=A.length-K;i++){
            sum+=A[i];
            double avage = sum/(i-index+1);
            // double temp = dfs(A,i+1,K-1);
            // memo[i+1][K-1] = temp;
            memo[i+1][K-1] = dfs(A,i+1,K-1);
            res = Math.max(res, avage+memo[i+1][K-1]);
        }
        memo[index][K] =res;
        return (double)res;
    }
}

```

#### Finally

​		Mnemonic recursion is actually very similar to dynamic programming, except that one is top-down tableau and the other is bottom-up tableau. Based on the idea of memetic recursion, we can rewrite memetic recursion as DP

```java
public double largestSumOfAverages(int[] A, int K) {
        double[][] dp = new double[A.length+1][K+1];
        double [] preSum = new double[A.length+1];
        for(int i=0;i<A.length;i++){
            preSum[i+1]= preSum[i]+A[i];
            dp[i+1][1] = preSum[i+1]/(i+1);//init
        }
        for(int i=1;i<=A.length;i++){
            for(int j=2;j<=Math.min(K, i);j++){
                //The maximum mean value of dp[i][j] should be the maximum of all possible values of dp[i'][j-1] + the mean sum of i'-i
                for(int t = 0;t<i;t++){
                    dp[i][j] = Math.max(dp[i][j], dp[t][j-1]+(preSum[i]-preSum[t])/(i-t));
                }
            }
        }
        return dp[A.length][K];
    }

```

