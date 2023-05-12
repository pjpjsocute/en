---
title: LeetCode-934
author: Ray
top: true
cover: false
date: 2023-05-12 15:22:55
categories: 技术
tags: 
  - 力扣之旅
  - java
  - DFS
---

## LeetCode-934

### Question:

给你一个大小为 `n x n` 的二元矩阵 `grid` ，其中 `1` 表示陆地，`0` 表示水域。

**岛** 是由四面相连的 `1` 形成的一个最大组，即不会与非组内的任何其他 `1` 相连。`grid` 中 **恰好存在两座岛** 。

你可以将任意数量的 `0` 变为 `1` ，以使两座岛连接起来，变成 **一座岛** 。

返回必须翻转的 `0` 的最小数目。

<!-- more -->

### Solution:

因为题干中只有2个岛，所以我们可以使用深搜，先找到其中一个岛。



对这个岛使用广度优先搜索，可以理解为对这个岛每次向外拓展1，当拓展第N次找到另外一个岛时，则为题干所求解。



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

