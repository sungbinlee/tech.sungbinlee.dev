---
title: "Python 변수, 스코프, 네임스페이스 이해하기"
categories:
  - Python
tags:
  - Python
  - Scope
  - Variable
  - Namespace
toc: true
toc_sticky: true
---

Python에서는 변수를 코드의 여러 위치에서 정의할 수 있다. 이 글에서는 파이썬에서 전역 변수, 지역 변수, 스코프, 네임스페이스가 어떻게 동작하는지 순서대로 정리하고자 한다.

## 변수와 객체 

### 전역 변수와 지역 변수

함수 바깥에 정의된 변수는 전역 변수다.

```python
a = 99

def first():
    x = 2

def second():
    ...
````

`a`는 전역 변수이므로 두 함수 모두에서 접근 가능하다. 함수 내부에서 정의된 변수는 지역 변수다. `x`는 `first()`의 지역 변수이므로 함수 외부에서는 접근 불가다.

**Python에서는 모든 것이 객체다.** 함수 역시 객체다. 위 코드에서 변수 `x`는 실제로:

* 메모리에 어딘가에 `10` 객체가 생성된다
* 이름 `x`가 그 객체에 바인딩된다

**변수는 값을 저장하는 상자가 아니라 객체를 가리키는 이름이다.**

## 스코프(Scope)

### 스코프의 종류

변수가 정의된 코드 영역을 스코프라고 한다.

* Global Scope → 모듈(파일) 전체
* Local Scope → 함수 내부
* Built-in Scope → 어디서든 접근 가능

![scopes](/assets/images/python/scopes-and-namespaces/scope.png)

* `a`는 전역 스코프
* `prefix`는 지역 스코프

## 이름 탐색 규칙

### LEGB 규칙

Python의 이름 탐색 순서:

1. Local
2. Enclosing
3. Global
4. Built-in

* `prefix`는 Local에서 발견된다
* `a`는 Global에서 발견된다
* `print`는 Built-in에서 발견된다

### Built-in 이름 가리기

Built-in 이름을 재정의하면 기존 함수가 가려진다.

```python
def print():
    pass
```

이후 `print()` 호출 시 새로 정의한 함수가 사용된다.

## 변수 할당과 오류

### 지역 변수 참조 전 할당 오류

```python
a = 10

def my_func():
    print(a)
    a = 35
```

오류가 발생한다.

이유:

* 함수 내부에 `a = 35` 존재
* Python이 `a`를 지역 변수로 판단
* `print(a)` 실행 시 값이 아직 없음

결과적으로 `UnboundLocalError`가 발생한다.

### global 키워드

함수 내부에서 전역 변수를 수정하려면 `global`을 사용한다.

```python
a = 10

def my_func():
    global a
    a = 35
```

전역 변수 `a`가 수정된다.
함수 종료 후에도 값이 유지된다.

### global로 전역 변수 생성

```python
def my_func():
    global a
    a = 100
```

전역 스코프에 `a`가 생성된다.

## 네임스페이스(Namespace)

### 네임스페이스 개념

변수 바인딩은 네임스페이스에 저장된다.
Global과 Local 네임스페이스는 딕셔너리로 구현된다.

코드 실행 시 전역 네임스페이스에는 다음 항목이 생성된다.

* 전역 변수
* 함수 객체

함수 호출 시 지역 네임스페이스가 새로 생성된다.

### globals()와 locals()

#### 전역 네임스페이스 접근:

```python
a = 35

def my_func(inp_param):
    local_var = "some string"
    print(inp_param, local_var)
```

아래 처럼 `global()` 빌트인 함수로 전역 네임 스페이스 딕셔너리를 다룰 수 있다.

```python
>>> print(globals())
{'__name__': '__main__',
 '__doc__': None,
 ....
 'a': 35,
 'my_func': <function my_func at 0xf8b2>
}

>>> global()['a']
35

>>> globals()['my_func']
<function my_func at 0xf8b2>

>>> globals()['my_func']("this is")
this is some string
```


#### 지역 네임스페이스 접근:

```python
a = 35

def my_func(inp_param):
    local_var = "some string"
    print(locals())
```

마찬가지로 지역 네임스페이스 또한 `locals()` 빌트인 함수로 다룰 수 있다.

```python
>>> my_func("this is")
{  'inp_param': 'this is',
   'local_var': 'some string'
}
```

## Enclosing Scope

### 중첩 함수

```python
def outer():
    x = 10

    def inner():
        print(x)
```

`inner()`는 `outer()`의 변수에 접근 가능하다.

### 내부 함수에서 변수 수정

```python
def outer():
    x = 10

    def inner():
        x = 20
```

`inner()`의 `x`는 새로운 지역 변수다.
`outer()`의 `x`는 변경되지 않는다.

### global 사용 시 동작

```python
def outer():
    x = 10

    def inner():
        global x
        x = 20
```

`global x`는 전역 변수로 처리된다.
`outer()`의 `x`와는 무관하다.

### nonlocal 키워드

Enclosing Scope 변수를 수정하려면 `nonlocal`을 사용한다.

```python
def outer():
    x = 10

    def inner():
        nonlocal x
        x = 20
```

`outer()`의 `x`가 변경된다.

### nonlocal 탐색 범위

* 현재 Local 제외
* Enclosing Scope만 탐색
* Global Scope는 탐색하지 않음

### nonlocal 오류

```python
a = 35

def outer():
    def inner():
        nonlocal a
```

Enclosing Scope에 `a`가 없으므로 오류가 발생한다.

### nonlocal과 global 체이닝

```python
a = 35

def outer():
    global a
    a = 25
    def inner1():
        def inner():
            nonlocal a
            a = 30
        inner2()
    inner1()
```

nonlocal은 Global Scope까지 올라가지 않으므로 실패한다.
