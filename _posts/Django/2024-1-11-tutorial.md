---
title: "Django 초기 설정"
categories:
  - Django
tags:
  - Tutorial
  - Python
toc: true
toc_sticky: true
---
## 프로젝트 생성 및 초기 프로젝트 환경설정

### 가상환경 생성
```
pyhton -m venv venv
# 활성화
source venv/bin/activate
# which 명령어로 잘들어와 졌는지 확인

pip install django~=4.2.0 # 버전 지정 설치

# 장고 프로젝트 생성
django-admin startproject 프로젝트명 .
```

### requirements 설정 
```
# 
vi requirements.txt # -r requirments/prod.txt
vi requirments/common.txt requirments/dev.txt requirments/prod.txt 

# common 공통 라이브러리(django~=x.x.x)
# dev 개발 전용 라이브러리(django sql toolbar) -r common.txt + 개발 라이브러리
# prod 프로덕션용 라이브러리 -r common.txt + 프로덕션 라이브러리
```

### media/static 및 template 설정
```python
# STATIC_URL = '/static/' 밑에
STATICFILES_DIRS = [
  os.path.join(BASE_DIR, 'instagram', 'static'),
]
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
# 정적인 파일들은 위를 통해 처리
# 모델에 있는 파일 필드나 이미지 필드를 통해서 업로드 된 파일은 미디어를 통해 처리
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# TEMPLATES = [
#   {
#        'BACKEND': 'django.template.backends.django.DjangoTemplates',
#        'DIRS': [
            os.path.join(BASE_DIR, 'instagram', 'templates'),
#       ],

```

### 버전 관리 등록
```
git init
# 이그노어 등록
vi .gitignore
# __pycache__
# db.sqlite3
# /static
# /media
# .idea
```

### url conf 초기 설정
```python
from django.conf import settings
# from django.contrib import admin
# from django.urls import path, include
from django.conf.urls.static import static

# urlpatterns = [
#    path('admin/', admin.site.urls),
# ]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL,
                          document_root=settings)
```
서버 구동해보기..

### 개발용 장고 디버그 툴바 설치
```
pip install django-debug-toolbar

dev.txt 에 등록 후 공식 문서 따라서 설치
```
... settings 도 분기 가능 하나 manage.py, wsgi, asgi 설정 건드려야하고 뎁스 늘어서 BASE_DIR 뎁스도 늘려줘야함

