---
title: >-
  Algorithm Practice --- Length Of Longest Substring Containing No Repetitive
  Characters.
tags:
  - algorithm
date: 2019-04-07 00:28:30
---


This post is not meant for readers of this site but for myself. So content might get confusing sometimes. However you will find the [reference link](https://leetcode.com/problems/longest-substring-without-repeating-characters/) useful.

## Problem Description

Given a ascii string, please find out the length of longest substring that contains no repetitive characters.

## Solution 1: Brute Force

Start and ending charaters together indicate a substring, and we can iterate over all the start-ending pairs to go over all substrings. 

There should be 3 loops. The first is letting `i` goes through all the characters. Then for every `i` value, `j` grows from `i + 1` to the end of whole string. Then we determine whether there are repetitives or not for each substring.

Code is as follows:
```C
int lengthOfLongestSubstring(char* s) {
    int maxLength = 0;
    int i=0, j=0, k=0;
    int strLen = strlen(s);
    for (i = 0; i < strLen - maxLength; i++) {
        for (j = i+1; (j < strLen) && s[j] != s[i]; j++) {
            for (k = i+1; (k < j) && (s[k] != s[j]); k++);
			if (k != j) break;
        }
        maxLength = (j-i)>maxLength ? (j-i) : maxLength;
    }
    return maxLength;
}
```

### Algorithm Complexity of Solution 1

For third loop, the time complexity of the third loop is $O(j-i)​$ because we only need to at most compare the character from index `i` to `j - 1` with the `j`th character. Thus, the second loop has a time complexity of $O(\sum_{j=i+1}^{n}(j-i)) = O(\frac{(n - i)(i + 1 + n)}{2})​$. So the total time complexity is $O(\sum_{i=0}^{n-1}{\frac{(n-i)(i+1+n)}{2}})​$, which equals to $O(n^3)​$

## Solution 2: Sliding Window

In previous solution, we spend much time checking whether there are repetitive characters of same string over and over again. But have you thought of keeping previous comparison results?

Here comes an greatly improved solution:
```C
int lengthOfLongestSubstring(char* s) {
    int i, j;
    int strLen = strlen(s);
    int maxLength = (strLen==1);
    int characterCountArray[128] = {0};

    for (i=0, j=1, characterCountArray[s[i]]=1; i < strLen-maxLength && j < strLen; ) {
        if (characterCountArray[s[j]]) {
            characterCountArray[s[i++]] = 0;
        } else {
            characterCountArray[s[j++]] = 1;
            maxLength = (j-i) > maxLength ? (j-i) : maxLength;
        }
    }
    return maxLength;
}
```
There are 3 points that you need to look closer at:
- The big loop breaks when `i >= strLen - maxLength`
- The comparison results are kept in the sliding window.
- Only after `j++` does `maxLength` need updates.

### Algorithm Complexity of Solution 2

In worst condition, all elements of the string will be access twice by `i` and `j`. So the time complexity is $O(n)$.
