---
title: "Django 튜토리얼"
categories:
  - Django
tags:
  - Basic
  - Python
  - Tutorial
toc: true
toc_sticky: true
toc_label: "Django 튜토리얼"
toc_icon: "book"
---

## 1. Django란?

Django는 파이썬으로 작성된 무료 오픈 소스 웹 프레임워크입니다. 웹 개발을 빠르고 쉽게 할 수 있도록 도와주는 도구 모음이라고 할 수 있습니다. Django는 웹 개발에서 반복되는 작업을 최소화하고 안정적인 기반을 제공하여 개발자가 보다 효율적으로 웹 애플리케이션을 개발할 수 있도록 돕습니다. 장고는 모델-뷰-컨트롤러(MVC) 아키텍처 패턴을 기반으로 한 MTV(Model-Template-View) 패턴을 사용하여 웹 애플리케이션의 구조를 구성합니다.

## 2. 환경 세팅

1. 파이썬 및 pip 설치: Django는 파이썬 언어로 작성되었으므로 우선적으로 파이썬을 설치해야 합니다. 또한, pip를 통해 Django를 설치할 것이므로 pip도 함께 설치합니다.

2. 가상환경(Virtual Environment) 설정: 프로젝트마다 독립적인 가상환경을 만들어 사용하는 것이 좋습니다. 가상환경은 파이썬 패키지를 격리된 환경에 설치하여 충돌을 방지하고 프로젝트 간의 의존성 관리를 용이하게 합니다.

## 3. 배포 프로세스

### 3.1 로컬 환경 세팅
1. 내가 window를 사용하고 있더라도 배포 환경과 동일 환경을 하나 구축합니다. (mac은 거의 mac에서 합니다.)
2. Django 코딩을 한 후 Github에 내가 작성한 코드를 업로드합니다.
3. 업로드한 코드를 서버쪽에서 다운로드 하여 실행시킵니다.(`python manage.py runserver`)
* 2번과 3번을 통합하는 것을 `CI/CD` 구축이라고 합니다. 그래서 push를 하면 자동으로 테스트 서버에 배포되고, 문제 없으면 실 서버에 배포 될 수 있도록 하는 편입니다.

## 4. Django tutorial

1. 어떤 파일이 수정되어야 하는지 아래 이미지를 참고하세요.
2. url을 설계합니다.

```
sungbin.com/
sungbin.com/about       ->  about.html
sungbin.com/product     ->  product.html
sungbin.com/product/1   ->  productdetails.html
sungbin.com/a           ->  a.html
sungbin.com/b           ->  b.html
sungbin.com/c           ->  c.html
```

3. urls.py를 수정합니다. 다만, 지금 Django가 설치되어 있지 않기 때문에 Django를 설치합니다.(설계가 우선이기 때문에 위에서 충분히 기획을 한 다음에 한 번에 개발합니다.)

```
pip install --upgrade pip
mkdir mysite
cd mysite
python -m venv myvenv
source myvenv/bin/activate
pip install django==3.2
django-admin startproject tutorialdjango .
python manage.py migrate
```

4. 설치가 다 되었으면 `mysite/tutorialdjango/settings.py` 파일을 열고 기본 세팅을 합니다. 지금은 tutorial이기 때문에 28번째 줄에 있는 `ALLOWED_HOSTS = ['*']`만 수정을 합니다.

5. `python manage.py runserver 0:80` 서버를 구동해봅니다. 클라우드에서 만들었기 때문에 바로 배포가 된 상태입니다.

6. urls.py를 설계대로 코딩하기 위해 main이라는 앱을 만들고 아래처럼 코딩합니다. 터미널에서 `python manage.py startapp main` 를 입력하고 mysite/tutorialdjango/settings.py파일에서 INSTALLED_APPS에 app을 추가합니다.

```
urlpatterns = [
    path('', views.home, name='home'),  # sungbin.com/ -> home.html
    path('about/', views.about, name='about'),  # sungbin.com/about -> about.html
    path('product/', views.products, name='products'),  # sungbin.com/product -> products.html
    path('product/<int:product_id>/', views.product_details, name='product_details'),  # sungbin.com/product/1 -> productdetails.html
    path('a/', views.a_page, name='a_page'),  # sungbin.com/a -> a.html
    path('b/', views.b_page, name='b_page'),  # sungbin.com/b -> b.html
    path('c/', views.c_page, name='c_page'),  # sungbin.com/c -> c.html
]
```

```
INSTALLED_APPS = [
    'main',
    'django.contrib.admin',
    'django.contrib.auth',
    #... 생략 ...
]
```

7. views.py 파일로 가서 함수를 아래와 같이 모두 만듭니다.

```
from django.shortcuts import render

def index(request):
    return render(request, 'main/index.html')

def about(request):
    return render(request, 'main/about.html')

def product(request):
    return render(request, 'main/product.html')

def productdetails(request):
    return render(request, 'main/productdetails.html')

def a(request):
    return render(request, 'main/a.html')

def b(request):
    return render(request, 'main/b.html')

def c(request):
    return render(request, 'main/c.html')
```

8. `mysite/main/templates/main`와 같은 형식으로 폴더를 만들고 그 안에 `index.html`을 비롯한 html파일을 모두 생성합니다. 안에는 구분할 수 있는 간단한 텍스트만 넣습니다.

9. `mysite/`로 이동하셔서 `python manage.py runserver 0:80`을 입력하고 각각 url을 test해봅니다. `product/1`만 제외하고 작동합니다. 위에 url이 작동하도록 만들어보도록 하겠습니다. `/mysite/main/views.py`로 들어갑니다. 아래와 같이 넣으면 `product/1`은 정상적으로 작동합니다. (data는 어떻게 .html에서 값을 읽어올 수 있는지 보여드리기 위한 의미 없는 예제입니다.)

```
def productdetails(request, pk):
    data = {
        'value': pk + 100,
        'one': [1, 2, 3, 4],
        'two': {'hello': 100, 'world': 200}
    }
    return render(request, 'main/productdetails.html', data)
```

다음은 productdetails.html입니다. python문법처럼 대괄호를 사용하시면 error가 납니다. index로 접근이어도 dot(`.`)을 이용해서 접근해주세요.

```
여기는 product에 {{ value }} 입니다.
읽어온 값들을 여기에 보여드리겠습니다.
{{one}}
{{one[0]}}
{{one.0}}
{{two.hello}}
```

애러가 나지 않는 코드는 아래와 같습니다.

```
여기는 product에 {{ value }} 입니다.
읽어온 값들을 여기에 보여드리겠습니다.
{{one}}
{{one.0}}
{{two.hello}}
```

10. models.py로 가서 이제 홈페이지에 들어갈 데이터베이스를 설계합니다. 코딩하기 전에 설계가 우선입니다. 보통은 2번 설계할 때 함께 설계를 합니다. 아래와 같이 models.py를 작성해주세요.

```
from django.db import models

class Cafe(models.Model):
    name = models.CharField(max_length=50)
    content = models.TextField()

    def __str__(self):
        return self.name
```

11. 위코드를 가지고 database를 만질 수 있는 명령어인 `python manage.py makemigrations`를 입력하고, 실제 DB에 반영하는 명령어인 `python manage.py migrate`

12. admin에 Cafe를 등록하고 직접 글을 쓰거나 사제를 해보는 시간을 가져보도록 하겠습니다. `admin.py`파일에 수정 내역입니다.

```
from django.contrib import admin
from .models import Cafe

admin.site.register(Cafe)
```

13. superuser를 만듭니다. 터미널에 `python manage.py createsuperuser`라고 입력해주세요. 비밀번호가 너무 짧으면 재 설정을 하라 하니 주의해주세요.

14. `python manage.py runserver 0:80`을 입력하신 후 `기본url/admin`으로 이동합니다. 로그인 후 cafes에 add를 클릭해 게시물 3개를 만듭니다.

15. views.py파일을 열어 index 함수를 수정합니다.

```
from django.shortcuts import render, redirect
from .models import Cafe

def index(request):
    cafes = Cafe.objects.all()
    context = {
        'cafes': cafes
    }
    return render(request, 'main/cafelist.html', context)
```

16. index.html을 아래와 같이 수정하시면 웹 페이지에서 게시물을 볼 수 있습니다.

```
<!DOCTYPE html>
<html>
<head>
  <title>cafelist</title>
</head>
<body>
  <h1>cafelist</h1>
  <table>
    {% for cafe in cafes %}
    <tr>
      <td>{{ cafe.name }}</td>
      <td>{{ cafe.content }}</td>
    </tr>
    {% endfor %}
  </table>
</body>
</html>
```
