---
title: problemIntersection
tags:
---


https://leetcode-cn.com/explore/interview/card/top-interview-questions-easy/1/array/26/

```

/**
 * Note: The returned array must be malloced, assume caller calls free().
 */
typedef struct hashNode {
    long key;
    int value;
    int hh;
} hashnode;

int* intersect(int* nums1, int nums1Size, int* nums2, int nums2Size, int* returnSize){
    int max = (nums1Size > nums2Size) ? nums1Size : nums2Size;
    int hashLength = max * 8;
    hashnode *hashMap = malloc(sizeof(hashnode)*hashLength);
    const hashnode nullNode = {.key=-1, .value=0, .hh=-1};
    
    for (int i = 0; i < hashLength; i++) {
        hashMap[i] = nullNode;
    }
    
    int *validFlag = malloc(sizeof(int));
    
    for (int i = 0; i < nums1Size; i++) {
        hashInsert(hashMap, hashLength, nums1[i], \
                   hashFind(hashMap, hashLength, nums1[i], validFlag)+1 );
    }
    
    int *result = malloc(sizeof(int)*nums2Size);
    *returnSize = 0;
    for (int j = 0; j < nums2Size; j++) {
        int findResult;
        if (findResult = hashFind(hashMap, hashLength, nums2[j], validFlag)) {
            hashInsert(hashMap, hashLength, nums2[j], findResult-1);
            result[(*returnSize)++] = nums2[j];
        }
    }
    
    return result;
} 

void hashInsert(hashnode *hashMap, int hashLength, long key, int value) {
    long newKey = (key < 0) ? -key : key;
    newKey = newKey % hashLength;
    while (1) {
        if (hashMap[newKey].key == key) {
            hashMap[newKey].value = value;
            // printf("Insertion: value: %d at newKey: %d for key: %ld\n", value, newKey, key);
            return;
        } else if (hashMap[newKey].key == -1) {
            hashMap[newKey].key = key;
            hashMap[newKey].value = value;
            // printf("Insertion: value: %d at newKey: %d for key: %ld\n", value, newKey, key);
            return;
        } else {
            if (hashMap[newKey].hh != -1) {
                newKey = hashMap[newKey].hh;
            } else {
                hashMap[newKey].hh = (newKey + 1) % hashLength;
                newKey = hashMap[newKey].hh;
            }
        }
    }
}

int hashFind(hashnode *hashMap, int hashLength, long key, int *isValid) {
    long newKey = (key < 0) ? -key : key;
    newKey = newKey % hashLength;
    *isValid = 1;
    while (1) {
        if (hashMap[newKey].key == key) {
            // printf("Found value: %d at newKey: %d for key: %ld\n", hashMap[newKey].value, newKey, key);
            return hashMap[newKey].value;
        } else {
            if (hashMap[newKey].hh == -1) {
                // printf("Found no value for key: %ld\n", key);
                *isValid = 0;
                return 0;
            } else {
                newKey = hashMap[newKey].hh;
            }
        }
    }
}

```
