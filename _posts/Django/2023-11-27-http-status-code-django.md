---
title: "HTTP 상태 코드와 Django에서의 활용"
categories:
  - Django
tags:
  - Python
  - Django
  - HTTP Protocol
  - HTTP Status Code
  - Error Handling
  - Backend
toc: true
toc_sticky: true
toc_label: "Django 튜토리얼"
toc_icon: "book"
---

HTTP 프로토콜은 클라이언트와 서버 간 통신에서 상태 코드를 사용해 요청의 성공, 실패 및 그 이유를 나타냅니다. 장고(Django)와 같은 웹 프레임워크에서는 이러한 상태 코드를 이용하여 적절한 응답을 제공합니다.


## 대표적인 상태 코드와 예시

### 200번대: 성공
- **200 (OK):** 클라이언트 요청이 성공적으로 처리되었음을 의미합니다.
  
  *예시:*
  - `HttpResponse`를 사용하여 페이지를 렌더링하는 경우
    ```python
    from django.http import HttpResponse

    def my_view(request):
        # ...
        return HttpResponse("This is a success message")
    ```

  - `JsonResponse`로 JSON 데이터를 반환하는 경우
    ```python
    from django.http import JsonResponse

    def my_api(request):
        # ...
        data = {'key': 'value'}
        return JsonResponse(data)
    ```

### 300번대: 추가 조치 필요
- **301 (Moved Permanently):** 요청한 리소스가 영구적으로 새 위치로 이동했음을 의미합니다.
- **302 (Found/Temporary Redirect):** 요청한 리소스가 일시적으로 다른 위치에 있음을 의미하며, 향후에는 원래 위치를 사용해야 함을 알려줍니다.
  
  *예시:*
  - `redirect()` 함수를 사용하여 다른 URL로 임시 이동하는 경우
    ```python
    from django.shortcuts import redirect

    def my_page(request):
        # ...
        return redirect('/new_location/')
    ```

### 400번대: 클라이언트 오류
- **400 (Bad Request):** 잘못된 요청을 보냈거나 서버가 이해할 수 없는 요청을 받았음을 나타냅니다.
- **401 (Unauthorized):** 권한이 없는 상태에서 보호된 리소스에 접근하려고 할 때 사용됩니다.
- **403 (Forbidden):** 필요한 권한이 없어 요청을 거부했음을 나타냅니다.
- **404 (Not Found):** 요청한 리소스를 서버에서 찾을 수 없음을 의미합니다.
- **405 (Method Not Allowed):** 지원되지 않는 HTTP 메소드를 사용하여 요청했을 때 반환됩니다.
  
  *예시:*
  - 직접 예외를 처리하는 방식
    ```python
    from django.http import Http404
    from .models import Book
    
    def get_book(request, book_id):
        try:
            book = Book.objects.get(pk=book_id)
        except Book.DoesNotExist:
            raise Http404("Book does not exist")
    ```
  - `get_object_or_404`를 사용하여 객체를 가져오는데 해당하는 객체가 없는 경우 `404` 반환
    ```python
    from django.shortcuts import get_object_or_404

    def get_book(request, book_id):
        book = get_object_or_404(Book, pk=book_id)
    ```


### 500번대: 서버 오류
- **500 (Internal Server Error):** 서버가 요청을 처리하는 동안 예기치 못한 오류가 발생했음을 나타냅니다.

**서버 오류 발생 시의 예시 및 처리 방법:**

**1. 예상치 못한 예외 처리**
```python
from django.http import HttpResponseServerError
import logging

def my_view(request):
    try:
        # 서버가 처리하는 과정에서 예외가 발생할 수 있는 코드
    except SpecificException as e:
        # 특정 예외에 대한 처리
        return HttpResponseServerError("Specific error occurred")

    except AnotherException as e:
        # 다른 예외에 대한 처리
        return HttpResponseServerError("Another error occurred")

    except Exception as e:
        # 모든 예상치 못한 예외에 대한 처리 및 로깅
        logging.error("Unexpected error: %s", str(e))
        return HttpResponseServerError("An unexpected error occurred")
```

**2. 예외 처리의 중요성**
- 예상된 예외를 세밀하게 처리하고, 예상치 못한 예외에 대해서는 500 상태 코드를 반환하며 로그를 남겨 나중에 대응할 수 있도록 합니다.
- 개발 시에는 타이트하게 예외를 처리하는 것이 중요합니다.
