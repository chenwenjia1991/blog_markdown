---
title: LeetCode 3.Longest Substring Without Repeating Characters
date: 2019-03-11 14:33:46
categories:
- LeetCode
tags:
- LeetCode
- C/C++
- Algorithm
---

该题目选自 LeetCode 算法题目第三题 [3.Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)。在中等难度题目中，个人认为该题目是非常适合作为面试手写代码的测试题之一。代码量少，但边界需要注意很多隐藏的问题，如参数合法性检验，字符集大小确定，子串长度计算等，细节处查考了基本功力。题目不复杂，但手写代码可以一次性通过测试还是有一定的难度和挑战。

<!--more-->
## <span id='1'> 问题描述 </span> ##

### 问题原文 ###
Given a string, find the length of the **longest substring** without repeating characters.

### 题目大意 ###
在给定字符串中找到不含重复字符的最长子串的长度。

### 测试用例 ###
Example 1
{% codeblock lang:yaml %}
Input: "abcabcbb"
Output: 3 
Explanation: The answer is "abc", with the length of 3. 
{% endcodeblock %}

Example 2
{% codeblock lang:yaml %}
Input: "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.
{% endcodeblock %}

Example 3
{% codeblock lang:yaml %}
Input: "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3. 
Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
{% endcodeblock %}

## 解决思路 ##
对算法题目，依据解题思路与复杂度分析两部分说明。
对题目而言，是一道典型空间换时间的题目。
伪代码如下：
{% blockquote %}
初始化不重复字符的子串起始位置 *start* 为 - 1，
1. 从字符串头开始遍历字符串，索引 *index* 及对应字符 *c*；
2. 检索 *c* 是否在字符哈希表中出现过：
    * 若出现，则判断其索引值是否大于 *start*，大于 *start* 则更新 *start* 值（当前位置之后的字符作为结尾字符的不重复最长子串起始位置从此开始）；
    * 未出现执行动作 3；
3. 更新索引哈希表中字符 *c* 对应的哈希值为索引 *index*；
4. 此时，以 *index* 处字符结尾的最长子串长度为 *index - start*;
5. 若该长度大于已知最长子串长度，更新最长字串长度值，继续执行 1。
{% endblockquote %}

时间复杂度为 O(n)，空间复杂度为 O(1)。

## 可执行代码 ##

在 LeetCode 中执行，runtime 击败了 97.58% 的 CPP 提交结果。
{% blockquote %}
987 / 987 test cases passed.
Status: **Accepted**
Runtime: **20 ms**
Memory Usage: **15.4 MB**
{% endblockquote %}

源代码如下
{% codeblock lang:cpp %}
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int len_s = s.size();
        if (len_s <= 1) return len_s;
        vector<int> indexHash(128, -1);
        int maxLen = 0, start = -1;
        for (int i = 0; i < len_s; i++) {
            if (indexHash[s[i]] > start)
                start = indexHash[s[i]];
            indexHash[s[i]] = i;
            int curLen = i - start;
            if (maxLen < curLen) maxLen = curLen;
        }// for
        return maxLen;
    }// lengthOfLongestSubstring
};
{% endcodeblock %}

## 总结 ##
本题在思路明确的基础上仍需注意边界的问题，主要包括以下几个方面：
1. 输入的合法性检验，如输入长度为 0 或者为 1 的字符串，可以直接返回结果；
2. 字符串哈希表大小，确定字符种类范围。此处经过提交测试发现，测试集中基本包含 ASCII 的字符，使用 128 空间大小的哈希表可以通过测试；
3. 子串起始点的判断及子串长度的计算。在出现两个字符时，若字符一样，长度是两者索引的差；不相同时，是两者索引的差值加一。 

使用哈希表加速计算在实际工程中是较为常见的策略之一，值得借鉴和学习。
