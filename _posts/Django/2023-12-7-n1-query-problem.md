---
title: "장고 ORM에서의 고전적인 N+1 문제"
categories:
  - Django
tags:
  - Python
  - Django
  - ORM
  - N+1
  - Backend
toc: true
toc_sticky: true
toc_label: "ORM N+1"
toc_icon: "book"
---

ORM(객체-관계 매핑)은 개발자가 데이터베이스와 상호작용하는 편리하고 추상화된 방법을 제공하지만, 종종 성능 문제에 직면할 수 있는 N + 1 문제가 있습니다. 이 문제는 데이터베이스 쿼리를 효율적으로 처리하지 못해 성능 저하로 이어질 수 있습니다. 이러한 문제를 해결하기 위해 Django ORM에서는 `select_related`와 `prefetch_related`와 같은 기능을 제공합니다.

### N + 1 문제란?

N + 1 문제는 ORM에서 객체 간 관계를 다룰 때 발생하는 성능 문제입니다. 예를 들어, 작가(`Author`)와 그들의 책(`Book`)이 일대다 관계로 연결되어 있을 때, 모든 작가와 그들의 책을 검색하려는 시나리오를 살펴보겠습니다.

```python
authors = Author.objects.all()  # 모든 작가 검색
for author in authors:
    books = author.books.all()  # 각 작가의 책 검색
    for book in books:
        print(book.title)
```

이 코드는 각 작가마다 책을 가져오기 위해 별도의 쿼리를 실행하여 N + 1 개의 쿼리를 발생시킵니다.

### 해결 방법: select_related와 prefetch_related

#### select_related 사용하기

`select_related`는 조인을 사용하여 관련된 객체를 한 번의 쿼리로 가져옵니다.

```python
authors = Author.objects.select_related('books').all()  # 관련된 책을 포함하여 작가 검색
for author in authors:
    books = author.books.all()
    for book in books:
        print(book.title)
```

#### prefetch_related 사용하기

`prefetch_related`는 별도의 추가 쿼리를 실행하여 연결된 객체를 가져온 후 메모리에 적재하여 성능을 최적화합니다.

```python
authors = Author.objects.prefetch_related('books').all()
for author in authors:
    books = author.books.all()
    for book in books:
        print(book.title)
```

이러한 메서드를 사용하여 데이터베이스 쿼리를 최적화하면 N + 1 문제를 해결하고 성능을 향상시킬 수 있습니다. `select_related`와 `prefetch_related`는 ORM에서 데이터베이스 쿼리를 효율적으로 처리하는 데 중요한 도구입니다.