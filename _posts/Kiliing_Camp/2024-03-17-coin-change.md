---
title: "Coin Change"
categories:
  - Killing Camp
tags:
  - Coding test
  - Python
  - LeetCode
  - BFS
toc: true
toc_sticky: true
toc_label: "Coin Change"
toc_icon: "book"
---
## 문제 파악
동전을 사용하여 주어진 금액을 만들 수 있는 최소 동전의 개수를 구하는 문제

[Coin Change - LeetCode](https://leetcode.com/problems/coin-change/)

## 접근 방법
BFS(너비 우선 탐색)을 사용하여 모든 가능한 동전의 조합을 탐색하여 최소 동전 개수를 찾을 수 있다.

1. 큐를 사용하여 BFS를 수행. 큐에서 상태를 하나씩 꺼내어 가능한 다음 상태를 계산하고 큐에 추가(각 상태의 깊이(사용한 동전)과 잔액을 기록
2. 큐에서 꺼낸 상태가 목표 금액을 만들었는지 확인하고, 만들었다면 탐색을 종료후 결과 반환
3. 중복된 계산을 하지않도록 잔액을 기록
4. 큐가 비어서 더이상 탐색을 못한다면 -1 반환

## 코드 구현

```python
from collections import deque

class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        answer = -1
        depth = 0
        
        visited = []
        
        q = deque()
        q.append((depth, amount))

        while q:
            curr_depth, curr_amount = q.popleft()
            if curr_amount == 0:
                answer = curr_depth
                break
            for coin in coins:
                next_amount = curr_amount - coin
                if next_amount >= 0 and next_amount not in visited:
                    q.append((curr_depth + 1, next_amount))
                    visited.append(next_amount)
        
        return answer
```

## 배우게 된 점

문제를 처음 접할때 그리디 접근을 시도 하였으나, 무작정 구현하면서 막혔다. 강의 힌트를 참고하니 그리디로 오해할수 있는 문제라고 하였다. 문제를 더 깊게 이해하기 위해서, 다양한 케이스를 대입하고 교차 검증을 통해 올바른 접근방법을 찾아내는것이 중요하다는것을 깨달았다.

BFS로 문제를 해결할 때 시간 초과와 메모리 초과 문제를 해결하는 것에 대해 배우게 되었습니다. 이를 해결하기 위해 중복 방지와 테스트케이스를 참고하여 예외를 찾는 것 이였다.
