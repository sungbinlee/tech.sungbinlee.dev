---
title: "Django 모델(ORM) 소개"
categories:
  - Django
tags:
  - ORM
  - Python
  - Django
toc: true
toc_sticky: true
toc_label: "Django 모델(ORM)이란?"
toc_icon: "book"
---
## 개요
장고 ORM인 RDBMS에 대한 데이터 관리를 지원하며, 다양한 파이썬 ORM 라이브러리들을 통해 확장 가능합니다.
- 장고 ORM과 다른 ORM 라이브러리
  - RDBMS 기반: Django Models, SQLAlchemy, Orator, Peewee, PonyORM
  - NoSQL 기반: django-mongodb-engine, hot-redis, MongoEngine, PynamoDB 등

### 장고의 강점
- 모델과 폼을 통해 강력한 데이터 모델링과 입력폼 기능을 제공합니다. 직접 SQL을 실행할 수 있지만, ORM 사용을 권장하여 SQL Injection을 방지할 수 있습니다.

### Django Model 설계
- 데이터베이스 테이블과 파이썬 클래스를 1:1로 매핑합니다.
- 모델 클래스명은 단수형(PascalCase)으로 지정되며, DB 테이블 필드와 일치해야 합니다.

모델을 만들기 전에, 서비스에 맞게 데이터베이스 설계가 필수입니다.

```python
class Post(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

## 모델 활용 순서

### 장고 모델을 통한 데이터베이스 형상 관리
1. 모델 클래스 작성
2. `makemigrations` 명령으로 모델 클래스로부터 마이그레이션 파일 생성
3. `migrate` 명령으로 마이그레이션 파일을 데이터베이스에 적용
4. 모델을 활용하여 데이터 처리

### 장고 외부에서 데이터베이스 형상 관리
1. 데이터베이스로부터 모델 클래스 소스 생성 -> `inspectdb` 명령
2. 모델 활용

## 모델명과 DB 테이블명

- DB 테이블명은 주로 "앱이름_모델명"의 형태를 따릅니다.
    - 예시:
        - blog 앱
            - Post 모델 -> "blog_post"
            - Comment 모델 -> "blog_comment"
        - shop 앱
            - Item 모델 -> "shop_item"
            - Review 모델 -> "shop_review"

## 기본 지원되는 모델 필드 타입

자세한 정보는 [Django 공식 문서](https://docs.djangoproject.com/en/4.2/ref/models/fields/#field-types)를 참고하세요.

- Primary Key: AutoField, BigAutoField
- 문자열: CharField, TextField, SlugField
- 날짜/시간: DateField, TimeField, DateTimeField, DurationField
- 참/거짓: BooleanField, NullBooleanField
- 숫자: IntegerField, SmallIntegerField, PositiveIntegerField, PositiveSmallIntegerField, BigIntegerField, DecimalField, FloatField
- 파일: BinaryField, FileField, ImageField, FilePathField
- 이메일: EmailField
- URL: URLField
- UUID: UUIDField
- IP: GenericIPAddressField

### 자주 사용되는 필드 옵션
- `blank`: 필드에 빈 값을 허용할지 여부를 결정합니다.
- `null`: DB에 NULL 값을 허용할지 여부를 결정합니다.
- `db_index`: 필드에 대한 DB 인덱스 생성 여부를 결정합니다.
- `unique`: 필드 값의 고유함을 보장합니다.
- `choices`: 선택할 수 있는 값들의 리스트를 정의합니다.
- `validators`: 유효성 검사를 수행하는 함수들의 리스트를 정의합니다.
- `verbose_name`: 필드의 사람이 읽기 쉬운 이름을 지정합니다.
- `help_text`: 필드를 설명하는 도움말 텍스트를 지정합니다.

## 모델 개발의 중요성

모델은 장고 개발의 핵심입니다. 데이터베이스 구조에 따라 필드 타입을 정확하게 지정하여 입력값 오류를 방지하는 것이 중요합니다.

- `blank`/`null` 지정은 최소화해야 합니다.
- 다양하고 타이트한 `validators`를 지정하는 것이 좋습니다.
- 프론트엔드에서의 유효성 검사는 사용자 편의를 위해 수행되지만, 백엔드에서의 검사는 필수입니다.
- 직접적인 유효성 로직을 만들기보다는 장고의 Form/Model, DRF의 Serializer 등 이미 구성된 기능을 활용하는 것이 권장됩니다.
