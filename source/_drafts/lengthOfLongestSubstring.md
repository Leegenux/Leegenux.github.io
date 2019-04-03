---
title: Algorithm Practice: Length Of Longest Substring Containing No Repetitive Characters.
tags: [algorithm]
---

This post is not meant for readers of this site but for myself. So content might get confusing sometimes. However you will find the reference links useful.

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

For third loop, the time complexity of the third loop is $O(j-i)$ because we only need to at most compare the character from index `i` to `j - 1` with the `j`th character. Thus, the second loop have a time complexity of $O(\sum_{j=i+1}^{n}(j-i)) = O(\frac{(n - i)(i + 1 + n)}{2})$. So the total time complexity is $O(\sum_{i=0}^{n-1}{\frac{(n-i)(i+1+n)}{2}})$, which equals to $O(n^3)​$

## My Solution

```C
int lengthOfLongestSubstring(char* s) {
    int maxLength = 0;
    int i=0, j=0, k=0;
    int strLen = strlen(s);
    for (i = 0; i < strLen - maxLength; i++) {
        
        char characterCountArray[128] = {0};
        characterCountArray[s[i]] += 1;

        for (j = i+1; (j < strLen) && !characterCountArray[s[j]]; j++) {
            characterCountArray[s[j]] += 1;
        }
        maxLength = (j-i)>maxLength ? (j-i) : maxLength;
    }
    return maxLength;
}
```
