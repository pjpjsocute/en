---
title: "LeetCode-005: How to Derive Dynamic Programming Step by Step"
date: 2023-05-21
updated: 2023-05-22
categories: technology
tags: 
  - LeetCode
  - java
  - Data structures
---

### Q:

Given a string `s`, find the longest palindromic substring in `s`.

A string is a palindrome if it reads the same backward as forward.

### Input:

```text
Input: s = "babad"
Output: "bab"
Explanation: "aba" is also a valid answer.
```

### Solution:

If this is your first time encountering such a problem, it might not be easy to immediately think of dynamic programming.  

However, with step-by-step reasoning and optimization, DP naturally emerges as a clean solution.

---

##### Step 1: Naive Approach

The simplest idea is:

1. Design a helper method `judgeIsPalindrome`.  
   Using a two-pointer technique, we can check whether `String(i, j)` is a palindrome in O(n).

2. Enumerate all substring lengths `k`, from large to small.  
   For each length, check all substrings `String(i, i+k-1)`.  
   If a palindrome is found, return it as the result.

This works, but it’s inefficient.  
- We must enumerate all `k` values (worst case: N).  
- For each `k`, we may check up to N substrings.  
- This leads to high complexity, which is not feasible.  

We need something more efficient.

---

##### Step 2: Binary Search Optimization

Instead of enumerating all possible lengths `k`, we could binary search for the maximum valid `k`.  

This reduces the search to O(log N).  
(Interested readers can try implementing this approach.)  

However, there’s still redundancy: each new substring is checked independently, without reusing previously computed results.

---

##### Step 3: Dynamic Programming Insight

Let’s improve by leveraging earlier computations.

Suppose for some length `k_i`, we already know whether every substring of length `k_i` is a palindrome.  

Now, consider extending a substring:  
If we know `String(i, i+k_i-1)` is a palindrome, then checking `String(i-1, i+k_i)` can be done in O(1):  

```
dp[i][j] = dp[i+1][j-1] && (s[i] == s[j])
```

That is:  
- If the inner substring `String(i+1, j-1)` is a palindrome,  
- and the boundary characters match (`s[i] == s[j]`),  
- then `String(i, j)` is also a palindrome.

---

##### DP Implementation

```java
public String longestPalindrome(String s) {
    int len = s.length();
    if (len < 2) {
        return s;
    }
    boolean[][] dp = new boolean[len][len];
    for (int i = 0; i < len; i++) {
        dp[i][i] = true;
    }
    int maxLen = 1;
    int start = 0;
    for (int j = 1; j < len; j++) {
        for (int i = j - 1; i >= 0; i--) {
            if (s.charAt(i) == s.charAt(j)) {
                if (j - i < 3) {
                    dp[i][j] = true;
                } else {
                    dp[i][j] = dp[i + 1][j - 1];
                }
            } else {
                dp[i][j] = false;
            }

            if (dp[i][j]) {
                int curLen = j - i + 1;
                if (curLen > maxLen) {
                    maxLen = curLen;
                    start = i;
                }
            }
        }
    }
    return s.substring(start, start + maxLen);
}
```

---

### Summary

- **Naive approach**: brute force with palindrome checking → O(N³).  
- **Optimization**: binary search on substring length → O(N² log N).  
- **Dynamic Programming**: reuse previously computed results → O(N²).  

This step-by-step refinement shows how dynamic programming becomes a natural and efficient solution.
