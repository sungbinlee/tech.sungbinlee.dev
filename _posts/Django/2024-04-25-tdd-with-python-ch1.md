---
title: "[TDD]파이썬을 이용한 클린코드를 위한 테스트 주도개발 - 챕터1"
categories:
  - Django
tags:
  - TDD
  - Python
toc: true
toc_sticky: true
toc_label: "파이썬을 이용한 클린코드를 위한 테스트 주도개발"
toc_icon: "book"
---
Chapter 1: 기능 테스트를 이용한 Django 설치
## 테스팅 고트님께 복종하라! 테스트가 없으면 아무것도 하지마라!
> 소프트웨어 개발에서 일반적으로 코드를 작성한 후에 테스트 케이스를 작성하는 방법이 흔히 사용되며, 코드를 작성한 후에 테스트를 통해 코드의 동작을 확인하고 문제를 해결한다.

### Testing Goat: 테스트하는 염소
- 염소는 한 번에 한 가지만 한다.
- 테스트를 먼저해, 테스트를 먼저 하라고!
  - TDD에서 가장 먼저 해야 하는 것은 "**테스트를 작성해라**"
  - 그리고 예상대로 실패하는지 확인한다.
  - 그 후 코드를 작성하고, 이 과정을 무한히 반복한다.

### 첫 번째 기능 테스트(Functional Test, FT)
- 크롬 브라우저 창을 실행하기 위해 셀레늄의 webdriver를 가동한다.
- 브라우저를 통해 로컬 PC상의 웹 페이지를 연다.
- 웹 페이지 타이틀에 'successfully' 라는 단어가 있는지 확인한다.

#### 요구사항 설치
```sh
pip install selenium
```
#### 기능 테스트 작성([functional_test.py](./functional_test.py))
```py
from selenium import webdriver

browser = webdriver.Chrome()
browser.get("http://localhost:8000/")

assert 'successfully' in browser.title
```
#### 실행해보기
```shell
python functional_test.py
```
예상대로 실패한다.
![img.png](img.png)
`selenium.common.exceptions.WebDriverException: Message: unknown error: net::ERR_CONNECTION_REFUSED`

해당 에러는 브라우저가 `http://localhost:8000/` 에접근을 못할때 발생하는 에러이다.

#### Django 가동 및 실행

```sh
# 장고 4.2 버전 설치
pip install Django==4.2
# 프로젝트 시작 
django-admin startproject superlists
# 장고 구동(manage.py 로 이동후)
python manage.py runserver
```
#### 테스트 다시 실행해보기
```shell
python functional_test.py
```
커맨드 라인상 변화가 없다. 이 말은 `AssertionError`가 발생하지 않는다는 뜻으로 테스트를 통과 한 것이다!

#### 구동페이지 확인
![img_1.png](img_1.png)

### 이번 장을 통해:
- Testing Goat에 비유한 TDD의 핵심 개념을 이해할 수 있었다.
- 프로젝트 시작부터 TDD가 가능하다는 작가의 의도를 알 수 있었다.
