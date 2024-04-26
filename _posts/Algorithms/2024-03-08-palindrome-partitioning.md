---
title: "Palindrome Partitioning"
categories:
  - Algorithms
tags:
  - Coding test
  - Python
toc: true
toc_sticky: true
toc_label: "Palindrome Partitioning"
toc_icon: "book"
---

## 문제 파악
주어진 문자열을 팰린드롬 부분 문자열(앞으로 읽으나 뒤로 읽으나 동일한 내용을 갖는)로 분할하는 문제이다.

[Palindrome Partitioning - LeetCode](https://leetcode.com/problems/palindrome-partitioning/description/)

## 접근 방법

주어진 문자열을 재귀적으로 탐색하면서, 각 위치에서 팰린드롬 부분 문자열을 찾아낸다.

1. 재귀적 백트래킹 함수 설계:
    - backtrack(start, path)의 형태로 설계 `start` : 현재 검사할 문자열의 시작 인덱스 `path` : 현재까지의 팰린드롬 부분 문자열들을 담은 리스트
    - **재귀 종료 조건**: **`start`**가 문자열의 길이와 같아지면 현재 경로에 담긴 부분 문자열들이 한 가지 분할의 결과이므로 이를 결과에 추가하고 재귀를 종료합니다.
2. **분할 후보군 탐색**:
    - **`start`**부터 문자열의 끝까지 반복하면서 모든 가능한 부분 문자열을 탐색합니다.
3. **팰린드롬인 경우 처리**:
    - 현재 검사하는 부분 문자열이 팰린드롬인 경우에만 이를 경로에 추가하고 재귀적으로 다음 부분을 탐색합니다.
4. **결과 반환**:
    - 모든 가능한 분할을 탐색한 후, 결과 리스트를 반환합니다.

## 코드 구현

```python
class Solution:
    def partition(self, s: str) -> List[List[str]]:
        def backtrack(start, path):
            # base case
            if start == len(s):
                ans.append(path[:])
                return
            
            # 분할 후보군 탐색
            for end in range(start + 1, len(s) + 1):
                if self.isPal(s[start:end]):
		                # 팰린드롬인 경우 처리
                    path.append(s[start:end]) # 현재 팰린드롬 부분을 경로에 추가
                    backtrack(end, path) # 다음 인덱스 부터 재귀적으로 탐색
                    path.pop() # 탐색이 끝난 후 현재 팰린드롬 부분을 경로에서 제거
        
        ans = []
        backtrack(0, [])
        return ans

    
    def isPal(self, s):
        return s == s[::-1]
    
```

## 배우게 된 점
