---
title: "Is Graph Bipartite?"
categories:
  - Killing Camp
tags:
  - Coding test
  - Python
  - LeetCode
toc: true
toc_sticky: true
toc_label: "Is Graph Bipartite?"
toc_icon: "book"
---

## 문제 파악
주어진 그래프가 이분 그래프인지를 판별하는 문제이다. 이분 그래프는 모든 정점을 두 그룹으로 나눌 수 있는 그래프로, 서로 인접하지 않아야한다.

[Is Graph Bipartite? - LeetCode](https://leetcode.com/problems/is-graph-bipartite/)

## 접근 방법
너비 우선 탐색(BFS)를 사용해서 인접 정점들을 서로 다른색으로 칠하면서 이분 그래프를 판별 가능

1. 각 정점의 색을 칠하는 리스트 생성
2. 모든 정점에 대해 BFS 수행
3. BFS를 수행하면서 인접한 정점 방문하고, 현재와 다른 색으로 지정
4. 만약 이미 이미 색이 지정된 정점을 방문하면서 색이 다르지 않은 경우, 이분 그래프가 아니므로 False 반환
5. 모든 정점을 방문하면 이분 그래프 이므로 True 반환

## 코드 구현

```python
from collections import deque

class Solution:
    def isBipartite(self, graph: List[List[int]]) -> bool:
        # 각 정점의 색을 저장할 리스트
        # 0: 아직 색이 칠해지지 않음
        # 1: 그룹 1
        # -1: 그룹 2
        color = [0] * len(graph)
        
        def bfs(start_v):
            q = deque()
            q.append(start_v)
            color[start_v] = 1
            
            while q:
                cur_v = q.popleft()
                for next_v in graph[cur_v]:
                    if color[next_v] == 0:
                        color[next_v] = color[cur_v] * -1
                        q.append(next_v)
                    elif color[next_v] == color[cur_v]:
                        return False
            return True
        
        # 완전탐색 수행
        for start_v in range(len(graph)):
            if color[start_v] == 0:
                if not bfs(start_v):
                    return False
        return True

```

## 배우게 된 점
문제를 좀 더 꼼꼼히 읽어보자…. 그래프가 연결되어 있지 않은 경우가 있다고 문제에 명시되어 있었는데 이를 고려 안 해서 코드가 제대로 동작하지 않았다. 모든 정점을 방문하면서 BFS를 수행하니까 그래프가 연결되어 있지 않은 경우에도 올바르게 이분 그래프를 판별할 수 있었다.
