---
title: LeetCode-934
author: Ray
top: true
cover: false
date: 2023-05-12 15:22:55
categories: technology
tags: 
  - LeetCode
  - java
  - DFS
---

## LeetCode-934

### Question:

You are given an `n x n` binary matrix `grid` where `1` represents land and `0` represents water.

An **island** is a 4-directionally connected group of `1`'s not connected to any other `1`'s. There are **exactly two islands** in `grid`.

You may change `0`'s to `1`'s to connect the two islands to form **one island**.

Return *the smallest number of* `0`*'s you must flip to connect the two islands*.

<!-- more -->

### Solution:

Since there are only 2 islands in the stem, we can use a deep search to find one of them first.



Using breadth-first search for this island can be interpreted as expanding this island outward 1 at a time, and when the expansion finds the other island for the Nth time, it is the solution sought by the stem.

### 

### Code：

```java
class Solution {
    public int shortestBridge(int[][] A) {
        int [][] direction = new int [][]{{1,0},{-1,0},{0,1},{0,-1}};
        Deque<int []> queue = new ArrayDeque<>();
        int ans = -1;
        boolean [][] visited = new boolean[A.length][A[0].length];
        boolean flag = true;
        for(int i=0;i<A.length&&flag;i++){
            for(int j=0;j<A[0].length;j++) {
                if (A[i][j] == 1) {
                    dfs(  A, i, j, queue, visited);
                    flag = false;
                    break;
                }
            }
        }
        while (!queue.isEmpty()){
            int size = queue.size();
            ans++;
            for(int i=0;i<size;i++){
                int []node = queue.poll();
                for(int j=0;j<4;j++){
                    int  nx = node[0]+direction[j][0];
                    int ny = node[1]+direction[j][1];
                    if(nx<0||nx>=A.length||ny<0||ny>=A[0].length||visited[nx][ny])    continue;
                    if(A[nx][ny]==1)    return ans;
                    visited[nx][ny] = true;
                    queue.add(new int []{nx,ny});
                }
            }
        }
        return ans;
    }
    private void dfs(int [][]A,int i,int j,Deque queue,boolean[][]visited){
        if(i<0||i>=A.length||j<0||j>=A[0].length||visited[i][j]||A[i][j]!=1)    return;
        visited[i][j] = true;
        queue.add(new int []{i,j});
        dfs( A, i-1, j, queue, visited);
        dfs( A, i+1, j, queue, visited);
        dfs( A, i, j-1, queue, visited);
        dfs( A, i, j+1, queue, visited);
        
    }
}

```

