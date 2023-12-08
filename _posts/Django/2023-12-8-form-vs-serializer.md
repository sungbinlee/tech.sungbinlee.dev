---
title: "Django의 Form과 Serializer 비교"
categories:
  - Django
tags:
  - Python
  - Django
  - Form
  - Serializer
  - Backend
toc: true
toc_sticky: true
toc_label: "ORM N+1"
toc_icon: "book"
---

## 특징
### Form / ModelForm
- **기능:** HTML 입력 폼으로부터의 입력 유효성 검사.
- **대부분의 사용처:** 생성/수정 작업에서 주로 활용되며, 주로 장고 어드민에서 사용됨.
- **사용처:** CreateView/UpdateView CBV를 통한 뷰 처리로 주로 단일 뷰에서 작동함.

### Serializer / ModelSerializer
- **기능:** 데이터 변환과 직렬화 지원(JSON 형식 등).
- **사용처:** List/Create 및 특정 레코드의 Retrieve/Edit/Delete 등에 활용됨.
- **사용처:** APIView를 통한 단일 뷰 처리 및 ViewSet을 통한 두 개의 뷰 및 두 개의 URL 처리에 활용됨.

## 주된 호출 주체

### Form
- **일반적 사용자:** 웹 브라우저를 통해 사용됨.
- **제출 방법:** HTML Form 제출 또는 JavaScript에 의한 비동기 호출.

### Serializer
- **사용자:** 다양한 클라이언트로부터 데이터 중심의 http(s) 요청을 받음.

## 클래스 정의 비교 예시

- **Form/ModelForm:**
  
```python
from django import forms

class PostForm(forms.Form):
    email = forms.EmailField()
    content = forms.CharField(max_length=200)
    created_at = forms.DateTimeField()

class PostModelForm(forms.ModelForm):
    class Meta:
        model = post
        fields = '__all__'
```

- **Serializer/ModelSerializer:**
  
```python
from rest_framework import serializers

class PostSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created_at = serializers.DateTimeField()
  
class PostModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = '__all__'
```

###유효성 검사 수행 시점 - Form vs Serializer

- **Form:** 
    - `is_valid()` 메서드 호출 시, 유효성 검사 결과를 불리언 값인 `True` 또는 `False`로 반환합니다. 유효하지 않은 경우 `False`를 반환합니다.

- **Serializer:** 
    - `is_valid()` 메서드 호출 시, 유효성 검사 결과를 확인하여 유효하지 않은 경우 `ValidationError` 예외를 발생시킵니다. 이 예외는 유효하지 않은 데이터와 함께 발생하며, 세부 정보를 담고 있습니다.

#### 커스텀 유효성 검사 루틴 - clean_* vs validate_* 예시

- **Form:** 
    - `clean_<field_name>()` 메서드를 통해 필드별로 커스텀 유효성 검사를 수행합니다. 

- **Serializer:** 
    - `validate_<field_name>()` 메서드를 통해 필드별로 커스텀 유효성 검사를 수행합니다.
