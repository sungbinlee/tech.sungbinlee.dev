---
title: "Django URL Dispatcher와 정규표현식"
categories:
  - Django
tags:
  - Python
  - Django
  - Regex
  - 
toc: true
toc_sticky: true
toc_label: "Django 튜토리얼"
toc_icon: "book"
---

## Django의 URL Dispatcher
Django에서 URL Dispatcher는 URL 패턴을 뷰(View)에 매핑하는 기능을 합니다. 각 앱의 `urls.py` 파일에서 라우팅 메커니즘이 설정되며, 들어오는 HTTP 요청을 처리합니다. 간략하게 살펴보겠습니다.

### URL 패턴 매칭
- **패턴 트리 구조:** `urlpatterns`를 사용하여 URL을 계층적으로 정의하며, 트리 구조처럼 설정합니다.
- **순차적 매칭:** 요청이 들어오면 Django는 `urlpatterns` 목록에서 위에서 아래로 URL 패턴과 일치하는 것을 찾을 때까지 매칭합니다.
- **Path()와 Re_path() 사용:** Django에서는 URL 패턴에 대해 `path()`와 `re_path()`를 구분합니다.
  - `path()`: 기본 경로 변환기를 사용하여 정규 표현식을 단순화합니다.
  - `re_path()`: 복잡한 패턴 매칭을 위해 정규 표현식을 사용합니다.

### Django 2.x에서 Path()와 Re_path() 예시
```python
from django.urls import path, re_path

urlpatterns = [
    # 기본 변환기를 사용한 경로
    path('articles/<int:year>/', views.year_archive),

    # 같은 패턴을 정규 표현식을 사용하여 설정
    re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
]
```
## Django의 정규 표현식
정규 표현식은 문자열 매칭을 위한 패턴 또는 규칙을 정의합니다. Django의 URL Dispatcher에서 URL 매칭을 가능케 합니다.

### 구문 및 예시
- **기본 구문:** 정규 표현식은 패턴을 정의하는 메타 문자와 기호로 구성됩니다.
- **예시:**
  - `^articles/([0-9]{4})/$`: "articles/"로 시작하는 URL과 네 자리 숫자가 이어지는 패턴과 일치합니다.
  - `^category/(?P<slug>[\w-]+)/$`: 글자, 숫자, 하이픈, 밑줄을 포함하는 카테고리 슬러그로 된 URL과 일치합니다.

### 반복 구문
반복 양자는 패턴에서 문자 또는 그룹이 반복되는 횟수를 지정합니다.
- `+`: 한 번 이상 발생하는 경우와 일치합니다.
- `*`: 0회 이상 발생하는 경우와 일치합니다.
- `?`: 0회 또는 1회 발생하는 경우와 일치합니다.
- `{n}`: 정확히 `n`회 발생하는 경우와 일치합니다.
- `{n,}`: 최소 `n`회 이상 발생하는 경우와 일치합니다.
- `{n,m}`: 최소 `n`회에서 최대 `m`회까지 발생하는 경우와 일치합니다.

### 커스텀 경로 변환기
커스텀 변환기는 URL 매개변수에 재사용 가능한 패턴을 정의하여 가독성과 재사용성을 높일 수 있습니다.
예시:
```python
from django.urls import path, register_converter

class MyCustomConverter:
    regex = '[0-9]{4}/[a-z]+/'  # 커스텀 정규 표현식 패턴 정의

register_converter(MyCustomConverter, 'custom')  # 커스텀 변환기 등록

urlpatterns = [
    path('custom_route/<custom:my_param>/', views.custom_view),
]
```
