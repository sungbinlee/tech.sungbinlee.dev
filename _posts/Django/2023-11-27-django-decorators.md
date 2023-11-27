---
title: "Django Decorators: 활용과 기능"
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

장고(Django)에서의 Decorators(장식자)는 함수를 감싸는 방식으로, 특정 기능이나 행동을 추가하거나 조정하는 데 사용됩니다. 이들은 코드 재사용과 유지보수를 쉽게 만들어주며, 장고에서 제공하는 여러 가지 내장 Decorators를 사용하여 웹 애플리케이션의 보안, HTTP 메소드 제한, 권한 관리 등을 손쉽게 다룰 수 있습니다.

## 기본적인 Decorators 사용 예시

### `login_required`

```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import render

@login_required
def protected_view(request):
    return render(request, "myapp/secret.html")
```

위의 예제는 사용자가 로그인했는지 확인하고, 로그인하지 않은 경우 로그인 페이지로 리디렉션하는 기능을 가진 `@login_required` Decorator를 사용합니다.

### `user_passes_test`와 `permission_required`

```python
from django.contrib.auth.decorators import user_passes_test, permission_required
from django.shortcuts import render

def custom_check(user):
    # 사용자 정의 함수로 사용자 검증 로직을 정의할 수 있습니다.
    return user.has_perm('some_permission')

@user_passes_test(custom_check, login_url='/login/')
def custom_protected_view(request):
    return render(request, "myapp/custom_secret.html")

@permission_required('some_permission', login_url='/login/')
def permission_protected_view(request):
    return render(request, "myapp/permission_secret.html")
```

`user_passes_test`는 사용자 정의 함수를 통해 사용자를 검증하고, `permission_required`는 특정 권한을 요구하여 접근을 제어합니다. 두 Decorators 모두 조건을 만족하지 않을 경우 지정된 로그인 페이지로 리디렉션합니다.

### HTTP Method 관련 Decorators

```python
from django.views.decorators.http import require_http_methods, require_GET, require_POST, require_safe
from django.http import HttpResponseNotAllowed

@require_http_methods(["GET", "POST"])
def multi_method_view(request):
    # GET 또는 POST 요청만 허용합니다.
    # 다른 메소드의 요청이 들어오면 HttpResponseNotAllowed 에러를 반환합니다.
    return render(request, "myapp/multi_method.html")

@require_GET
def get_only_view(request):
    # GET 요청만 허용합니다.
    return render(request, "myapp/get_only.html")

@require_POST
def post_only_view(request):
    # POST 요청만 허용합니다.
    return render(request, "myapp/post_only.html")

@require_safe
def safe_methods_view(request):
    # GET, HEAD, OPTIONS 요청만 허용합니다.
    return render(request, "myapp/safe_methods.html")
```

위의 예시들은 HTTP 요청 메소드를 제한하거나 안전한 메소드만을 허용하는 Decorators를 보여줍니다. 요청이 허용되지 않는 경우 적절한 응답이 반환됩니다.

물론, 아래에 블로그 글을 작성한 것과 비슷한 내용을 Markdown 형식으로 제공해 드리겠습니다:

---

## 클래스 Decorator와 LoginRequiredMixin 활용하기

클래스 Decorators는 클래스 전체를 변경하거나 확장하는 데 사용되며, 장고(Django)에서 주로 뷰(View)를 조작하거나 확장하는 데 활용됩니다. 특히 `@method_decorator`를 사용하여 클래스의 메소드를 데코레이팅하거나, 믹스인(mixin) 패턴을 통해 기존 클래스에 기능을 추가하는 등의 방법으로 사용됩니다.

### 클래스 데코레이터를 활용한 뷰(View) 보호하기

```python
from django.utils.decorators import method_decorator
from django.contrib.auth.decorators import login_required
from django.views import View
from django.http import HttpResponse

@method_decorator(login_required, name='dispatch')
class ProtectedView(View):
    def get(self, request):
        # 보호되는 뷰의 로직을 작성합니다.
        return HttpResponse("This is a protected view.")
```

위의 예시는 `@method_decorator`를 사용하여 `ProtectedView` 클래스에 `login_required` 데코레이터를 적용하는 것을 보여줍니다. 이를 통해 `dispatch` 메소드를 통과하는 모든 요청에 로그인이 요구됩니다.

### LoginRequiredMixin을 활용한 로그인 요구 설정

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views import View
from django.http import HttpResponse

class ProtectedView(LoginRequiredMixin, View):
    def get(self, request):
        # 보호되는 뷰의 로직을 작성합니다.
        return HttpResponse("This is a protected view.")
```

`LoginRequiredMixin`을 사용하는 경우, 해당 클래스를 상속함으로써 로그인이 필요한 뷰를 간단히 구현할 수 있습니다. 이 방식은 클래스 데코레이터보다 더 간편하며, 특정 뷰에 로그인이 필요한 경우에 적합합니다.

클래스 데코레이터와 `LoginRequiredMixin`은 장고에서 뷰 보호 및 권한 관리를 위한 강력한 기능들로, 코드를 간결하게 작성하고 중복을 피하는 데 큰 도움을 줍니다.
