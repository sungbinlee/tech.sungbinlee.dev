---
title: "Effective Python: 좋은 파이썬 코드를 작성하는 125가지 방법"
categories:
  - Python
tags:
  - Python
  - Effective Python
toc: true
toc_sticky: true
toc_icon: "book"
---

좋은 파이썬 코드를 작성하는 125가지 방법

## 1. Pythonic Thinking

### Item 1: 어떤 파이썬 버전을 쓰는지 알아라

* 신규 프로젝트는 `Python 3`를 사용해라.
* 커맨드라인에서 실행하는 `python`/`python3`가 “네가 의도한 파이썬(가상환경)”인지, “시스템 파이썬”인지부터 확인해라.
* `Python 2`는 공식적으로 코어 개발자들에게 더이상 메인테인 받지 않는다.

### Item 2: `PEP 8` 스타일 가이드를 따라라

* 파이썬 코드를 작성할 때는 항상 `PEP 8`을 따라라.
* 공통 스타일을 공유하면 파이썬 커뮤니티/팀에서 협업이 쉬워진다.
* 일관된 스타일을 유지해서 코드를 수정하기 쉽게 만들어라.
* `Black` 그리고 `pylint` 같은 커뮤니티 툴로 `PEP 8` 준수를 자동화해라.

### Item 3: 파이썬이 항상 컴파일 할때 에러를 감지할거라고 기대하지 마라

* 파이썬은 에러 체크 대부분을 `runtime`으로 미룬다.
* 실행하기 전에는 “당연히 잡힐 것 같은 실수”도 안 잡힐 수 있다고 생각해라.
* `linter`와 `static analysis`로 흔한 에러를 실행 전에 잡아라.

### Item 4: 복잡한 표현(코드) 대신 `helper function`을 작성해라

* 한 줄로 억지로 우겨 넣어서 너무 복잡하게 만들지 마라.
* 복잡한 `expression`은 `helper function`으로 빼라.
* 특히 그 로직을 반복해서 쓸 거면 확실하게 함수로 옮겨라.

### Item 5: `indexing` 말고 `unpacking`을 선호해라

* 시퀀스를 `index`로 덕지덕지 접근하지 마라.
* 가능한 곳에서는 `unpacking`으로 여러 값을 한 번에 받아라.
* `unpacking`으로 시각적 잡음을 줄이고 가독성을 올려라.

### Item 6: 항상 single-element `tuple`은 괄호로 감싸라

* `tuple` 리터럴은 보통 괄호가 optional 이고, trailing comma도 케이스에 따라 들어간다.
* single-element `tuple`은 원소 뒤에 trailing comma가 필수다. 괄호는 있어도 되고 없어도 된다.
* 표현식 끝에 실수로 trailing comma를 붙이면 의미가 single-element `tuple`로 바뀐다. 프로그램을 깨먹을 수 있으니 조심해라.

### Item 7: 간단한 inline 로직이면 `conditional expression`을 고려해라

* 파이썬은 `if`를 표현식 자리에도 넣을 수 있게 `conditional expression`을 제공한다.
* `conditional expression`은 다른 언어의 삼항연산자랑 순서가 다르다. 헷갈리지 않게 조심해라.
* 애매해지거나 처음 보는 사람이 읽기 어려워지면 쓰지 마라.
* 이득이 확실하지 않으면 일반 `if` 문이랑 `helper function`을 써라.

### Item 8: `assignment expression`으로 반복을 막아라

* `assignment expression`은 walrus operator `:=`로 “할당 + 평가”를 한 번에 한다. 반복을 줄여라.
* 큰 표현식 안의 subexpression으로 쓰면 괄호로 감싸라.
* 파이썬에 `switch/case`나 `do/while`은 없지만, 이런 패턴 일부는 `assignment expression`으로 더 명확하게 흉내낼 수 있다.

### Item 9: 흐름 제어에서 구조 분해가 필요하면 `match`를 고려해라. `if`로 충분하면 쓰지 마라

* 단순 `if`를 `match`로 억지로 바꾸지 마라. 실수하기 쉽고 직관적이지 않은 함정이 있다.
* `match`는 `isinstance` 체크 + destructuring을 flow control과 같이 묶어야 할 때 강하다. heterogeneous object graph나 semi-structured data 해석할 때 써라.
* `case pattern`은 `list`/`tuple`/`dict` 같은 built-in 구조랑 사용자 정의 클래스에 쓸 수 있다. 근데 타입마다 semantics가 다르다. 바로 안 보이면 함부로 쓰지 마라.

## 2. Strings and Slicing

### Item 10: `bytes` 와 `str`의 차이를 알아라

* `bytes`는 8-bit 값의 시퀀스고, `str`은 유니코드 코드 포인트의 시퀀스다.
* 네가 다루는 입력이 어떤 문자 시퀀스 타입인지 보장하려면 `helper function`을 써라.
* `bytes`와 `str`은 같이 섞어서 연산하지 마라(예: `>`, `==`, `+`, `%`). 애초에 같이 못 쓰는 경우가 많다.
* 파일에 binary 데이터를 읽고/쓸 때는 항상 바이너리 모드로 열어라(예: `"rb"`, `"wb"`).
* 파일에 유니코드 텍스트를 읽고/쓸 때는 시스템 기본 인코딩을 믿지 마라. `open`에 `encoding=...`을 명시해라.

### Item 11: C-style `%` / `str.format` 말고 `f-string`을 써라

* `%` 포맷팅은 gotcha도 많고 장황하다. 쓰지 마라.
* `str.format`은 포맷 스펙 미니-언어 같은 유용한 개념은 있지만, 결국 `%`의 실수를 반복한다. 피하는 게 낫다.
* `f-string`은 문자열에 값을 넣는 최신 문법이고, `%` 방식의 큰 문제를 해결한다.
* `f-string`은 짧고 강력하다. 포맷 스펙 안에 임의의 파이썬 `expression`을 바로 넣을 수 있다.

### Item 12: 객체 출력할 때 `repr` vs `str` 차이를 알아라

* built-in 타입에 `print`를 호출하면 사람이 읽기 쉬운 문자열(`str`)이 나온다. 타입 정보 같은 건 숨겨질 수 있다.
* built-in 타입에 `repr`를 호출하면 “출력 가능한 표현”이 나온다. 경우에 따라 `repr` 문자열은 `eval`로 다시 원래 값으로 복구될 수도 있다.
* 포맷 문자열에서 `%s`는 `str`처럼 사람이 읽기 쉬운 문자열을 만든다. `%r`은 `repr`처럼 출력 가능한 표현을 만든다. `f-string`은 기본이 `str`이고, `!r`을 붙이면 `repr`로 바뀐다.
* 커스텀 클래스는 `__repr__`과 `__str__`을 정의해라. 디버깅이 쉬워지고, 사람 대상 UI/로그에 객체를 붙이기도 편해진다.

### Item 13: 리스트에서는 암시적 문자열 결합 말고 명시적으로 붙여라

* 파이썬은 문자열 리터럴 두 개가 코드에서 붙어 있으면 `+`가 있는 것처럼 자동으로 합친다(C 언어 스타일과 비슷하다).
* `list`/`tuple` 리터럴 안에서는 암시적 결합을 피하라. 작성자 의도가 애매해져서 읽는 사람이 헷갈린다. `+`로 명시적으로 붙여라.
* 함수 호출에서는 positional argument가 1개이고 keyword argument만 추가되는 형태라면 암시적 결합을 써도 괜찮다.
* positional argument가 여러 개라면 암시적 결합을 쓰지 마라. `+`로 명시적으로 붙여라.

### Item 14: 시퀀스 슬라이싱을 제대로 알아라

* 슬라이싱을 쓸 때 불필요하게 장황하게 쓰지 마라. 시작 인덱스에 `0`을 넣거나, 끝 인덱스에 시퀀스 길이를 넣지 마라.
* 슬라이싱은 범위를 넘어가는 인덱스에 관대하다. 앞/뒤 경계 슬라이스를 쉽게 표현해라(예: `a[:20]`, `a[-20:]`).
* `list` 슬라이스에 값을 할당하면 원본 시퀀스의 해당 구간이 통째로 교체된다. 길이가 달라도 그대로 치환된다.

### Item 15: 한 표현식에서 striding과 slicing을 같이 쓰지 마라

* 시작/끝/stride를 한 번에 지정하는 슬라이스는 읽기 어렵고 혼란스럽다.
* stride가 필요하면 start/end 없이 양수의 stride만 쓰는 쪽을 선호해라. 음수 stride는 가능하면 피하라.
* start/end/stride를 한 번에 써야 한다면 두 단계로 나눠라(먼저 stride로 뽑고, 그 다음 slice로 자르기). 아니면 `itertools`의 `islice`를 써라.

### Item 16: slicing보다 catch-all `unpacking`을 선호해라

* `unpacking`에는 starred expression을 넣을 수 있다. 남는 값들을 한 번에 `list`로 받아라.
* starred expression은 패턴 어디에나 둘 수 있다. 결과는 항상 0개 이상 값을 담는 `list`가 된다.
* 리스트를 겹치지 않는 조각으로 나눌 때는, slicing과 indexing을 여러 줄로 쓰지 마라. catch-all `unpacking`이 더 덜 위험하고 실수도 줄어든다.


## 3. Loops 그리고 Iterators

### Item 17: `range` 말고 `enumerate`를 써라

* `enumerate`는 이터레이터를 돌면서 각 아이템의 인덱스를 같이 받는 문법을 간결하게 제공한다.
* `range`로 루프 돌리고 시퀀스를 `index`로 찍는 방식 대신 `enumerate`를 써라.
* `enumerate`는 두 번째 인자로 시작 카운트를 지정할 수 있다(기본은 `0`이다).

### Item 18: 이터레이터를 병렬로 처리하려면 `zip`을 써라

* `zip`은 여러 이터레이터를 병렬로 순회할 때 써라.
* `zip`은 튜플을 만들어내는 lazy generator다. 무한 입력에도 쓸 수 있다.
* 길이가 다른 이터레이터를 넣으면 가장 짧은 쪽에 맞춰서 조용히 잘린다.
* 조용히 잘리는 걸 막고 싶으면 `zip`에 `strict=True`를 넘겨라. 길이가 다르면 `runtime` 에러로 터지게 해라.

### Item 19: `for`/`while` 뒤에 붙는 `else` 블록은 피하라

* 파이썬은 `for`/`while` 바로 뒤에 `else` 블록을 붙일 수 있는 특수 문법이 있다.
* 루프 뒤 `else`는 루프 바디가 `break`를 만나지 않았을 때만 실행된다.
* 루프 뒤 `else`는 직관적이지 않다. 헷갈리니까 쓰지 마라.

### Item 20: 루프가 끝난 뒤 `for` 변수는 절대 쓰지 마라

* `for` 루프 변수는 루프가 끝나도 현재 스코프에서 접근 가능하다. 그거 믿고 쓰지 마라.
* 루프가 한 번도 돌지 않으면 `for` 변수는 스코프에 할당조차 안 될 수 있다.
* `generator expression`이랑 `list comprehension`은 기본적으로 루프 변수를 밖으로 새지 않게 만든다.
* `exception handler`도 예외 인스턴스 변수를 밖으로 새지 않게 한다.

### Item 21: 인자 이터레이션은 방어적으로 해라

* 함수/메서드가 입력 인자를 여러 번 순회하는지 조심해라. 인자가 `iterator`면 값이 사라지거나 이상한 동작이 난다.
* 파이썬 `iterator protocol`이 `iter`/`next`와 `for` 루프 같은 동작을 어떻게 연결하는지 이해해라.
* 너만의 iterable 컨테이너가 필요하면 `__iter__`를 generator로 구현해라.
* 값이 `iterator`인지 확인하려면 `iter(x) is x` 패턴을 써라(같은 객체가 나오면 iterator일 가능성이 크다). 또는 `isinstance(x, collections.abc.Iterator)`로 판별해라.

### Item 22: 순회 중에는 컨테이너를 절대 수정하지 마라. 대신 복사본이나 캐시를 써라

* `list`/`dict`/`set`을 순회하면서 요소를 추가/삭제하지 마라. 예측하기 어려운 `runtime` 에러나 버그가 난다.
* 수정이 필요하면 컨테이너의 복사본을 순회해라.
* 성능 때문에 복사를 피해야 하면, 변경 사항을 두 번째 컨테이너(캐시)에 모아뒀다가 나중에 원본에 합쳐라.

### Item 23: 효율적인 short-circuit 로직엔 `any`와 `all`에 이터레이터를 넘겨라

* `all`은 모든 항목이 truthy면 `True`를 반환한다. falsey를 만나면 즉시 멈추고 `False`를 반환한다.
* `any`는 반대로 동작한다. 모든 항목이 falsey면 `False`고, truthy를 만나면 즉시 멈추고 `True`를 반환한다.
* `any`/`all`은 항상 `True` 또는 `False`만 반환한다. `and`/`or`처럼 “마지막으로 평가된 값”을 반환하지 않는다.
* `any`/`all`에 `list comprehension`을 쓰지 마라. `generator expression`을 써서 short-circuit 이점을 살려라.

### Item 24: 이터레이터/제너레이터 작업엔 `itertools`를 고려해라

* `itertools`는 이터레이터/제너레이터를 다룰 때 쓰는 표준 도구 모음이다.
* 큰 분류는 3가지로 생각해라: 이터레이터 연결, 출력 필터링, 조합 생성.
* 더 고급 함수, 파라미터, 유용한 레시피는 공식 문서를 참고해라.

## 4. Dictionaries

### Item 25: `dict` 삽입 순서(insertion ordering)에 의존할 땐 조심해라

* `Python 3.7`부터 `dict`를 순회하면 키가 처음 추가된 순서대로 나온다고 기대해도 된다.
* 하지만 `dict`처럼 동작하는 “딕셔너리 비슷한 타입”은 `dict` 인스턴스가 아닐 수 있다. 이런 타입은 삽입 순서를 보장한다고 가정하지 마라.
* 딕셔너리 같은 클래스에 안전하게 대응하려면 셋 중 하나로 가라.

  * 삽입 순서에 의존하지 않는 코드로 짜라.
  * 런타임에 `dict` 타입인지 명시적으로 검사해라.
  * 타입 힌트와 `static analysis`로 입력을 `dict`로 강제해라.

### Item 26: 키가 없을 때는 `in`/`KeyError` 말고 `get`을 우선 써라

* 딕셔너리에서 missing key를 처리하는 흔한 방법은 4개다: `in`, `KeyError`, `get`, `setdefault`.
* 기본 타입(카운터 같은 것) 다룰 때는 `get`이 제일 무난하고 깔끔하다.
* 기본값 생성 비용이 크거나 예외가 날 수 있으면 `get`을 쓰고, 필요하면 `assignment expression`으로 중복을 줄여라.
* `setdefault`가 좋아 보이면 `defaultdict`가 더 맞는지 먼저 검토해라.

### Item 27: 내부 상태 관리에는 `setdefault` 말고 `defaultdict`를 써라

* 가능한 키가 계속 늘어날 수 있는 내부 상태용 딕셔너리를 만든다면 `collections.defaultdict`를 우선 고려해라.
* 딕셔너리가 외부에서 들어오는 값이라 네가 생성 과정을 컨트롤 못 하면, 접근은 기본적으로 `get`으로 해라.
* 그래도 코드가 확 짧아지고 기본값 생성 비용이 낮은 일부 상황에서는 `setdefault`를 써도 된다.

### Item 28: 키에 따라 기본값이 달라져야 하면 `__missing__`을 써라

* 기본값 생성 비용이 크거나 예외가 날 수 있으면 `dict.setdefault`는 나쁜 선택이다.
* `defaultdict`의 factory 함수는 인자를 받을 수 없다. 그래서 “키에 의존하는 기본값”을 만들 수 없다.
* 키에 따라 기본값을 만들어야 하면 `dict`를 상속하고 `__missing__`을 정의해서 처리해라.

### Item 29: 딕셔너리/리스트/튜플을 깊게 중첩하지 말고 클래스를 조합해라

* 값이 또 딕셔너리인 딕셔너리, 긴 튜플, 복잡하게 중첩된 built-in 타입 덩어리를 만들지 마라.
* 완전한 클래스가 필요해지기 전까지는 가벼운 데이터 컨테이너로 `dataclasses`를 먼저 써라.
* 내부 상태용 딕셔너리가 복잡해지기 시작하면 bookkeeping 코드를 여러 클래스로 쪼개서 관리해라.

## 5. Functions

### Item 30: 함수 인자는 변형될 수 있다는 걸 알아라

* 파이썬 인자는 reference로 전달된다. 함수/메서드가 인자의 attribute나 내부 값을 바꿀 수 있다.
* 입력을 수정할 거면 이름이랑 문서로 “수정한다”는 걸 분명히 드러내라. 아니면 수정하지 마라.
* 실수로 원본을 건드리기 싫으면, 받은 컬렉션/객체를 복사해서 써라.

### Item 31: 3개 초과 언패킹 강요하지 말고 전용 결과 객체를 반환해라

* 함수는 튜플로 여러 값을 반환하고, 호출자는 `unpacking`으로 받을 수 있다.
* catch-all starred expression으로 여러 반환값을 받을 수도 있다.
* 4개 이상 변수로 언패킹하게 만들지 마라. 실수하기 쉽다. 대신 가벼운 클래스 인스턴스를 반환해라.

### Item 32: `None` 반환하지 말고 예외를 던져라

* 특별한 의미로 `None`을 반환하게 만들지 마라. `None`은 `Boolean`에서 falsey라서 `0`이나 `""` 같은 값이랑 섞이면 버그 난다.
* 특수 상황은 `None`이 아니라 예외로 표현해라. 문서화하고 호출자가 예외를 처리하게 만들어라.
* 타입 힌트로 “이 함수는 어떤 경우에도 `None`을 반환하지 않는다”를 명확히 해라.

### Item 33: 클로저 스코프랑 `nonlocal`을 제대로 알아라

* 클로저 함수는 자신을 감싸는 스코프의 변수를 참조할 수 있다.
* 기본적으로 클로저는 바깥 스코프 변수에 “할당”해서 영향을 주지 못한다.
* 바깥 스코프 변수를 수정해야 하면 `nonlocal`을 써라. 모듈 레벨 이름을 수정해야 하면 `global`을 써라.
* `nonlocal`은 단순한 경우 외에는 남발하지 마라. 복잡해지면 코드가 읽기 어려워진다.

### Item 34: 가변 위치 인자로 visual noise를 줄여라

* 함수가 위치 인자를 여러 개 받게 하려면 정의에서 `*args`를 써라.
* 시퀀스의 아이템을 위치 인자로 풀어서 넘기려면 호출에서 `*`를 써라.
* generator에 `*`를 바로 쓰지 마라. 메모리를 과하게 잡아먹어서 터질 수 있다.
* `*args` 받는 함수에 새 위치 인자를 추가하지 마라. 호출자 쪽 버그가 조용히 생기기 쉽다.

### Item 35: keyword 인자로 옵션 동작을 제공해라

* 함수 인자는 position으로도, keyword로도 넘길 수 있다.
* 위치 인자만 쓰면 의도가 헷갈리는 곳은 keyword로 명확히 해라.
* 기본값 있는 keyword 인자는 기존 호출자를 다 안 고치고도 새 동작을 추가할 수 있게 해준다.
* optional keyword 인자는 항상 keyword로 넘겨라. position으로 넘기지 마라.

### Item 36: 동적인 기본값은 `None`과 docstring으로 지정해라

* 기본 인자 값은 함수 정의 시점(모듈 로드 시점)에 한 번만 평가된다. 동적인 값(함수 호출, 새 객체, 컨테이너)은 이상한 버그를 만든다.
* 동적으로 초기화돼야 하는 기본값은 placeholder로 `None`을 써라. 실제 기본값 의도는 docstring에 적어라. 함수 바디에서 `None` 체크해서 올바른 기본 동작을 트리거해라.
* `None` placeholder 방식은 타입 힌트랑 같이 써도 깔끔하게 맞는다.

### Item 37: keyword-only / positional-only 인자로 호출 의도를 강제해라

* keyword-only 인자는 특정 인자를 반드시 keyword로 넘기게 만들어서 호출 의도를 명확하게 한다. 인자 리스트에서 `*` 뒤에 둬라(`*` 단독이든 `*args`든 상관없다).
* positional-only 인자는 특정 파라미터를 keyword로 못 넘기게 해서 결합도를 줄인다. 인자 리스트에서 `/` 앞에 둬라.
* `/`와 `*` 사이에 있는 파라미터는 기본 규칙대로 position/keyword 둘 다 허용해라.

### Item 38: 데코레이터는 `functools.wraps`로 정의해라

* 데코레이터는 런타임에 한 함수가 다른 함수를 수정하는 문법이다.
* 데코레이터를 잘못 쓰면 디버거 같은 introspection 도구에서 이상하게 보일 수 있다.
* 커스텀 데코레이터를 만들 때는 반드시 `functools.wraps`를 써서 메타데이터가 깨지지 않게 해라.

### Item 39: glue 함수는 `lambda`보다 `functools.partial`을 선호해라

* `lambda`는 인자 재배치나 특정 값 고정으로 인터페이스를 맞출 때 짧게 쓸 수 있다.
* `functools.partial`은 위치/키워드 인자를 고정해서 새 함수를 만드는 범용 도구다.
* 인자 순서를 바꿔야 하는 경우에만 `lambda`를 쓰고, 그 외에는 `partial`로 고정해라.

### Item 40: `map`/`filter` 말고 `comprehension`을 써라

* `list comprehension`은 `map`/`filter`보다 보통 더 명확하다. `lambda`를 강요하지 않기 때문이다.
* `list comprehension`은 `if` 절로 입력 항목을 쉽게 건너뛸 수 있다. `map`은 단독으로 이걸 못 한다.
* `dict`와 `set`도 `comprehension`으로 만들 수 있다.
* `list comprehension`은 평가 시 결과를 전부 메모리에 만든다. 입력이 크면 메모리 많이 먹는다고 생각해라.

### Item 41: `comprehension`에 control subexpression 2개 넘게 넣지 마라

* `comprehension`은 여러 레벨 루프와 여러 조건을 지원한다.
* 제어 subexpression이 2개를 넘어가면 읽기 너무 어렵다. 피하라.

### Item 42: `assignment expression`으로 `comprehension`의 반복을 줄여라

* `assignment expression`은 `comprehension`/`generator expression`에서 한 번 계산한 값을 같은 식 안에서 재사용하게 해준다. 가독성과 성능을 같이 챙겨라.
* 조건 밖에서 `assignment expression`을 억지로 쓰지 마라. 안정적으로 동작하지 않는 경우가 있다.
* `comprehension`에서 `assignment expression`으로 만든 변수는 바깥 스코프로 새어 나간다. 반면 `comprehension`의 루프 변수는 기본적으로 새지 않는다.

### Item 43: 리스트 반환 대신 generator를 고려해라

* 누적 결과 리스트를 만들어 반환하는 것보다 generator가 더 명확한 경우가 많다.
* generator는 함수 바디의 `yield`로 전달한 값들을 순서대로 만들어내는 이터레이터를 반환한다.
* generator는 모든 입력/출력을 메모리에 쌓지 않는다. 그래서 입력이 매우 커도 동작하게 만들어라.

### Item 44: 큰 `list comprehension`이면 `generator expression`을 고려해라

* 입력이 큰데 `list comprehension`을 쓰면 메모리 때문에 문제가 날 수 있다.
* `generator expression`은 출력을 하나씩 만들기 때문에 메모리 문제를 피한다.
* `generator expression`은 다른 `generator expression`의 `for` 입력으로 넘겨서 조합해라.
* 체이닝해도 빠르고 메모리 효율이 좋다.

### Item 45: 여러 generator는 `yield from`으로 합쳐라

* `yield from`은 중첩된 여러 generator를 하나의 generator로 합치는 문법이다.
* `yield from`으로 중첩 generator를 수동으로 돌면서 `yield`하는 boilerplate를 없애라.


### Item 46: generator에 데이터를 넣으려면 `send` 말고 “입력 이터레이터”를 인자로 넘겨라

* `send`는 generator에 값을 주입해서 `yield` 표현식이 값을 받게 만들 수 있다.
* `send`와 `yield from`을 섞지 마라. 예상 못 한 타이밍에 `None`이 튀어나오는 등 놀라운 동작이 생길 수 있다.
* generator들을 조합해야 하면 `send`로 밀어 넣지 말고, 입력 이터레이터를 인자로 전달하는 구조로 짜라. `send`는 피하라.

### Item 47: 상태 전이는 generator `throw` 말고 클래스로 관리해라

* `throw`는 generator 안에서 “마지막 `yield` 지점”에 예외를 다시 던질 수 있다.
* `throw`를 쓰면 예외 raise/catch를 위해 중첩과 boilerplate가 늘어난다. 가독성이 깨진다.
* 반복 처리 + 상태 전이가 필요하면 stateful 클래스를 만들어라. iteration과 state transition을 메서드로 분리해라.

## 7. Classes and Interfaces

### Item 48: 단순 인터페이스면 클래스 말고 함수를 받아라

* 클래스 정의/인스턴스화 대신, 단순한 컴포넌트 인터페이스는 함수로 해결해라.
* 파이썬에서 함수/메서드 레퍼런스는 first-class다. 다른 타입처럼 표현식에 넣어라.
* 클래스 인스턴스를 함수처럼 호출 가능하게 하려면 `__call__`을 구현해라.
* 상태가 필요한 “함수”가 필요하면 stateful closure로 억지로 만들지 마라. `__call__`을 가진 클래스를 고려해라.

### Item 49: `isinstance` 분기 함수 말고 OOP 다형성을 써라

* `isinstance`로 타입을 검사해서 동작을 바꾸는 코드를 남발하지 마라.
* 다형성(polymorphism)은 런타임에 “가장 구체적인 서브클래스 구현”으로 메서드 호출을 디스패치하는 OOP 기법이다.
* 많은 타입 분기가 필요하면 `isinstance` 체인 대신 다형성으로 설계해라. 읽기/유지보수/확장/테스트가 쉬워진다.

### Item 50: OOP 대신 함수형 스타일이 필요하면 `functools.singledispatch`를 고려해라

* OOP는 클래스 중심으로 코드가 흩어지기 쉽다. 큰 프로그램에서 관리가 어려워질 수 있다.
* single dispatch는 메서드 다형성 대신 “함수 기반” 동적 디스패치를 제공해서 관련 기능을 한곳에 모으기 좋다.
* 파이썬은 `functools`에 `singledispatch` 데코레이터를 제공한다. 필요하면 써라.
* 서로 독립적인 시스템들이 같은 데이터 타입을 대상으로 동작한다면, OOP 대신 single dispatch의 함수형 스타일이 더 맞을 수 있다.

### Item 51: 가벼운 클래스 정의엔 `dataclasses`를 선호해라

* `dataclasses`의 `@dataclass`로 boilerplate 없이 가볍고 실용적인 클래스를 정의해라.
* 표준 OOP 문법의 장황함과 실수 포인트를 줄이는 데 `dataclasses`가 도움이 된다.
* `dataclasses`는 `asdict`/`astuple` 같은 변환 헬퍼와 `field` 같은 고급 속성 제어도 제공한다.
* 나중에 커스터마이즈가 더 필요해지면 `dataclasses` 없이도 OOP 패턴을 직접 구현할 수 있어야 한다. 그 길로 옮길 준비를 해라.

### Item 52: `@classmethod` 다형성으로 객체를 범용적으로 생성해라

* 파이썬 클래스는 생성자가 사실상 하나다. `__init__`만 있다고 생각해라.
* 대안 생성자가 필요하면 `@classmethod`로 만들어라.
* `@classmethod` 다형성으로 “여러 구체 하위 클래스”를 공통 방식으로 생성/연결할 수 있게 해라.

### Item 53: 부모 클래스 초기화는 `super`로 해라

* 파이썬의 표준 `MRO`(메서드 해석 순서)가 부모 초기화 순서 문제랑 다이아몬드 상속 문제를 정리해준다.
* 부모 초기화나 부모 메서드 호출은 `super()`(인자 0개)로 해라.

### Item 54: 기능은 `mix-in` 클래스로 조합하는 걸 고려해라

* `mix-in`으로 해결 가능하면, 인스턴스 속성이나 `__init__`까지 얹는 다중 상속은 피 해라.
* 클래스별 커스터마이징이 필요하면, 인스턴스 단위로 끼울 수 있는 “플러그형 동작”도 같이 고려해라.
* `mix-in`은 필요에 따라 인스턴스 메서드도 넣고, 클래스 메서드도 넣어라.
* 작은 동작을 `mix-in`으로 만들고, 여러 개를 조합해서 복잡한 기능을 구성해라.

### Item 55: `private` 속성보다 `public` 속성을 선호해라

* 파이썬에서 `private` 속성은 컴파일러가 엄격하게 강제하지 않는다.
* 처음부터 “서브클래스가 내부 API/속성을 더 활용할 수 있게” 설계해라. 아예 막아버리는 방향으로 가지 마라.
* 접근을 강제로 막으려 하지 말고, `protected` 필드는 문서로 규칙을 잡아서 서브클래스가 따라오게 해라.
* 서브클래스를 네가 통제 못 하는 상황에서 “이름 충돌”을 피해야 할 때만 `private` 속성을 고려해라.
