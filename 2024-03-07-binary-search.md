---
title: "Binary search"
categories:
  - Killing Camp
tags:
  - Coding test
  - Python
toc: true
toc_sticky: true
toc_label: "Binary search"
toc_icon: "book"
---

[Binary Search - LeetCode](https://leetcode.com/problems/binary-search/description/)

## 문제 파악

주어진 배열속에서 주어진 target을 찾는 문제이며 시간복잡도 로그 n 의 풀이로 풀어야한다.

## 접근 방법

배열이 정렬되어있는 경우 이진탐색을 통해 효율적인 풀이가 가능.

1. 배열의 가운데 요소과 target 값을 비교한다.
2. 대상이 가운데 요소보다 작으면 배열의 왼쪽 반을 살펴보고, 대상이 크면 배열의 오른쪽 반을 살펴본다.
3. 이를 대상을 찾거나 배열의 끝에 도달할 때까지 반복합니다.

## 코드 구현

```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums)-1
        
        while left <= right:
            mid = (left + right) // 2
            if target == nums[mid]:
                return mid
            elif target > nums[mid]:
                left = mid + 1
            else:
                right = mid - 1
                
        return -1
```

## 배우게 된 점

이진 탐색은 배열이 정렬되어 있을 때 효율적으로 요소를 검색하는 데 사용될 수 있는 강력한 알고리즘이며 주어진 문제를 해결하기 위해 배열의 중간 요소와 대상을 반복적으로 비교함으로써 검색 속도를 크게 향상시킬 수 있다.
