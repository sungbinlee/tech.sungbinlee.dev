---
title: "Number of Islands"
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
toc_label: "Number of Islands"
toc_icon: "book"
---

## 문제 파악
주어진 이차원 그리드에서 섬의 개수를 찾는 문제이다(섬은 연결된 1로 이루어진 영역을 의미)

[Number of Islands - LeetCode](https://leetcode.com/problems/number-of-islands/)


## 접근 방법

BFS를 사용하여 해결할 수 있다. 주어진 그리드를 순회하면서 각 섬의 시작점을 찾고, 해당 섬을 BFS로 탐색하여 연결된 모든 지점을 방문하면서 재방문 하지 않도록 visited 배열을 활용. 이를 통해 섬의 개수를 세어 반환한다.

1. 주어진 그리드를 순회하면서 각 셀을 탐색
2. 만약 현재 셀이 땅(값: 1) 이고, 이전에 방문하지 않았다면:
    - 이 셀을 시작으로 새로운 섬을 찾은것으로 간주
    - 해당 셀을 방문처리하고 인접한 땅을 찾기위해 BFS를 통해 모든땅을 탐색
    - BFS를 통해 찾은 모든 땅을 방문 처리
    - 섬의 개수를 증가시킨 후에 새로운 섬을 탐색한다.
3. 모든 셀의 탐색이 끝나면 섬의 개수를 반환

## 코드 구현

```python
from collections import deque

class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        cnt = 0
        rows = len(grid)
        cols = len(grid[0])
        visited = [[False]*cols for _ in range(rows)]

        
        def bfs(x, y):
            dx = [-1, 1, 0, 0]
            dy = [0, 0, -1, 1]
            
            q = deque()
            q.append((x, y))
            visited[x][y] = True
            
            while q:
                cur_x, cur_y = q.popleft()
                for i in range(4):
                    next_x = cur_x + dx[i]
                    next_y = cur_y + dy[i]
                    if next_x >= 0 and next_x < rows and next_y >= 0 and next_y < cols:
                        if not visited[next_x][next_y] and grid[next_x][next_y] == "1":
                            visited[next_x][next_y] = True
                            q.append((next_x, next_y))
        
        for x in range(rows):
            for y in range(cols):
                if grid[x][y] == "1" and not visited[x][y]:
                    bfs(x, y)
                    cnt += 1
        
        return cnt
```

## 배우게 된 점
암시적그래프에도 BFS를 적용할수 있게 되었다. 각 셀은 그래프의 정점이고 셀 사이의 인접관계는 간선이다.
