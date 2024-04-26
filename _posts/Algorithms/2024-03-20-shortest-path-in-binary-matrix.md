---
title: "Shortest Path in Binary Matrix"
categories:
  - Algorithms
tags:
  - Coding test
  - Python
  - LeetCode
  - Graph
  - BFS
toc: true
toc_sticky: true
toc_label: "Shortest Path in Binary Matrix"
toc_icon: "book"
---

## 문제 파악
주어진 이차원 그리드에서 시작점(0,0) 에서 결승점(n-1, n-1)까지 이동할 때, 최단 경로의 길이를 구하는 문제이다.

[Shortest Path in Binary Matrix - LeetCode](https://leetcode.com/problems/shortest-path-in-binary-matrix/description/)


## 접근 방법

시작점에서부터 BFS를 수행하여 도착점까지의 최단 경로를 탐색

1. 시작점이나 도착점이 벽으로 막혀있으면 바로 -1 반환한다. 아니면 시작점과 경로길이를 큐에 추가 한다. (0, 0, 1)
2. BFS 수행: 
    - 큐에서 셀을 꺼낸다
    - 해당 셀에서 이동할 수 있는 모든 방향(상하좌우 및 대각선) 탐색한다.
    - 이동할 수 있는 셀과 경로길이(+1)를 큐에 추가하고, 해당 셀을 방문 처리 한다.
    - 도착점에 도달하면 현재까지의 경로 길이를 반환한다.

## 코드 구현

```python
from collections import deque

class Solution:
    def shortestPathBinaryMatrix(self, grid: List[List[int]]) -> int:
        shortest_path = -1
        rows = len(grid)
        cols = len(grid[0])
        visited = [[False] * cols for _ in range(rows)]
        q = deque()
        q.append((0, 0, 1))
        visited[0][0] = True
        
        if grid[0][0] == 1 or grid[-1][-1] == 1:
            return shortest_path
        
        delta = [(1, 0), (-1, 0), (0, 1), (0, -1),
                (1, 1), (-1, 1), (1, -1), (-1, -1)]
        
        while q:
            cur_x, cur_y, cur_len = q.popleft()
            
            if cur_x == rows - 1 and cur_y == cols - 1:
                shortest_path = cur_len
                break
                
            for dx, dy in delta:
                next_x = cur_x + dx
                next_y = cur_y + dy
                if next_x >= 0 and next_x < rows and next_y >= 0 and next_y < cols:
                    if grid[next_x][next_y] == 0 and not visited[next_x][next_y]:
                        q.append((next_x, next_y, cur_len + 1))
                        visited[next_x][next_y] = True
                
        
        return shortest_path
```
