---
title: Problem --- Move Zeroes
tags:
---

https://leetcode-cn.com/problems/move-zeroes/

void moveZeroes (int *nums, int numsSize) {
	int zerosCount = 0;
	for (int i=0; i<numsSize; i++) {
		if (nums[i]) nums[zerosCount++] = nums[i];
	}
	for (; zerosCount < numsSize; zerosCount++) nums[zerosCount] = 0;
}
