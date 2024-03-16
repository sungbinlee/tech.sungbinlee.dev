---
title: "Daily Temperatures"
categories:
  - Killing Camp
tags:
  - Coding test
  - Python
  - LeetCode
  - 스택
toc: true
toc_sticky: true
toc_label: "Daily Temperatures"
toc_icon: "book"
---
## 문제 파악
주어진 일일 온도 리스트에서 각 날짜마다 따뜻한 온도가 몇 일 후에 오는지를 계산하는 문제 (따뜻한 온도가 오지 않는 경우에는 0을 반환)

[Daily Temperatures - LeetCode](https://leetcode.com/problems/daily-temperatures/)

## **접근 방법**

이 코드는 스택을 사용하여 각 날짜마다 따뜻한 온도가 언제 오는지를 계산합니다. 스택에는 온도의 인덱스가 저장되며, 현재 온도보다 낮은 온도들의 인덱스는 스택에서 제거하면서 해당 온도까지 기다려야 하는 일 수를 계산

1. 각 날짜마다 일일 온도를 순회하면서 현재 온도와 스택의 top에 위치한 온도를 비교
2. 만약 현재 온도가 스택의 top에 위치한 온도보다 높다면, 해당 인덱스의 온도는 따뜻해질 것이므로 현재 날짜와 스택에 저장된 날짜의 차이를 계산하여 해당 인덱스의 따뜻한 온도까지의 기다려야 하는 일 수를 저장
3. 위 과정을 반복하여 따뜻한 온도가 오지 않는 경우는 스택에 현재 날짜를 저장

## 코드 구현

```python
class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        # temperatures 어레이는 일일온도 담겨있음
        # 얼마나 기다려야 따뜻해지는지 담긴 배열 반환, 따뜻해질 예정 없으면 값 0 
        
        answer = [0] * len(temperatures)
        
        stack = []
        
        for cur_d, cur_t in enumerate(temperatures):
            while stack and stack[-1][1] < cur_t:
                prev_d, _ = stack.pop()
                answer[prev_d] = cur_d - prev_d 
            
            stack.append((cur_d, cur_t))
            
        return answer
```
