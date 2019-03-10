---
title: LeetCode 2.Add Two Numbers
date: 2019-03-10 15:50:00
categories:
- LeetCode
tags:
- LeetCode
- C/C++
- Algorithm
---

该题目选自 LeetCode 算法题目第二题 [2.Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)。与第一题相比，该题目更偏重对数据结构的熟练应用。当然为方便实现，这里的链表是以逆序的方式存储的。

<!--more-->
## <span id='1'> 问题描述 </span> ##

### 问题原文 ###
You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order** and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.
You may assume the two numbers do not contain any leading zero, except the number 0 itself.

### 题目大意 ###
给定两个非空链表来表示两个非负整数。数字在链表中以逆序的方式存储，每个节点存储一位数字。将这两个数相加并返回表示结果的链表的头。
可假设这两个数字除 0 本身外，不存在前面有有多 0。

### 测试用例 ###
这里未定义异常数据或不满足条件的数据返回情况，自定义异常情况下返回 NULL 指针。

Example
{% codeblock lang:yaml %}
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
{% endcodeblock %}

## 解决思路 ##
对算法题目，依据解题思路与复杂度分析两部分说明。
该题目主要考察对链表的应用，实现即可。
假设两个链表的长度分别为 *m* 和 *n*，则时间复杂度为 *O(max(m, n)*，结果链表最长为 *O(max(m,n ) + 1)*，空间复杂度也为 *O(max(m,n ))*。

## 可执行代码 ##

在 LeetCode 中执行，runtime 击败了 96.58% 的 CPP 提交结果。
{% blockquote %}
1563 / 1563 test cases passed.
Status: **Accepted**
Runtime: **40 ms**
Memory Usage: **19 MB**
{% endblockquote %}

源代码如下
{% codeblock lang:cpp %}
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        if (l1 == NULL || l2 == NULL)
            return NULL;
        int sum = l1->val + l2->val;
        ListNode *sum_result = new ListNode(sum % 10);
        ListNode *cur_p = sum_result, *p1 = l1->next, *p2 = l2->next;
        int flag_carry = sum / 10;
        while(p1 && p2) {
            sum = p1->val + p2->val + flag_carry;
            cur_p->next = new ListNode(sum % 10);
            flag_carry = sum / 10;
            cur_p = cur_p->next;
            p1 = p1->next;
            p2 = p2->next;
        }// while
        while(p1) {
            sum = p1->val + flag_carry;
            cur_p->next = new ListNode(sum % 10);
            flag_carry = sum / 10;
            cur_p = cur_p->next;
            p1 = p1->next;
        }// while
        while(p2) {
            int sum = p2->val + flag_carry;
            cur_p->next = new ListNode(sum % 10);
            flag_carry = sum / 10;
            cur_p = cur_p->next;
            p2 = p2->next;
        }// while
        if (flag_carry > 0)
            cur_p->next = new ListNode(flag_carry);
        return sum_result;
    }// addTwoNumbers
};
{% endcodeblock %}


## 总结 ##
对该类题目，除了要对常见的数据结构熟悉及灵活应用外，一定要注意对边界条件的判断。在面试及日常工程代码中，入口处先进性输入参数的合理性检查是非常有必要的。确定对异常结果如何处理非常考验工程实践水平和逻辑的完整性。
该题目并不难理解，也容易通过，主要注意边界情况的判断即可。