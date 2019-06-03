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

该题目选自 LeetCode 算法题目第四题 [4.Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)，LeetCode 中第一道难度为 Hard 的题目。在没有时间复杂度要求的前提下，归并排序得到需要的解是简单有效的实现方式。进一步优化，通过找第 K 大数的方式每次减少 K/2 的搜索范围，可以在 log(m+n) 的时间复杂度下解决。最终的优化方案是找可以将两个数组分为两部分的边界值，复杂度可以优化到 log(min(m, n))。详细解题思路及源代码如下。

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

二分查找在此处容易想到的是在两个排好序的序列中找到第 *K* 个大的数。该方案的复杂度为 *O(log K)* 的，由此处为中位数即时间复杂度为 *O(log(m+n))*。
在两个有序数组中查找第 *K* 大的数的伪代码如下：
{% blockquote %}
对有序数组 *A*，*B*，查找其第 *K* 大的数字；
1. 若其中数组 *A* 为空，*B[K-1]* 即为所寻值；
2. 若其中数组 *B* 为空，*A[k-1]* 即为所寻值；
3. 若 *K=1* 返回 *Min(A[0], B[0])*；
4. 取中间索引 *midA = Min(len(A)-1, K/2-1)*，*midB = Mind(len(B)-1, K/2)*，
    - 若 *A[midA] < B[midB]*，在数组 *A[midA+1:end]*, *B* 中找第 *K-midA-1* 大的数；
    - 若 *A[midA] < B[midB]*，在数组 *A*, *B[midB+1:end]* 中找第 *K-midB-1* 大的数；
    - 若 *A[midA] = B[midB]*，即为所求数值。
{% endblockquote %}

基于上述寻找第 *K* 大的数方法继续延伸探究，我们发现该题目本质是找可以将数组 *A* 和 数组 *B* 可以划分为左右两个集合 *MinSet* 和 *MaxSet*，其 *Max(MinSet) <= Min(MaxSet)*，即寻求对两个数组切分为两个数量相等的集合（若为奇数个元素，则中间元素在左右两个集合均出现）。对于数组 *A* 集合索引 *indexL* 之前的元素均属于 *MinSet* 集合的话，对于 *B* 集合，则为 *K/2-indexL* 之前的集合属于 *MinSet*，即寻找该集合的切分索引 *indexL*。此处我们可以通过二分的方法查找，因此时间复杂度变为 *O(log(len(A)))*。假定数组 *A* 是长度较少的数组（若不是，可以交换 *A*、*B* 数组）。这样，时间复杂度即为 *O(log(min(m, n)))*，空间复杂度为 *O(1)*。伪代码如下：
{% blockquote %}
对有序数组 *A*，*B*，查找其划分左右集合的均值；
1. 判断 *A*，*B* 长度，若数组 *A* 长度大于数组 *B* 长度，*A*、*B* 数组进行交换；
2. 初始化索引切分范围 *lo = 0*，*hi = 2 * len(A)*;
3. 计算数组 *A* 的切分索引 *indexA = lo + (hi - lo)/2*，则 *indexB = len(A) + len(B) - indexA*，其切分左右边界为 *L1 = nums1[(indexA-1)/2]*，*R1=nums1[(indexA)/2]*；
4. 同理，计算得到数组 *B* 的 *L2*，*R2*：
    - 若 *L1 > R2*，更新 *lo = mid2 + 1*；
    - 若 *L2 > R1*，更新 *hi = mid2 - 1*；
    - 否则，即为所有分界线，返回 *(max(L1,L2) + min(R1, R2)) / 2.0*。
{% endblockquote %}

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
        int len1 = nums1.size(), len2 = nums2.size();
        if (len1 < 1 && len2 < 1) return -1;
        if (len1 < 1) return (nums2[len2/2] + nums2[(len2-1)/2]) * 0.5;
        if (len2 < 1) return (nums1[len1/2] + nums1[(len1-1)/2]) * 0.5;
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
一般而言，对排序好的数组查找指定元素大都是二分查找，此处主要是如何将查找中间数的问题变为二分查找。

源代码如下
{% codeblock lang:cpp %}
class Solution {
public:
    double findMedianSortedArrays_findKth(vector<int>& nums1, vector<int>& nums2) {
        if (nums1.empty() && nums2.empty())
            return -1;
        int len1 = nums1.size(), len2 = nums2.size();
        double resultMid = this->findKth2(nums1, 0, nums2, 0, (len1 + len2 + 1) / 2);
        if ((len1 + len2) % 2 == 1) {
            return resultMid;
        }
        double resultNext = this->findKth2(nums1, 0, nums2, 0, (len1 + len2 + 2) / 2);
        return (resultMid + resultNext) / 2;
    }// findMedianSortedArrays_findKth

private:
    int findKth1(vector<int> nums1, vector<int> nums2, int k) {
        if (nums1.empty()) return nums2[k - 1];
        if (nums2.empty()) return nums1[k - 1];
        if (k == 1) return min(nums1[0], nums2[0]);
        int i = min((int) nums1.size(), k / 2), j = min((int) nums2.size(), k / 2);
        if (nums1[i - 1] > nums2[j - 1]) {
            return this->findKth1(nums1, vector<int>(nums2.begin() + j, nums2.end()), k - j);
        }
        return this->findKth1(vector<int>(nums1.begin() + i, nums1.end()), nums2, k - i);
    }// findKth1

    int findKth2(vector<int>& nums1, int i, vector<int>& nums2, int j, int k) {
        if (i >= nums1.size()) return nums2[j + k - 1];
        if (j >= nums2.size()) return nums1[i + k - 1];
        if (k == 1) return min(nums1[i], nums2[j]);
        int midVal1 = (i + k / 2 - 1 < nums1.size()) ? nums1[i + k / 2 - 1] : INT_MAX;
        int midVal2 = (j + k / 2 - 1 < nums2.size()) ? nums2[j + k / 2 - 1] : INT_MAX;
        if (midVal1 < midVal2) {
            return this->findKth2(nums1, i + k / 2, nums2, j, k - k / 2);
        }
        return this->findKth2(nums1, i, nums2, j + k / 2, k - k / 2);
    } //findKth2
};
{% endcodeblock %}


优化变种
源代码如下
{% codeblock lang:cpp %}
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int len1 = nums1.size(), len2 = nums2.size();
        if (len1 < len2)
            return this->findMedianSortedArrays(nums2, nums1);
        int lo = 0, hi = len2 * 2;
        while (lo <= hi) {
            int mid2 = (lo + hi) / 2;   // Try Cut 2
            int mid1 = len1 + len2 - mid2;  // Calculate Cut 1 accordingly
            // Get L1, R1, L2, R2 respectively
            double L1 = (mid1 == 0) ? INT_MIN : nums1[(mid1-1)/2];
            double L2 = (mid2 == 0) ? INT_MIN : nums2[(mid2-1)/2];
            double R1 = (mid1 == len1 * 2) ? INT_MAX : nums1[(mid1)/2];
            double R2 = (mid2 == len2 * 2) ? INT_MAX : nums2[(mid2)/2];
            // A1's lower half is too big; need to move C1 left (C2 right)
            if (L1 > R2) lo = mid2 + 1;
            // A2's lower half too big; need to move C2 left.
            else if (L2 > R1) hi = mid2 - 1;
            // Otherwise, that's the right cut.
            else return (max(L1,L2) + min(R1, R2)) / 2;
        }// while
        return -1;
    }// findMedianSortedArrays
};// Solution
{% endcodeblock %}    

## 总结 ##
本题在思路明确的基础上仍需注意边界的问题，主要包括以下几个方面：
1. 两个数组为空的校验，即两个都为空的情况，其中一个为空的情况，各对应返回要求值；
2. 对于切分值 *L1*，*R1*，*L2*，*R2* 四个值比较情况的梳理及确定；
3. 时间复杂度与空间复杂度的分析。


