---
title: "유연한 URL 시스템 구축하기: URL Reverse"
categories:
  - Django
tags:
  - Python
  - Django
  - URL
  - Backend
toc: true
toc_sticky: true
toc_label: "URL Reverse"
toc_icon: "book"
---

Django에서는 URL Dispatcher 시스템을 활용하여 Django 앱에서 유연하고 효과적으로 URL을 관리하고, URL Reverse를 통해 URL이 변경되더라도 Reverse가 자동으로 해당 변경된 URL을 추적할 수 있습니다.

## URL Reverse 함수

### 1. URL 템플릿 태그

```django
<!-- 템플릿에서 reverse 사용 -->
<a href="{% url 'view_name' %}">Link</a>
```

### 2. `reverse` 함수

```python
from django.urls import reverse

# URL Reverse를 통한 URL 생성
url = reverse('view_name')
```

### 3. `resolve_url` 함수

```python
from django.shortcuts import resolve_url

# 매핑 URL이 없을 때 문자열 그대로 반환
url = resolve_url('view_name')  # 또는 resolve_url('view_name', args=('arg1',))
```

##### 4. `redirect` 함수
내부적으로는 `resolve_url()`을 사용한다.
```python
from django.shortcuts import redirect

# 매핑 URL이 없을 때 인자 문자열을 URL로 사용
return redirect('view_name')  # 또는 redirect('view_name', arg1='arg1')
```

### 모델 객체에 대한 Detail 주소 계산

모델 클래스에 `get_absolute_url` 메서드를 추가하여 객체의 detail 주소를 계산합니다.

```python
from django.db import models
from django.urls import reverse

class YourModel(models.Model):
    # 모델 필드들

    def get_absolute_url(self):
        return reverse('model_detail_view', args=[str(self.pk)])
```

## 활용

### CreateView / UpdateView

`success_url`을 제공하지 않을 경우, 해당 모델 인스턴스의 `get_absolute_url` 주소로 자연스럽게 이동 가능합니다.

### Detail 뷰 설정 시

특정 모델에 대한 Detail 뷰를 작성할 때, URLConf 설정 시 `get_absolute_url` 설정을 권장합니다. 이렇게 하면 코드가 더 간결해지며, 생성 또는 수정 후 detail 화면으로의 자연스러운 이동이 가능합니다.

### 추가 팁

- URL Reverse를 사용하여 View에서 URL을 하드코딩하지 않고, 유지보수를 편리하게 할 수 있습니다.
- `get_absolute_url`은 모델 인스턴스를 사용해 해당 객체의 URL을 가져오는 가장 일반적인 방법입니다.

이런 유연한 URL 시스템을 통해 Django 애플리케이션을 보다 모듈화하고 유지보수 가능하게 만들 수 있습니다.
