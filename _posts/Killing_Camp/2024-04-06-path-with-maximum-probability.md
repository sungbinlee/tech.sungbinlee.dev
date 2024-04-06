---
title: "Path with Maximum Probability"
categories:
  - Killing Camp
tags:
  - Coding test
  - Python
  - LeetCode
  - Priority Queue
  - Heap
  - Dijkstra
toc: true
toc_sticky: true
toc_label: "Path with Maximum Probability"
toc_icon: "book"
---

## 문제 파악
주어진 그래프에서 시작 노드부터 도착 노드까지 이동할 때 최대 확률을 계산하는 문제이며 주어진 간선의 가중치는 간선을 따라 이동할 때 성공 확률을 나타낸다.

[](https://leetcode.com/problems/path-with-maximum-probability/description/)


## 접근 방법
그래프를 생성하고, 다익스트라 알고리즘을 사용하여 시작 노드부터 도착 노드까지의 최대확률을 찾는다.

- 우선순위 큐를 사용하여 현재 노드까지의 최대확률을 갱신한다.
- 다음 노드로 이동할 때 이전 확률에 현재 간선의 확률을 곱하여 새로운 확률을 계산한다.
- 도착 노드에 도달하면 해당 노드까지의 최대 확률을 반환한다.

## 코드 구현

```python
from collections import defaultdict
from heapq import heappush, heappop

class Solution:
    def maxProbability(self, n: int, edges: List[List[int]], succProb: List[float], start_node: int, end_node: int) -> float:
        # 그래프 생성
        graph = defaultdict(list)
        for i in range(len(edges)):
            graph[edges[i][0]].append((edges[i][1], succProb[i]))
            graph[edges[i][1]].append((edges[i][0], succProb[i]))

        probabilities = [0] * n
        pq = []
        heappush(pq, (-1, start_node))
        
        while pq:
            cur_prob, cur_node = heappop(pq)
            cur_prob *= -1
            
            if probabilities[cur_node] >= cur_prob:
                continue
                
            probabilities[cur_node] = cur_prob
            
            if cur_node == end_node:
                return cur_prob
            
            for next_node, prob in graph[cur_node]:
                next_prob = cur_prob * prob
                if next_prob > probabilities[next_node]:
                    heappush(pq, (-next_prob, next_node))
                    
        return 0
    
```
