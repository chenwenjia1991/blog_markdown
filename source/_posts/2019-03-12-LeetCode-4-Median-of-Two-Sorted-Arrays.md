---
title: LeetCode 4.Median of Two Sorted Arrays
date: 2019-03-12 11:08:23
categories:
- LeetCode
tags:
- LeetCode
- C/C++
- Algorithm
- BinarySearch
---

该题目选自 LeetCode 算法题目第四题 [4.Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)。

<!--more-->
## <span id='1'> 问题描述 </span> ##

### 问题原文 ###
There are two sorted arrays **nums1** and **nums2** of size m and n respectively.
Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).
You may assume **nums1** and **nums2** cannot be both empty.

### 题目大意 ###
两个排序好的数组 *nums1* 和 *nums* 长度分布为 *m* 和 *n*。
找到两个排序数组的中位数。运行时间复杂度应为 *O(log (m+n))*。
假设 *nums1* 和 *nums2* 均不为空。

### 测试用例 ###
Example 1
{% codeblock lang:yaml %}
nums1 = [1, 3]
nums2 = [2]

The median is 2.0
{% endcodeblock %}

Example 2
{% codeblock lang:yaml %}
nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5
{% endcodeblock %}

## 解决思路 ##
对算法题目，依据解题思路与复杂度分析两部分说明。
从题目的时间复杂度要求来看，明显的有了二分查找的提示。在不考虑该要求的前提下，可以通过归并排序的方式找其中的中位数，其时间复杂度为 *O(m+n)*，空间复杂度为 *O(1)*

在二分查找的思路下，找到中位数。
时间复杂度为 *O(log(min(m, n)))*，空间复杂度为 *O(1)*。

## 可执行代码 ##
### 归并排序思路 ###
在 LeetCode 中执行，runtime 击败了 97.31% 的 CPP 提交结果。
{% blockquote %}
2084 / 2084 test cases passed.
Status: **Accepted**
Runtime: **40 ms**
Memory Usage: **21.5 MB**
{% endblockquote %}

源代码如下
{% codeblock lang:cpp %}
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        if (nums1.size() == 0) {
            if (nums2.size() < 1)
                return 0;
            if (nums2.size() % 2 == 1)
                return (double)nums2[nums2.size() / 2];
            return (nums2[nums2.size() / 2] + nums2[nums2.size() / 2 - 1]) * 0.5;
        }
        if (nums2.size() == 0) {
            if (nums1.size() % 2 == 1)
                return (double)nums1[nums1.size() / 2];
            return (nums1[nums1.size() / 2] + nums1[nums1.size() / 2 - 1]) * 0.5;
        }

        int res[] = {0, 0};
        int curIndex = 0, midNum = (nums1.size() + nums2.size() + 1) / 2;
        int i = 0, j = 0;
        while(curIndex <= midNum) {
            if (nums1[i] < nums2[j]) {
                res[curIndex % 2] = nums1[i];
                i++;
            } else {
                res[curIndex % 2] = nums2[j];
                j++;
            }
            curIndex += 1;
            if (i == nums1.size()) {
                while(curIndex <= midNum) {
                    res[curIndex % 2] = nums2[j];
                    j++;
                    curIndex++;
                }// while
            }// if
            if (j == nums2.size()) {
                while(curIndex <= midNum) {
                    res[curIndex % 2] = nums1[i];
                    i++;
                    curIndex++;
                }// while
            }// if
        }// while
        if ((nums1.size() + nums2.size()) % 2 == 1)
            return (double)(min(res[0], res[1]));
        return (res[0] + res[1]) * 0.5;
    }// findMedianSortedArrays
};
{% endcodeblock %}

### 二分查找思路 ###
On the way！

## 总结 ##
本题在思路明确的基础上仍需注意边界的问题，主要包括以下几个方面：
1. 输入的合法性检验，如输入长度为 0 或者为 1 的字符串，可以直接返回结果；
2. 字符串哈希表大小，确定字符种类范围。此处经过提交测试发现，测试集中基本包含 ASCII 的字符，使用 128 空间大小的哈希表可以通过测试；
3. 子串起始点的判断及子串长度的计算。在出现两个字符时，若字符一样，长度是两者索引的差；不相同时，是两者索引的差值加一。 

使用哈希表加速计算在实际工程中是较为常见的策略之一，值得借鉴和学习。
