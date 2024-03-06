---
title: "Permutations"
categories:
  - Killing Camp
tags:
  - Coding test
  - Python
toc: true
toc_sticky: true
toc_label: "Two Sum"
toc_icon: "book"
---


[Permutations - LeetCode](https://leetcode.com/problems/permutations/)

## 문제 파악

주어진 숫자 리스트 nums의 순열을 생성하는 문제이다.

## 접근 방법

1. 백트래킹을 활용해서 순열 생성
2. 현재의 순열길이와 주어진 nums 의 길이가 같으면 결과 리스트에 현재 순열을 추가
3. nums의 요소를 선택하고, 선택한 요소가 순열에 없으면 추가하고 재귀적으로 백트래킹 수행
4. 재귀호출이 종료되면 선택된 요소를 순열에서 제거

## 코드 구현

```python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        def backtrack(curr):
            # base case
            if len(nums) == len(curr):
                ans.append(curr[:])
                return
            
            for num in nums:
                if num not in curr:
                    curr.append(num)
                    backtrack(curr)
                    curr.pop()
                    
        ans = []
        backtrack([])
        return ans
        
```

## 배우게 된 점

백트래킹은 완전탐색(Exhaustive Search)의 한 형태로, 가능한 모든 경우의 수를 탐색하여 해답을 찾는 알고리즘 기법이다.

- **백트래킹과 완전탐색의 차이:**  백트래킹은 유효한 조건을 만족하지 않는 후보의 탐색을 포기하여 탐색 효율을 높이는 기법이며, 완전탐색은 모든 경우의 수를 고려하여 탐색하는 방법이다.
- 백트래킹은 후보의 수가 많은 조합, 순열 문제 등에 유용하게 사용된다.
