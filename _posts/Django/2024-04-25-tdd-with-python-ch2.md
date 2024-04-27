---
title: "[TDD]파이썬을 이용한 클린코드를 위한 테스트 주도 개발: 챕터2"
categories:
  - Django
tags:
  - TDD
  - Python
toc: true
toc_sticky: true
toc_label: "Chapter 2: unittest 모듈을 이용한 기능 테스트 확장"
toc_icon: "book"
---
Chapter 2: unittest 모듈을 이용한 기능 테스트 확장
## 프로젝트의 방향
### To-Do Application 구축하기
- 매우 기본적이면서도 간단한 예제(최소 기능 구현으로 적합)
- 추가 기능(알림, 공유, 마감일, etc.)을 붙일 수 있다.

해당 예제를 통해 웹 프로그래밍의 전반과 TDD 적용 방법에 대해 배울 수 있다.

## 기능 테스트를 이용한 최소 기능의 어플리케이션 설계
### 기능 테스트(Funtional Test, FT)란?
챕터 1에서 `selenium` 을 통해 실제 웹 브라우저를 구동해서 어떻게 `동작` 하는지 확인했다.
- 사용자 관점의 테스트
- 특정 사용자가 어떻게 사용을 하며 이에 대한 어플리케이션의 반응을 확인하는 방식
- FT는 사람이 이해할 수 있는 스토리를 가져야 함
- 스토리를 먼저 주석으로 작성 가능

### [funtional_tests.py](functional_tests.py)
```py
from selenium import webdriver

browser = webdriver.Chrome()

# 에디스(Edith)는 멋진 작업 목록 온라인 앱이 나왔다는 소식을 들고
# 해당 웹 사이트를 확인하러 간다
browser.get ('http://localhost:8000')
# 웹 페이지 타이틀과 헤더가 ' To-Do'를 표시하고 있다
assert 'To-Do' in browser.title
# 그녀는 바로 작업을 추가하기로 한다
# "공작깃털 사기" 라고 텍스트 상자에 입력한다
# (에디스의 취미는 날치 잡이용 그물을 만드는 것이다)
# 엔터키를 치면 페이지가 갱신되고 작업 목록에 # "1: 공작깃털 사기" 아이템이 추가된다
# 추가 아이템을 입력할 수 있는 여분의 텍스트 상자가 존재한다
# 다시 " 공작깃털을 이용해서 그물 만들기"라고 입력한다 (에디스는 매우 체계적인 사람이다)
# 페이지는 다시 갱신되고, 두 개 아이템이 목록에 보인다
# 에디스는 사이트가 입력한 목록을 저장하고 있는지 궁금하다
# 사이트는 그녀를 위한 특정 URL을 생성해준다
# 이때 URL에 대한 설명도 함께 제공된다
# 해당 URL에 접속하면 그녀가 만든 작업 목록이 그대로 있는 것을 확인할 수 있다
# 만족하고 잠자리에 든다
browser.quit()
```
### 실행해보기
```sh
python functional_tests.py

/Users/iseungbin/Desktop/2024/tdd-python/venv/bin/python functional_tests.py 
Traceback (most recent call last):
  File "/Users/iseungbin/Desktop/2024/tdd-python/ch2/functional_tests.py", line 9, in <module>
    assert 'To-Do' in browser.title
           ^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError
```
우선 테스트 서버가 실행되고 있어야 하며, `title`을 변경한 적이 없기 때문에 기대한 대로 실패한다.
## 파이썬 기본 라이브러리의 unittest 모듈
### 기본 파이썬 테스트 코드의 문제
- `AssertionError`라는 메시지가 도움이 안됨(단순 Exception)

파이썬에서는 테스트를 위한 별도 솔루션이 이미 존재한다. 기본 라이브러리의 `unittest` 모듈이 그것이다.
바로 적용해 보자
### [functional_tests_with_unittest.py](functional_tests_with_unittest.py)
```py
from selenium import webdriver
import unittest


class NewVisitorTest(unittest.TestCase):

    def setUp(self):
        self.browser = webdriver.Chrome()
        self.browser.implicitly_wait(3)

    def test_can_start_a_list_and_retrieve_it_later(self):
        # 에디스(Edith)는 멋진 작업 목록 온라인 앱이 나왔다는 소식을 들고
        # 해당 웹 사이트를 확인하러 간다
        self.browser.get('http://localhost:8000')
        # 웹 페이지 타이틀과 헤더가 ' To-Do'를 표시하고 있다
        self.assertIn('To-Do', self.browser.title)
        self.fail('Finish the test!')
        # 그녀는 바로 작업을 추가하기로 한다
        # "공작깃털 사기" 라고 텍스트 상자에 입력한다
        # (에디스의 취미는 날치 잡이용 그물을 만드는 것이다)
        # 엔터키를 치면 페이지가 갱신되고 작업 목록에 # "1: 공작깃털 사기" 아이템이 추가된다
        # 추가 아이템을 입력할 수 있는 여분의 텍스트 상자가 존재한다
        # 다시 "공작깃털을 이용해서 그물 만들기"라고 입력한다 (에디스는 매우 체계적인 사람이다)
        # 페이지는 다시 갱신되고, 두 개 아이템이 목록에 보인다 # 에디스는 사이트가 입력한 목록을 저장하고 있는지 궁금하다
        # 사이트는 그녀를 위한 특정 URL을 생성해준다
        # 이때 URL에 대한 설명도 함께 제공된다
        # 해당 URL에 접속하면 그녀가 만든 작업 목록이 그대로 있는 것을 확인할 수 있다
        # 만족하고 잠자리에 든다

    if __name__ == '__main__':
        unittest.main(warnings='ignore')
```
- `unittest.TestCase`를 상속해서 테스트를 클래스 형태로 만든다.
- 모든 테스트는 `test_` 라는 명칭으로 시작하는 메소드만 실행된다.
- `setUp`과 `tearDown`은 테스트 시작 전과 후에 실행된다.
- `assert` 대신 `self.assertIn`을 사용하여 결과를 검증한다.
  - unittest는 `assertEqual`, `assertTrue`, `assertFalse` 과 같은 유용한 함수를 다수 제공한다
  - 공식문서: https://docs.python.org/3/library/unittest.html
- `self.fail`은 강제적으로 테스트를 실패시킨다.

### 실행해보기
```sh
python -m unittest functional_tests_with_unittest.py

F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later (functional_tests_with_unittest.NewVisitorTest.test_can_start_a_list_and_retrieve_it_later)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/iseungbin/Desktop/2024/tdd-python/ch2/functional_tests_with_unittest.py", line 19, in test_can_start_a_list_and_retrieve_it_later
    self.assertIn('To-Do', self.browser.title)
AssertionError: 'To-Do' not found in 'The install worked successfully! Congratulations!'

----------------------------------------------------------------------
Ran 1 test in 0.882s

FAILED (failures=1)
```
예상대로 실패하였고, `aseertIn` 을 통해 보다 자세한 정보를 확인할 수 있다.
## 이번 장을 통해:
- `unittest`를 통해 기능 테스트를 구현해 보았다.
- 유용한 TDD 개념을 접할 수 있었다.
  - 사용자 스토리(User Story): 사용자 관점에서 어떻게 앱이 동작하는지 기술 한 것이며 기능테스트 구조화를 위해 사용한다.
  - 예측된 실패(Expected failure): 의도적으로 구현한 테스트 실패
