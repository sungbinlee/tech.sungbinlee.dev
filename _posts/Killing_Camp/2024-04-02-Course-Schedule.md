---
title: "Course Schedule"
categories:
  - Killing Camp
tags:
  - Coding test
  - Python
  - LeetCode
  - Topological Sort
  - Graph
toc: true
toc_sticky: true
toc_label: "Course Schedule"
toc_icon: "book"
---
## 문제 파악
수강 과목의 선수과목이 주어졌을때, 모든 과목을 수강할 수 있는지 여부를 판단하는 문제이다.

[LeetCode - The World's Leading Online Programming Learning Platform](https://leetcode.com/problems/course-schedule/description/)

## 접근 방법

위상 정렬 알고리즘을 사용하여 해결할 수 있다. 

1. 주어진 선수과목 정보를 바탕으로 각 과목의 선수 과목들을 담은 그래프와 진입 치수를 저장한다.
2. 선행 과목이 없는 과목들(진입 차수가 0인)들을 큐에 넣는다.
3. 큐에서 과목을 빼면서 해당 과목의 진입 차수를 감소시키고, 진입 차수가 0인 과목들을 큐에 추가한다.
4. 위 과정을 마친 후, 방문 과목의 수가 총 과목 수 와 동일 하면 모든과목을 수강할수 있으므로  True  반환, 아니면 False 를 반환한다.

## 코드 구현

```python
from collections import deque

class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        visited = []
        graph = [[] for _ in range(numCourses)]
        indegree = [0] * numCourses
        
        for v, u in prerequisites:
            graph[u].append(v)
            indegree[v] += 1
        
        print(graph, indegree)
        q = deque()
        
        for v in range(numCourses):
            if indegree[v] == 0:
                q.append(v)
        
        while q:
            cur_v = q.popleft()
            visited.append(cur_v)
            
            for next_v in graph[cur_v]:
                indegree[next_v] -= 1
                
                if indegree[next_v] == 0:
                    q.append(next_v)
        
        if len(visited) == numCourses:
            return True
        
        return False
            
            
            
```

## 배우게 된 점

- 위상 정렬 알고리즘의 활용에 대해 배웠다.
    - 그래프의 선후 관계가 있는 작업을 효율적으로 정렬하고, 사이클이 없는 경우에 한해 적용할 수 있다.
    - 그래프의 각 정점의 진입 차수(특정 정점으로 들어오는 간선의 수)를 기준으로 동작한다. 진입 차수가 0인 정점부터 차례대로 방문하면서 해당 정점과 연결된 간선을 제거하고, 이로 인해 진입 차수가 0이 된 새로운 정점들을 큐에 추가하는 방식으로 작동한다. 이러한 과정을 반복하여 그래프의 모든 정점을 방문하면서 정렬된 순서를 얻을 수 있다.
