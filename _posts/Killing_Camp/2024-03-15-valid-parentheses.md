---
title: "Valid Parentheses"
categories:
  - Killing Camp
tags:
  - Coding test
  - Python
  - LeetCode
  - 스택
toc: true
toc_sticky: true
toc_label: "Valid Parentheses"
toc_icon: "book"
---

## 문제 파악
주어진 문자열이 유효한 괄호 문자열인지를 판별하는 문제이다. 주어진 문자열은 여는 괄호 '(', '{', '[' 와 닫는 괄호 ')', '}', ']' 로만 이루어져 있으며, 괄호의 쌍이 올바르게 맞아야 한다.

[Valid Parentheses - LeetCode](https://leetcode.com/problems/valid-parentheses/)

## 접근 방법

1. 문자열을 순회하면서 각 문자를 검사
2. 여는 괄호 '(','{','['를 만나면 해당하는 닫는 괄호를 스택에 추가
3. 닫는 괄호 ')'','}',']'를 만나면 스택에서 최상위 요소를 빼내어 해당 괄호와 짝이 맞는지 확인
4. 만약 스택이 비어있거나 짝이 맞지 않는 경우, 유효한 괄호 문자열이 아니므로 False를 반환
5. 문자열을 모두 순회한 후에 스택이 비어있으면 모든 괄호가 짝을 이루었으므로 유효한 괄호 문자열이라고 판단하고 True를 반환

## 코드 구현

```python
class Solution:
    def isValid(self, s: str) -> bool:
        stack = []
        
        for p in s:
            if p == "(":
                stack.append(")")
            elif p == "{":
                stack.append("}")
            elif p == "[":
                stack.append("]")
            elif not stack or stack.pop() != p:
                return False
            
        return not stack
```
