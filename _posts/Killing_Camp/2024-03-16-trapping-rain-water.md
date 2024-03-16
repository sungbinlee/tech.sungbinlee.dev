---
title: "Trapping Rain Water"
categories:
  - Killing Camp
tags:
  - Coding test
  - Python
  - LeetCode
  - 스택
toc: true
toc_sticky: true
toc_label: "Trapping Rain Water"
toc_icon: "book"
---

## 문제 파악
주어진 높이 리스트로 형성된 지형에서 물이 차는 양을 계산하는 문제이다.

[Trapping Rain Water - LeetCode](https://leetcode.com/problems/trapping-rain-water/)

## 접근 방법

스택을 활용하여 물이 차는 양을 계산한다. 스택에는 현재 인덱스 까지의 높이가 내림차순으로 저장

이전 인덱스에서 현재 인덱스로 이동하면서 변곡점을 찾고, 변곡점에서 스택에서 빼낸 인덱스를 기준으로 물이차는 양을 계산한다.

1. 주어진 높이 리스트를 순회한다.
2. 현재 높이가 스택의 top에 위치한 높이보다 높을 때까지 스택에서 높이를 pop하여 변곡점을 찾는다.
3. 변곡점을 찾으면 이전 인덱스와 현재 인덱스 사이의 거리를 계산.
4. 이전 인덱스와 현재 인덱스 사이의 거리에 해당하는 가로축과 높이의 차이를 계산하여 물이 차는 양을 구한다.
5. 물이 차는 양을 누적시킨다.

## 코드 구현

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        stack = []
        volume = 0
    
        for i in range(len(height)):
        # 변곡점
            while stack and height[i] > height[stack[-1]]:
            # 스택에서 꺼낸다
                top = stack.pop()

                if not len(stack):
                    break

                distance =  i - stack[-1] - 1
                waters = min(height[i], height[stack[-1]]) - height[top]

                volume += distance * waters

            stack.append(i)
        return volume
```
