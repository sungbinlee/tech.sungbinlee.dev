---
title: "[TDD]파이썬을 이용한 클린코드를 위한 테스트 주도 개발: 챕터3"
categories:
  - Django
tags:
  - TDD
  - Python
toc: true
toc_sticky: true
toc_label: "Chapter 3: 단위 테스트를 이용한 간단한 홈페이지 테스트"
toc_icon: "book"
---
Chapter 3: 단위 테스트를 이용한 간단한 홈페이지 테스트

## 첫 Django App과 첫 단위 테스트
- Django는 코드를 앱(App) 형태로 구조화 하도록 도와준다.
- 다른 프로젝트에서 앱을 재사용할 수 있다.

### 작업 목록 앱 생성
```sh
$ python manage.py startapp lists
```
실행을 하면 superlists 동일선상에 lists 폴더가 생성된다.
```shell
$ tree
```
```
superlists
├── functional_test.py
└── superlists
    ├── lists
    │   ├── __init__.py
    │   ├── admin.py
    │   ├── apps.py
    │   ├── migrations
    │   │   └── __init__.py
    │   ├── models.py
    │   ├── tests.py
    │   └── views.py
    ├── manage.py
    └── superlists
        ├── __init__.py
        ├── settings.py
        ├── urls.py
        └── wsgi.py
```
## 단위 테스트 vs 기능 테스트
### 기능 테스트(Functional Test)
- 사용자 관점에서 애플리케이션 외부를 테스트를 하는 것
### 단위 테스트(Unit Test)
- 프로그래머 관점에서 그 내부를 테스트 하는 것

TDD 에서는 양쪽 테스트를 모두 적용한다.
### TDD 작업 순서
1. **기능 테스트 작성**: 사용자 관점에서의 새로운 기능성을 정의
2. **기능 테스트 실패**: 어떻게 하면 테스트를 통과 할지 고민
   1. **단위 테스트 작성**: 고민을 단위 테스트로 작성
   2. **단위 테스트 실패**: 최소한의 코드 작성
3. **기능 테스트 재실행**: 성공할 때까지 2번 단계를 반복

즉, **상위 레벨의 기능을 테스트**하고 이를 통과하기 위해 **하위 레벨의 단위 테스트를 작성하고 실패**시키며 **점진적으로 코드를 구현**해가는 방식으로 개발하는게 **핵심**이다.
## Django에서의 단위 테스트
고의적인 실패 테스트를 만들어서 확인해 보자.
### [lists/tests.py](superlists/lists/tests.py)
```python
from django.test import TestCase

class SmokeTest(TestCase):
    
    def test_bad_maths(self):
        self.assertEqual(1 + 1, 3)
```
### 실행해보자
```shell
$ python manage.py test
```
```
Found 1 test(s).
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_bad_math (lists.tests.SmokeTest.test_bad_math)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/iseungbin/Desktop/2024/tdd-python/ch3/superlists/lists/tests.py", line 7, in test_bad_math
    self.assertEqual(1 + 1, 3)
AssertionError: 2 != 3

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)

```
## Django의 MVC, URL, View 함수
Django 는 대체로 MVC 패턴을 따른다. Django 에서는 MTV(Model-Template-View)로 표현한다.
### 장고의 처리 흐름
1. 특정 URL에 대한 HTTP `요청`을 받는다.
2. 요청을 `URLconf` 을 통해 해석 하고 적절한 뷰를 매핑한다.
3. 해당 뷰 기능이 요청을 처리해서 HTTP 응답을 반환한다.
### 우리가 테스트 해야할 것은?
- URL의 루트("/")를 해석해서 특정 뷰 기능에 매칭시킬 수 있는가?
- 이 뷰 기능이 특정 HTML을 반환하게 해서 기능 테스트를 통과할 수 있는가?


### [lists/tests.py - 첫 번째 단위 테스트 작성](superlists/lists/tests.py)
```python
from django.urls import resolve
from django.test import TestCase
from lists.views import home_page


class HomePageTest(TestCase):
    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)
```
### 실행해보자
```shell
$ python manage.py test
```
```
...
ImportError: cannot import name 'home_page' from 'lists.views' (/Users/iseungbin/Desktop/2024/tdd-python/ch3/superlists/lists/views.py)

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
```
예상한대로 에러가 발생한다. 
- 아직 존재하지도 않는 것을 임포트 하려고 했기 때문이다.
- TDD 관점에서는 좋은 소식이다.
- 우리는 이제 **실패한 기능 테스트**와 **실패한 단위 테스트**를 가지고 있다.
## 마침내 실질적인 애플리케이션 코드를 작성한다
실패 테스트를 해결하기 위한 최소한의 수정만 한다.
- 우리가 가지고 있는 실패 테스트는?
  - `list.views` 에서 `home_page`를 임포트 할수 없다.
### [lists/views.py](superlists/lists/views.py)
```py
from django.shortcuts import render

# Create your views here.
home_page = None
```
**...???** 아무리 최소한의 수정이라고는 하지만 너무 한게 아닌가? 라는 생각이 든다.
저자에 따르면 TDD 실습의 시발점이자 모든것이라고 하니 계속 따라해보자.

### 실행해보자
```shell
$ python manage.py test
```
다음과 같은 에러에 당면한다.
```
Found 1 test(s).
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
E
======================================================================
ERROR: test_root_url_resolves_to_home_page_view (lists.tests.HomePageTest.test_root_url_resolves_to_home_page_view)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/iseungbin/Desktop/2024/tdd-python/ch3/superlists/lists/tests.py", line 15, in test_root_url_resolves_to_home_page_view
    found = resolve('/')
            ^^^^^^^^^^^^
  File "/Users/iseungbin/Desktop/2024/tdd-python/venv/lib/python3.11/site-packages/django/urls/base.py", line 24, in resolve
    return get_resolver(urlconf).resolve(path)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/iseungbin/Desktop/2024/tdd-python/venv/lib/python3.11/site-packages/django/urls/resolvers.py", line 702, in resolve
    raise Resolver404({"tried": tried, "path": new_path})
django.urls.exceptions.Resolver404: {'tried': [[<URLResolver <URLPattern list> (admin:admin) 'admin/'>]], 'path': ''}

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
Destroying test database for alias 'default'...
```

## urls.py
- Django는 `urls.py`라는 파일을 이용해서 어떻게 URL을 뷰 함수에 맵핑할지 정의한다. [관련 글 보기](https://sungbinlee.dev/django/url-dispatcher/)

책에서의 Django는 1.7 기준으로 작성되어 있다. 필자는 4.2 버전으로 진행하는 만큼 최신버전에 맞게 코드를 수정을 했다.

### [superlists/urls.py](superlists/superlists/urls.py)
```py
from django.contrib import admin
from django.urls import path, include
from lists import views as home_views

urlpatterns = [
    # path("admin/", admin.site.urls),
    path('', home_views.home_page, name='home'),
]
```
### 실행해보자
```shell
$ python manage.py test
```
```
Creating test database for alias 'default'...
Destroying test database for alias 'default'...
Traceback (most recent call last):
[...]
  File "../superlists/superlists/urls.py", line 22, in <module>
    path('/', lists.views.home_page, name='home'),
  File "/lib/python3.7/site-packages/django/urls/conf.py", line 73, in _path
    raise TypeError('view must be a callable or a list/tuple in the case of include().')
TypeError: view must be a callable or a list/tuple in the case of include().
```
에러가 발생한다. 
- url과 view를 성공적으로 매핑을 하였으나 
- view가 호출할수 없다 라는 에러메세지가 발생하고 있다. 
- 실제로 최소한의 작동가능한 코드만 구현해놨기 때문이다(`home_page`가 아직 함수가 아님)

코드를 실제 함수로 구현해 보자.
### [lists/views.py](superlists/lists/views.py)
```py
from django.shortcuts import render

# Create your views here.
def home_page():
    pass
```
### 다시 실행해보자
```shell
$ python manage.py test
```
```
Found 1 test(s).
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
Destroying test database for alias 'default'...
```

와! 성공이다. 모든 코드 변경은 테스트에 의해 검증돼야 한다.

이로써 첫 단위 테스트와 url 맵핑 그리고 임시 뷰까지 완료되었다.
## 뷰를 위한 단위 테스트
- 이제 빈 함수에서 HTML 형식의 실제 응답을 반환하는 함수를 작성해야 한다.
- 이제 우리가 해야할 일은..? **테스트를 먼저 작성하자.**
### [lists/views.py](superlists/lists/views.py)
```python
# from django.core.urlresolvers import resolve # Django 2.0 에서 삭제
from django.urls import resolve
from django.test import TestCase
from django.http import HttpRequest

from lists.views import home_page


class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)
    
    # 추가된 테스트
    def test_home_page_returns_correct_html(self):
        request = HttpRequest() 
        response = home_page(request) # home_page 뷰에 요청해서 응답을 받는다
        self.assertTrue(response.content.startswith(b'<html>')) # 응답 내용이 <html>로 시작하는가?
        self.assertIn(b'<title>To-Do lists</title>', response.content) # 타이틀이 To-Do lists 를 포함하는가?
        self.assertTrue(response.content.endswith(b'</html>')) # 응답 내용이 </html> 끝나는가?
```

요약하자면, 응답 내용이 아래와 같이 오는지 검증하는 내용이다.
```html
<html>
    <title>To-Do lists</title>
</html>
```
테스트를 돌려보면 역시나 실패한다. 
```html
TypeError: home_page() takes 0 positional arguments but 1 was given
```
이제 점진적으로 코드를 구현해 나갈 것이다.

### 단위 테스트-코드 주기
1. 터미널에서 단위 테스트를 실행해서 어떻게 실패하는지 확인한다.
2. 편집기상에서 현재 실패 테스트를 수정하기 위한 최소한의 코드를 변경한다.

**TDD 단위 테스트-코드 주기**에 대해 생각해보자.
- 코드 품질을 높이려면? 코드 변경을 최소화
- 최소화한 코드 구현은 하나하나 테스트에 의해 검증

즉, 작은 단위로 나누어 코드 변경을 해라. 이 주기를 따라가보자

- **최소한의 코드 변경**

어떤 에러가 발생 했었지?
```html
TypeError: home_page() takes 0 positional arguments but 1 was given
```
해당 에러는 home_page 함수에 argument가 없어서 발생하는 문제이다 따라서 이를 추가해주자.
```python
# lists/views.py
def home_page(request):
    pass
```
- **테스트**
```
self.assertTrue(response.content.startswith(b'<html>'))
AttributeError: 'NoneType' object has no attribute 'content'
```
- **코드**

아 리턴값이 없다! 그러면 HttpResponse로 응답해주자.
```python
# lists/views.py
from django.http import HttpResponse

def home_page(request):
    return HttpResponse()
```
- **다시 테스트**
```
self.assertTrue(response.content.startswith(b'<html>'))
AssertionError: False is not true
```
- **다시 코드**

리턴 값 객체는 맞게 왔는데 content가 없다. 내용을 채워서 보내자.
```python
# lists/views.py
from django.http import HttpResponse

def home_page(request):
    return HttpResponse('<html><title>To-Do lists</title></html>')
```

- **테스트**
```sh
python manage.py test
```
```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.002s

OK
Destroying test database for alias 'default'...
```
성공!
### 기능 테스트
단위 테스트가 끝났으니 이제 사용자 관점에서의 기능 테스트를 실행 시켜보자!
```sh
# 장고 서버가 실행 중 이여야 한다.
$ python -m unittest functional_tests_with_unittest.py
```
```
F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later (functional_tests_with_unittest.NewVisitorTest.test_can_start_a_list_and_retrieve_it_later)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/iseungbin/Desktop/2024/tdd-python/ch3/functional_tests_with_unittest.py", line 20, in test_can_start_a_list_and_retrieve_it_later
    self.fail('Finish the test!')
AssertionError: Finish the test!

----------------------------------------------------------------------
Ran 1 test in 1.508s

FAILED (failures=1)
```
의도적으로 심어준 self.fail() 때문에 unittest는 실패했다고 나오지만 사실상 성공한 것이다. 저자가 끝까지 긴장감을 주기 위해 설치한 트릭이였다. 그러면 무사히 첫 기능 테스트도 완료했다!

## 이번 장을 통해:
- **단위 테스트-코드 주기** 와 **최소한의 코드 변경** 
  - 터미널에서 단위 테스트 실행
  - 최소한의 코드 수정
  - 반복

- 실제로 구현한 코드의 양은 얼마 되지도 않는다. 
  - 하지만 TDD는 이 작업을 **매우 고되게** 만들어 준다. 
  - 반대로 익숙해지면은 빠르게 **고품질의 코드**를 생산할수가 있겠구나!
