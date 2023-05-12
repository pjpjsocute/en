---

title: LeetCode-406
author: Ray
top: true
cover: false
date: 2023-05-12 16:13:50
categories: technology
tags: 
  - LeetCode
  - java
---

### Q:

You are given an array of people, `people`, which are the attributes of some people in a queue (not necessarily in order). Each `people[i] = [hi, ki]` represents the `ith` person of height `hi` with **exactly** `ki` other people in front who have a height greater than or equal to `hi`.

Reconstruct and return *the queue that is represented by the input array* `people`. The returned queue should be formatted as an array `queue`, where `queue[j] = [hj, kj]` is the attributes of the `jth` person in the queue (`queue[0]` is the person at the front of the queue).

<!-- more -->

#### Input:

```
Input: people = [[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]
Output: [[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]
Explanation:
Person 0 has height 5 with no other people taller or the same height in front.
Person 1 has height 7 with no other people taller or the same height in front.
Person 2 has height 5 with two persons taller or the same height in front, which is person 0 and 1.
Person 3 has height 6 with one person taller or the same height in front, which is person 1.
Person 4 has height 4 with four people taller or the same height in front, which are people 0, 1, 2, and 3.
Person 5 has height 7 with one person taller or the same height in front, which is person 1.
Hence [[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]] is the reconstructed queue.
```

### S:

A person has 2 attributes , h stands for height, and the person with the same height should be behind the person with k greater. In order to determine where should go, we can consider ranking the heights backwards so that the number of people taller than him in front is known. Suppose some queue goes to , for the people who have been inserted in front of him, the heights are ≥ him, so when he is inserted in the ith position, it has no effect on the others (because when the heights are the same,i will be sorted in positive order)

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        Arrays.sort(people,new Comparator<int[]>(){
            @Override
            public int compare(int[] o1, int [] o2) {
                return o1[0]==o2[0]?o1[1]-o2[1]:o2[0]-o1[0];
            }
        });
        List<int[]> res = new ArrayList<>();
        for(int [] peo:people){
            res.add(peo[1], peo);
        }
        return res.toArray(new int [res.size()][2]);
    }
}
```

