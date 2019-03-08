---
title: LeetCode 1.Two Sum
date: 2019-03-08 17:14:21
categories:
- LeetCode
tags:
- LeetCode
- C/C++
- Algorithm
---

LeetCode 网站题目多为一些经典公司面试应聘者的题目，大多数程序员都会通过刷题来应聘一些互联网公司。毕业之后，从 C/C++ 转为了 Python 开发，对 C/C++ 的很多基本用法有所遗忘，期望通过一段时间的刷题巩固寻求做编程题的感觉。通过博客形式的分享与记录鞭策自己不断前行，愿在技术的道路上可以走的更远。

该题目选自 LeetCode 算法题目第一题 [1.Two Sum](https://leetcode.com/problems/two-sum/)。

<!--more-->
## <span id='1'> 问题描述 </span> ##

### 问题原文 ###
Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.
You may assume that each input would have **exactly** one solution, and you may not use the same element twice.

### 题目大意 ###
给定整数数组，若存在两个数之和等于期望的目标值，返回这两个数的索引。
这里假设每一个输入只有一个解且同一个数不能使用两次。

### 测试用例 ###
这里未定义异常数据或不满足条件的数据返回情况，自定义选为两个 [-1, -1] 的索引，表示其不存在。

Example
{% codeblock lang:yaml %}
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
{% endcodeblock %}

## 解决思路 ##
对算法题目，依据解题思路与复杂度分析两部分说明。

### Brute Force ###
对程序员而言，碰到一个问题最初的解决方案一般是暴力解决。
即通过两层循环，依次判断两个数的组合是否满足要求。
伪代码如下：
{% blockquote %}
1. 从数组中取出未检查过的数 *x*；
2. 遍历剩余数字：
    * 若存在 *target - x*，返回两个数的索引值；
    * 重复步骤 1 直至数组中不存在未检查过的数字，执行 3；
3. 重复上述过程若无解，返回 [-1, -1]。
{% endblockquote %}

该算法的时间复杂度为 *O(n^2)*，空间复杂度为 *O(1)*。

### Hash Table ###
对上述解法而言，我们可以在查找目标数字是否存在时，可以通过建立 Hash Table 的方式搜索加速其检索的过程。
伪代码如下：
{% blockquote %}
1. 初始化 Hash Table *hash_table*
2. 从数组中取出未检查过的数 *x* 及其对应索引 *index*；
3. 查询 *hash_table* 是否存在目标值 *target - x*：
    * 若存在 *target - x*，返回其对应索引值；
    * 若不存在，将 *x* 及其对应索引值 *index* 存入 *hash_table* 中，执行 2 直至数组中不存在未检查的数字，执行 4；
4. 重复上述过程若无解，返回 [-1, -1]。
{% endblockquote %}

该算法优化了检索目标值存在的过程时间复杂度为 *O(1)*，因此整体时间复杂度为 *O(n)*，空间复杂度为 *O(n)*。

## 可执行代码 ##
### Brute Force ###
在 LeetCode 中执行，runtime 击败了 36.77% 的 CPP 提交结果。
{% blockquote %}
29 / 29 test cases passed.
Status: **Accepted**
Runtime: **136 ms**
Memory Usage: **9.4 MB**
{% endblockquote %}

源代码如下
{% codeblock lang:cpp %}
class Solution {
public:
    vector<int> twoSum_BruteForce(vector<int>& nums, int target) {
        for (int i = 0; i < nums.size(); i++)
            for (int j = i + 1; j < nums.size(); j++) {
                if (nums[i] == (target - nums[j]))
                    return vector<int>{i, j};
            }
        return vector<int>{-1, -1};
    }// twoSum_BruteForce
};
{% endcodeblock %}

### Hash Table ###
在 LeetCode 中执行，runtime 击败了 94.91% 的 CPP 提交结果。
{% blockquote %}
29 / 29 test cases passed.
Status: **Accepted**
Runtime: **12 ms**
Memory Usage: **10.3 MB**
{% endblockquote %}

源代码如下
{% codeblock lang:cpp %}
class Solution {
public:
    vector<int> twoSum_HashTable(vector<int>& nums, int target) {
        if (nums.size() < 2)
            return vector<int>{-1, -1};
        map<int, int> map_value_index;
        for (int i = 0; i < nums.size(); i++) {
            map<int, int>::iterator find_iter = map_value_index.find(target-nums[i]);
            if (find_iter != map_value_index.end()) {
                // Find the two numbers
                int index = find_iter->second;
                if (i > index)
                    return vector<int>{index, i};
                return vector<int>{i, index};
            }// if
            else {
                map_value_index[nums[i]] = i;
            }// else
        }// for
        return vector<int>{-1, -1};
    }// twoSum_HashTable
};
{% endcodeblock %}

## 总结 ##
On the way！