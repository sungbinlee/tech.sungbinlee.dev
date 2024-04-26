---
title: "Word Search"
categories:
  - Algorithms
tags:
  - Coding test
  - Python
toc: true
toc_sticky: true
toc_label: "Word Search"
toc_icon: "book"
---

## 문제 파악
2차원 보드에서 단어를 찾는 문제이다. 보드에서 단어는 수직 또는 수평으로 인접한 문자들로 이루어져있어야 한다.

[Word Search - LeetCode](https://leetcode.com/problems/word-search/)

## 접근 방법

주어진 보드에서 모든셀을 시작점으로 선택하고, 각 셀에서부터 단어가 존재하는지 백트래킹 기법을 통해 확인한다.

1. 보드의 모든 셀을 시작점으로 선택하여 DFS를 수행
2. 현재 위치에서 상하좌우로 이동하면서 단어의 다음 문자와 일치하는지 확인
3. 만약 현재 위치의 문자가 단어의 문자와 일치하면, 해당 위치를 방문했다고 표시하고, 다음 문자를 찾기위해 재귀적으로 DFS를 호출.
4. 단어의 모든 문자를 찾았을 경우, True를 반환
5. 만약 현재 위치에서 단어를 못찾았을 경우, 이전단계로 돌아가 다른경로를 탐색
6. 모든 셀에서 DFS 를 수행하여 단어를 찾지 못했을 경우에는 False를 반환

## 코드 구현

```python
class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        m, n = len(board), len(board[0])
        
        dx = [1, 0, -1, 0]
        dy = [0, -1, 0, 1]
        
        def backtrack(i, j, start):
            if start == len(word):
                return True
            
            if i < 0 or i >= m or j < 0 or j >= n or board[i][j] != word[start]:
                return False
            
            tmp, board[i][j] = board[i][j], '#'
            
            for d in range(4):
                if backtrack(i + dx[d], j + dy[d], start + 1):
                    return True
                
            board[i][j] = tmp
            return False
        
        for i in range(m):
            for j in range(n):
                if backtrack(i, j, 0):
                    return True
                
        return False
```

## 배우게 된 점

- 재귀 함수를 이용하여 DFS를 구현할 때, 각 단계에서 상태를 저장하고 되돌리는 것이 중요하다.
