---
title: "장고 뷰 이해하기: FBV vs CBV"
categories:
  - Django
tags:
  - Python
  - Django
  - FBV
  - CBV
toc: true
toc_sticky: true
toc_label: "FBV vs CBV"
toc_icon: "book"
---

## 장고의 호출 가능한 객체
장고에서 뷰는 사용자 요청에 어떻게 응답할지를 결정합니다. 함수 기반 뷰 (FBV)와 클래스 기반 뷰 (CBV)는 두 가지 주요 패러다임으로 뷰를 구현하는 방식입니다.

### 함수 기반 뷰 (FBV)

FBV는 장고에서 뷰를 구현하는 기본적인 방식입니다. 함수를 사용하여 만들며, 공통 기능들은 종종 장식자 문법을 활용합니다:
```python
@api_view(['GET'])
@throttle_classes([OncePerDayUserThrottle])
def my_view(request):
    return Response({"message": "Hello world!"})
```

### 클래스 기반 뷰 (CBV)

CBV는 함수를 생성하는 클래스입니다. 상속을 통해 기능을 물려받을 수 있어 더 체계적인 방법으로 뷰를 다룰 수 있습니다.
```python
class MyView(APIView):
    throttle_classes = [OncePerDayUserThrottle]

    get(self, request):
        return Response({"message": "Hello world!"})
```

### CBV 파헤쳐보기

CBV는 뷰 함수를 생성하는 클래스입니다. `as_view()` 클래스 함수를 통해 뷰 함수를 생성하고, 상속을 활용하여 여러 기능을 믹스인할 수 있습니다. 장고의 기본 CBV 패키지는 `django.views.generics`에 있습니다. 이외에도 `django-braces`와 같은 써드파티 CBV 패키지도 있습니다.

#### 함수를 통한 동일한 View 함수 생성 예시
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(['GET'])
def my_view(request):
    return Response({"message": "Hello, world!"})
```

#### 클래스로 동일한 View 함수 구현 예시
```python
from rest_framework.views import APIView
from rest_framework.response import Response

class MyView(APIView):
    def get(self, request):
        return Response({"message": "Hello, world!"})
```

#### 장고 기본 제공 CBV 활용 예시
```python
from django.views.generic import TemplateView

class HomePageView(TemplateView):
    template_name = 'home.html'
```

#### 상속을 통한 CBV 속성 정의 예시
```python
from django.views.generic import ListView
from .models import MyModel

class MyListView(ListView):
    model = MyModel
    template_name = 'my_model_list.html'
```

### CBV의 이해와 권장 사항

CBV는 장고의 관례를 따를 경우 아주 간결한 코드로 구현할 수 있습니다. 하지만 이러한 관례를 이해하는 것이 중요하며, FBV를 통한 경험이 도움이 됩니다. 필요한 설정값을 제공하거나 특정 함수를 재정의하여 커스터마이징할 수 있지만, 관례를 벗어난 구현은 복잡성을 증가시킬 수 있습니다.

CBV를 제대로 이해하기 위해서는 파이썬 클래스에 대한 이해가 필요합니다. 특히 상속과 인자의 패킹/언패킹에 대한 이해가 필요합니다. CBV 코드를 동일하게 동작하는 FBV로 구현해보는 연습을 권장합니다.
