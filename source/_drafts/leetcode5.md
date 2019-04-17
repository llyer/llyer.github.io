---
title: 测试草稿文章
tags:
---


腾讯面试题：
https://www.nowcoder.com/questionTerminal/28c1dc06bc9b4afd957b01acdf046e69

删除一个字符串中的几个元素，


本题目的要求是删除删除一个字符串中的几个元素之后，剩下的字符串是倒序的回文字符串。


子串：必须是连续的

子序列：可以不连续

# 区分子串和子序列

给定 "pwwkew" ，

子串是pww,wwk等很多个子串 是连在一起的

子序列是 pwk,pke等很多个子序列 ，但是子序列中的字符在字符串中不一定是连在一起的。 




难度： medium+

最长回文子串：
https://leetcode-cn.com/problems/longest-palindromic-substring/solution/


求一个字符串中的最长回文子序列，不能使用上面的倒序求公共子串或者子序列的方法，因为那样可能会产生错误

例如：abacdfgdcaba


在上面那道腾讯面试题中，删除 fg 就生成了回文子串


在本道题中：倒序后比较出来的子序列是 abacd 这个答案是错误的




最长公共子序列：
https://zh.wikipedia.org/wiki/%E6%9C%80%E9%95%BF%E5%85%AC%E5%85%B1%E5%AD%90%E5%BA%8F%E5%88%97
