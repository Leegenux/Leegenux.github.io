---
title: Problemi --- Add One
tags:
---

https://leetcode-cn.com/submissions/detail/20039459/
understanding the circumstances is really important!

int* plusOne(int* digits, int digitsSize, int* returnSize) {
    *returnSize = digitsSize;
   for (int i = digitsSize - 1;i >= 0;i--){
       if (digits[i] != 9){
           digits[i] += 1;
           
           return digits;
           
       }
       
       digits[i] = 0;
   }
    
    int *rev = malloc(sizeof(int) * (digitsSize + 1));
    rev[0] = 1;
    for (int i = 1;i < digitsSize + 1;i++){
        rev[i] = 0;
    }
    *returnSize = digitsSize + 1; 
    
    return rev;
}
