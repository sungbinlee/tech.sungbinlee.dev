---
title: "Longest Valid Parentheses"
categories:
  - Algorithms
tags:
  - Coding test
  - Python
  - LeetCode
  - Stack
toc: true
toc_sticky: true
toc_label: "Longest Valid Parentheses"
toc_icon: "book"
---

## 문제 파악
주어진 문자열에서 가장 긴 유효한 괄호 문자열의 길이를 찾는 문제

[Longest Valid Parentheses - LeetCode](https://leetcode.com/problems/longest-valid-parentheses/)


## 접근 방법
스택을 이용하여 유효한 괄호 문자열의 길이를 계산

1. 주어진 문자열을 순회하면서 열린괄호 ‘(’를 만나면 스택에 0을 추가
2. 닫힌 괄호 ‘)’를 만나면 스택에서 값을 꺼내어 짝을 이루는 열린 괄호의 개수를 누적하여 저장
    - 스택의 길이가 1 이상인 경우에만 짝이 있으므로 해당 경우에만 계산
3. 계산된 값중 가장 긴값을 저장하여 반환

## 코드 구현

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        stack = [0]
        longest = 0
        
        for c in s:
            if c == "(":
                stack.append(0)
            else:
                if len(stack) > 1:
                    val = stack.pop()
                    stack[-1] += val + 2
                    longest = max(longest, stack[-1])
                else:
                    stack = [0]
                    
        return longest

```
