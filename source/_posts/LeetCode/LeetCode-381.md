---
title: LeetCode-381
author: Ray
top: true
cover: false
date: 2023-05-12 17:14:10
categories: technology
mathjax: true
tags: 
  - LeetCode
  - java
  - Data structures
---

### Q:

`RandomizedCollection` is a data structure that contains a collection of numbers, possibly duplicates (i.e., a multiset). It should support inserting and removing specific elements and also reporting a random element.

Implement the `RandomizedCollection` class:

- `RandomizedCollection()` Initializes the empty `RandomizedCollection` object.
- `bool insert(int val)` Inserts an item `val` into the multiset, even if the item is already present. Returns `true` if the item is not present, `false` otherwise.
- `bool remove(int val)` Removes an item `val` from the multiset if present. Returns `true` if the item is present, `false` otherwise. Note that if `val` has multiple occurrences in the multiset, we only remove one of them.
- `int getRandom()` Returns a random element from the current multiset of elements. The probability of each element being returned is **linearly related** to the number of the same values the multiset contains.

You must implement the functions of the class such that each function works on **average** `O(1)` time complexity.

**Note:** The test cases are generated such that `getRandom` will only be called if there is **at least one** item in the `RandomizedCollection`.

<!-- more -->

### Input:

```
Input
["RandomizedCollection", "insert", "insert", "insert", "getRandom", "remove", "getRandom"]
[[], [1], [1], [2], [], [1], []]
Output
[null, true, false, true, 2, true, 1]

Explanation
RandomizedCollection randomizedCollection = new RandomizedCollection();
randomizedCollection.insert(1);   // return true since the collection does not contain 1.
                                  // Inserts 1 into the collection.
randomizedCollection.insert(1);   // return false since the collection contains 1.
                                  // Inserts another 1 into the collection. Collection now contains [1,1].
randomizedCollection.insert(2);   // return true since the collection does not contain 2.
                                  // Inserts 2 into the collection. Collection now contains [1,1,2].
randomizedCollection.getRandom(); // getRandom should:
                                  // - return 1 with probability 2/3, or
                                  // - return 2 with probability 1/3.
randomizedCollection.remove(1);   // return true since the collection contains 1.
                                  // Removes 1 from the collection. Collection now contains [1,2].
randomizedCollection.getRandom(); // getRandom should return 1 or 2, both equally likely.
```

### S:

Because the topic needs to find out if the value exists when inserting, we need to achieve a lookup of O1, so there are lists and hashmaps (lists by index) that can do this.

However, since list cannot directly find the value of an element O1, we can consider using list and hashmap together.

The map stores (value, index) and the list stores the value. 

Since list can only be O1 when the tail element is deleted, we can swap the element to be deleted with the tail element and then delete the value and index key-value pairs stored in map.

Since a value may have multiple indexes, the indexes also need to be encapsulated in a collection.

 Considering that a value will not have 2 identical indexes, and that operations such as delete swap require the deletion of the worthy indexes, set is used to hold the index sequence

```java
class RandomizedCollection {
    int n ;// collection size
    HashMap<Integer,Set<Integer>>map;
    ArrayList<Integer>list;
    Random random;
    /** Initialize your data structure here. */
    public RandomizedCollection() {
        this.random = new Random();
        this.map = new HashMap();
        this.n = 0;
        this.list = new ArrayList<>();
    }
    
    /** Inserts a value to the collection. Returns true if the collection did not already contain the specified element. */
    public boolean insert(int val) {
        Set set = map.get(val);
        if(set==null)   set = new HashSet<>();
        set.add(n);// add index
        list.add(val);
        map.put(val, set);
        n++;
        return set.size()==1;
    }
    
    /** Removes a value from the collection. Returns true if the collection contained the specified element. */
    public boolean remove(int val) {
        if(map.containsKey(val)){
            int lastIndex = n-1;// get last index
            Set lastset = map.get(list.get(lastIndex));
            Set set = map.get(val);
            int currIndex = (int)set.iterator().next();//get current index  tip:: iterator in set is ordered when you use next
            //delete
            swap(list, currIndex, lastIndex);
            list.remove(n-1);// list delete
            set.remove(currIndex);//delete set
            if(set.size()==0)   map.remove(val);//delete map
            //edit last index
            lastset.remove(n-1);
            lastset.add(currIndex);
            n--;
        }else{
            return false;
        }
        return true;
    }
    
    /** Get a random element from the collection. */
    public int getRandom() {
        return list.get(random.nextInt(n));
    }
    private void swap(List<Integer> list ,int i,int j){
        int temp = list.get(i);
        list.set(i, list.get(j));
        list.set(j, temp);
    }
}
```

