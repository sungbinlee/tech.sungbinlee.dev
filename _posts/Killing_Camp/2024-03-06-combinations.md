---
title: "Combinations"
categories:
  - Killing Camp
tags:
  - Coding test
  - Python
toc: true
toc_sticky: true
toc_label: "Combinations"
toc_icon: "book"
---

[Combinations - LeetCode](https://leetcode.com/problems/combinations/)

## 문제 파악

주어진 범위 내에서 숫자를 조합하여 주어진 길이의 조합을 찾는 문제

## 접근 방법

백트래킹을 통해 재귀적으로 조합을 찾기

1. `curr` 의 길이가 주어진 길이 `k` 와 같아지면, 해당 조합을 결과 리스트에 추가후 종료
2. 그렇지 않다면, `start` 부터 `n` 까지의 숫자를 손회하면서 현재 조합에 해당 숫자가 없으면 추가후, 재귀적으로 다음 숫자를 탐색
3. 재귀 호출이 끝나면 해당 숫자들을 제거

## 코드 구현

```python
class Solution:
    def combine(self, n: int, k: int) -> List[List[int]]:
        def backtrack(start, curr):
            # base case
            if len(curr) == k:
                ans.append(curr[:])
                return
            
            for num in range(start, n + 1):
                if num not in curr:
                    curr.append(num)
                    backtrack(num + 1, curr)
                    curr.pop()
            
        ans = []
        backtrack(1, [])
        return ans
```

## 배우게 된 점

새로운 점을 배웠다면, 백트래킹을 응용해서 문제 해결 방법을 익혔다. 이러한 방식은 순열, 조합 뿐만 아니라 다양한 조합관련 문제를 해결하는데 유용.
