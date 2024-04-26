---
title: "Network Delay Time"
categories:
  - Algorithms
tags:
  - Coding test
  - Python
  - LeetCode
  - Graph
  - Dijkstra
toc: true
toc_sticky: true
toc_label: "Network Delay Time"
toc_icon: "book"
---

## 문제 파악
네트워크 상에서 한 지점에서 출발하여 다른 모든 노드까지 도달하는 최소 시간을 계산하는 문제이다.

[Network Delay Time - LeetCode](https://leetcode.com/problems/network-delay-time/description/)

## 접근 방법

1. 주어진 입력을 기반으로 그래프를 생성 한다.
2. 다익스트라 알고리즘을 사용해서 각 출발점을 기준으로 각 노드마다 도달하는 시간을 계산한다. 
3. 도달할 수 없는 노드를 확인한다. (-1 반환)
4. 최단 소요시간 중 가장 큰 값을 반환한다.

## 코드 구현

```python
from collections import defaultdict
from heapq import heappush, heappop

class Solution:
    def networkDelayTime(self, times: List[List[int]], n: int, k: int) -> int:
        # 그래프 구현
        graph = defaultdict(list)
        for time in times:
            graph[time[0]].append((time[2], time[1]))
            
        costs = {}
        pq = []
        heappush(pq, (0, k))

        # 다익스트라 구현
        while pq:
            cur_cost, cur_node = heappop(pq)
            if cur_node not in costs:
                costs[cur_node] = cur_cost
                for cost, next_node in graph[cur_node]:
                    next_cost = cur_cost + cost
                    heappush(pq, (next_cost, next_node))
            
        # 조건 확인
        for i in range(1, n+1):
            if i not in costs:
                return -1
        
        # 최대값 반환
        return max(costs.values())
```

## 배우게 된 점

- 우선순위 큐를 활용하여 다익스트라 알고리즘을 효율적으로 구현하는 방법을 배웠다.
