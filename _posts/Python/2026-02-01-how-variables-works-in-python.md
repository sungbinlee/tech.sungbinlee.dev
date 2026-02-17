---
title: "파이썬 변수와 객체의 동작 원리"
categories:
  - Python
tags:
  - Python
  - Variables
  - Memory
  - Interning
toc: true
toc_sticky: true
toc_label: "파이썬 변수와 객체"
toc_icon: "book"
---

파이썬을 배우다 보면 이런 순간을 자주 마주한다.
- 변수에 값을 넣었는데 예상과 다르게 동작한다
- 리스트를 복사했는데 함께 변경된다
- `==` 와 `is` 의 결과가 다르다
- 함수의 기본값이 이상하게 누적된다

이런 혼란의 대부분은 파이썬의 변수 모델을 C 언어 방식으로 이해했기 때문이다.

파이썬은 아래와 같은 철학을 지닌 객체지향 언어이다

- 변수에 값이 저장되지 않는다
- 모든 것이 객체다
- 변수는 객체를 가르키는 이름이다

## C 언어에서의 변수 동작 방식

C 언어에서는 변수를 생성하고 `=` 연산자로 값을 대입하면 해당 함수의 스택 메모리에 값이 저장된다.

- 변수 타입이 메모리 크기를 결정한다(int -> 4 bytes, float -> 8 bytes)
- 변수는 값을 담는 컨테이너 역할을 한다
- 타입 정보는 변수에 연결된다

![Variables in C](/assets/images/python/how-variable-works/c_variables.png)

- 기존 변수에 새로운 값 대입: 동일한 메모리 위치의 값이 교체된다
- 다른 변수에 대입: 새로운 메모리 할당 + 값 복사

## 파이썬에서의 변수 동작 방식
 
파이썬(CPython)에서는:

- 모든것이 객체(object)
- 값은 힙에 객체로 생성된다
- 변수는 객체를 가르키는 참조(reference)다

객체는 내부적으로 다음 정보를 가진다.

- 타입 정보 (int, float, string 등)
- 참조 카운트 (reference count)
- 실제 값

객체는 한번 생성되면 메모리 주소가 고정된다.

![Variables in Python](/assets/images/python/how-variable-works/python_variables.png)

- 변수 `x`는 객체를 참조하며 참조 카운트는 1이 된다.
- 재할당 시에는 새로운 객체 생성
- `x` -> 새 객체 참조
- 이전 객체 참조 카운트 감소

**참조 카운트가 0이되면 가비지 컬렉션 대상이 된다.**

- 다른 변수에 대입시에는 새로운 객체가 생성되지 않는다
- 동일 객체에 대한 참조만 추가된다

> 파이썬 변수는 값을 저장하는 상자가 아니라 객체에 붙는 이름(라벨)이다.

## 리스트는 내부적으로 어떻게 표현될까?

리스트는 여러 값을 저장하는 컨테이너다.

![List Vriable in Python](/assets/images/python/how-variable-works/python_list.png)

리스트는 내부적으로:

- PyVarObject 구조 사용
- Size가 있음
- 값이 아니라 요소 저장공간을 가르키는 포인터를 저장

이 구조 덕분에:

- 서로 다른 타입 저장 가능
- 요소 추가시에는 새로운 객체 생성
- 참조 배열에 포인터 추가
- Size 증가

### 리스트 용량의 확장

리스트의 내부 배열은 크기가 고정되어있다.

CPython 내부에서 사이즈가 꽉 차게 되면

1. 더 큰 메모리 할당 (보통 2배)
2. 기존 참조 복사
3. 포인터 업데이트
4. 이전 배열 해제

> 리스트 객체 자체 주소는 변경되지 않는다 (Mutable)

## Interning (객체 재사용)

파이썬은 메모리를 효율적으로 쓰기위해서 일부 객체를 재사용 한다.

```python
x = None
y = None
hex(id(x)) # 0x105ce5228
hex(id(y)) # 0x105ce5228
````

동일한 `None` 객체 참조가 된다.

또한 `-5 ~ 256` 범위의 작은 정수 캐싱 또한 지원한다.

```python
a = 100
b = 100
hex(id(a)) # 0x105dc7120
hex(id(b)) # 0x105dc7120

# 범위 밖은 새로운 객체
a = 257
b = 257

hex(id(a)) # 0x1048e62f0
hex(id(b)) # 0x1048e6610
```

문자열같은 경우에는
식별자 형태 문자열:

```python
a = "hello"
b = "hello"

hex(id(a)) # 0x1048a6730
hex(id(b)) # 0x1048a6730
```

-> intern이 되서 객체를 재사용 하게된다

공백 포함 문자열:

```python
a = "hello world"
b = "hello world"

hex(id(a)) # 0x1048bd8f0
hex(id(b)) # 0x104c23430
```

-> 새 객체를 반환한다.

## 동등성 비교 연산자

파이썬에는 두가지 비교 방식이 존재한다.

| 연산자  | 의미        |
| ---- | --------- |
| `==` | 값 비교      |
| `is` | 객체 동일성 비교 |

```python
x = [1, 2, 3]
y = [1, 2, 3]

x == y # True
x is y # False
```

![Equality1](/assets/images/python/how-variable-works/equality1.png)

> 내용은 같지만 다른 객체

```python
x = [1, 2, 3]
y = x

x == y # True
x is y # True
```

![Equality2](/assets/images/python/how-variable-works/equality2.png)

> 같은 객체

## 함수로 전달되는 변수

파이썬에서 변수가 함수에 전달될 때 객체의 참조(reference)가 값으로 전달된다.

```python
def assign_new_list(my_list):
    my_list = [42, 34, 27]

nums = [1, 2, 3]
assign_new_list(nums)
print(nums) # [1, 2, 3]
```

* 함수는 기본적으로 다른 스코프를 가진다
* 최초의 `my_list`는 함수의 스코프에서 생성되며 `nums`를 참조하고 있다
* `my_list` 에 새로운 값이 할당될때 메모리에서 새로운 객체가 생성되고 `my_list`는 새로운 객체를 참조하게 된다.(함수 종료시 가비지 콜렉티드 된다: 참조 카운트 0)

![pass by ref1](/assets/images/python/how-variable-works/passby_ref1.png)

```python
def add_ten_to_list(my_list):
    my_list.append(10)

nums = [1, 2, 3]
add_ten_to_list(nums)
print(nums) # [1, 2, 3, 10]
```

* 객체의 참조가 값으로 전달된다
* `my_list`와 `nums`는 동일한 리스트 객체를 참조
* .append()는 객체 자체를 수정하는 연산
* 따라서 원본 리스트가 변경된다

![pass by ref2](/assets/images/python/how-variable-works/passby_ref2.png)

파이썬에서 함수에 변수가 전달되는 방식을 이해하지 못한다면 객체가 예상치 못하게 변경되는 버그를 만들기 쉽다.

## Mutable 기본값 파라미터

파이썬은 함수를 정의할때 매개변수에 기본값을 정의할 수 있다.

```python
def add_two_to_list(my_list=[]):
    my_list.append(2)
    return my_list

first = add_tow_to_list()
print(first) # [2]

second = add_tow_to_list()
print(second) # [2, 2]
```

![Mutable default parameter](/assets/images/python/how-variable-works/mutable_default_parameter.png)

앞서 말했듯이 파이썬에서는 모든 것이 객체다. 함수또한 객체이다. 기본값은 함수 정의 시점에 단 한 번만 생성된다.

위 코드의 예제를 보면 겉보기에는 매번 새로운 빈 리스트가 생성될 것 처럼 보인다. 하지만 실제 동작은 다르다.

함수 정의가 실행되는 순간:

1. 빈 리스트 객체 [] 생성
2. 함수 객체 생성
3. 기본값 리스트가 함수 객체 내부에 저장

함수 객체는 기본값을 내부 속성에 저장하기 때문애 아래와 같이 확인해 볼 수 있다.

```python
add_two_to_list.__defaults__
# ([], )
```

해당 튜플 안의 리스트 객체가 모든 호출에서 재사용 된다. 그래서 append 결과가 계속 누적된다.

```python
def add_two_to_list(my_list=None):
    # my_list = my_list or []
    if not my_list:
        my_list = []
    
    my_list.append(2)
    return my_list
```

위와 같은 상황을 방지하려면 Mutable한 객체에 대해서 `None` 값을 기본 파라미터로 지정하고 함수 내부에서 확인후 새로운 객체를 생성해주면 된다.

## `+=` 연산자

`+=` 연산자(그리고 `-=`, `*=`, `/=` 등 복합 대입 연산자)에는 일반 대입과 다른 특별한 동작이 존재한다.

### 동일 객체 참조

```python
x = [1, 2]
x_copy = x

hex(id(x)), hex(id(x_copy))
# ('0x104c1dc00', '0x104c1dc00')
```

두 변수는 같은 리스트 객체를 참조한다.

### 일반 `+` 연산

```python
x = x + [3, 4]   # [1, 2, 3, 4]

hex(id(x))       # 0x104c21e00
x_copy           # [1, 2]
```

동작 순서:

1. 오른쪽 항 `[3, 4]` 평가 (파이썬은 항상 오른쪽 항 부터 연산한다.)
2. 기존 리스트 복사
3. 새 리스트 생성
4. x가 새 객체를 참조

### `+=` 연산

```python
x += [3, 4]
hex(id(x))       # (이전과 동일)
x_copy           # [1, 2, 3, 4]
```

✔ 기존 리스트 객체 직접 수정 (in-place)

### 내부적으로 일어나는 일

```python
x += [3, 4]
```

는 개념적으로 다음과 같다:

```python
x = x.__iadd__([3, 4])
# list.__iadd__(self, iterable)
```

실제 호출 형태:

```python
x = list.__iadd__(x, [3, 4])
```

동작 과정:

1. `__iadd__()` 호출
2. 리스트 내부 상태 변경
3. 수정된 자기 자신(self) 반환
4. x에 다시 대입

## Tuple 내부의 Mutable 객체

Tuple은 `immutable`이다.

```python
t = 1, [2,3]

t[1] = 4, 5 # 에러가 발생한다
```

하지만 내부 요소가 `mutable`한 객체일 경우에는 Tuple 에 대한 변경이 아니고 리스트 객체에 대한 변경이므로 값을 변경할 수 있다.

```python
t[1].append(4)
```

`+=` 연산자를 사용한 경우에는 예기치 못한 오류를 볼 수 있다. 아래처럼 예외 오류가 났음에도 불구하고 값을 확인해보면 리스트 값은 변경된 것을 볼 수 있다.

![Mutable elements in tuple](/assets/images/python/how-variable-works/mutable_elements_tuple.png)

동작 순서:

1. 내부적으로 `list.__iadd__()` 실행
2. 리스트 변경 (오른쪽 항 먼저 연산)
3. Tuple 재할당 시도 → 예외 발생

> 결론적으로 주소값은 변경되지 않았지만 immutable 한 객체에 다시 재할당을 하려했기 때문에 해당 예외가 발생한것이다.
