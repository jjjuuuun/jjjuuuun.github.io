---
layout: post
title: "[Python] Underscore의 사용"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, Underscore]
tags: [Python]
comments: true
toc: true
---

Python에서는 Underscore( `_` )를 다양한 상황에서 사용합니다. 오늘은 Python에서 사용하는 Underscore에 대해서 알아보려고 합니다.

---

## 1. Python에서 언더스코어를 사용하는 경우

- 변수에 값을 할당하지 않고 무시할 때
- 숫자 자릿수를 구분 할 때
- 모듈(.py)내에서만 변수 / 함수를 사용할 때
- 예약어와 같은 이미 지정된 변수 / 함수의 이름과의 충돌을 피하기 위해
- 맨글링(Mangling)
- 매직 메소드(Magic Method)

위의 경우들을 지금부터 하나씩 알아보겠습니다.

## 2. 변수에 값을 할당하지 않고 무시할 때

```python
list_ex = [1, 2, 3, 4, 5]
one, two, _, four, five = list_ex
print(one, two, _, four, five)
    >>  1 2 3 4 5
print(f'{_} == three')
    >>  3 == three
```

위와 같이 리스트를 언패킹 할 때 언더스코어가 사용됩니다. 그러나 언더스코어를 사용했다고 해서 값 자체가 할당이 되지 않는 것은 아닙니다.

언더스코어가 값을 할당 받았기 때문에 변수로 사용이 가능할 것 같은데 왜 언더스코어를 사용해서 무시를 할지 한 번 생각해 봐도 좋을 것 같습니다.

➡️ 저의 생각은 다음과 같습니다. 누군가가 프로그래밍을 할 때는 혼자만 사용하기 위한 코드를 작성하지 않습니다. 해당 프로그램을 공유 할 수도 있고 누군가와 협업을 진행 할 수도 있습니다. 그렇다면 네이밍이라는 것은 굉장히 중요해지며 해당 코드를 이해하는데 불필요한 것들은 최대한 줄이는 것이 굉장히 중요합니다. 그렇기 때문에 이후에 사용하지 않는다는 의미에서 언더스코어를 사용하지 않았을까 생각합니다.😁

## 3. 숫자 자릿수를 구분 할 때

😂 많이 사용은 안하는 것 같습니다.

숫자를 세는 경우 천 단위로 끊어서 보통 콤마(,)를 사용합니다. 그러나 python에서 숫자를 세기 위해 콤마를 찍을 순 없습니다. 이 때 언더스코어를 사용하시면 콤마의 역할을 대체할 수 있습니다.

```python
money = 10000000000
print(money)
    >>  10000000000

money = 10_000_000_000
print(money)
    >>  10000000000
```

## 4. 모듈(.py)내에서만 변수 / 함수를 사용할 때

모듈 Test_module.py를 만들어 보겠습니다.

```python
# Test_module.py
def default_function():
    print("기본적인 함수")


def _underscore_function():
    print("underscore를 앞에 하나 붙인 함수")
```

```python
# Test.py
from Test_module import *

default_function()
    >>  기본적인 함수
_underscore_function()
    >>  NameError: name '_underscore_function' is not defined
```

모듈 Test_module.py를 만들고 Test.py에서 test를 해본 결과 언더스코어를 함수 앞에 하나 붙인 \_underscore_function의 경우 사용할 수 없는 것을 알 수 있습니다.

방금한 test에서는 Test_module로부터 모든 method들을 불러왔습니다. 이번에는 직접 하나씩 불러오도록 해보겠습니다.

```python
from Test_module import default_function, _underscore_function

default_function()
    >>  기본적인 함수
_underscore_function()
    >>  underscore를 앞에 하나 붙인 함수
```

위의 코드에서도 볼 수있듯이 언더스코어를 앞에 붙인 method를 직접 import 시켰더니 사용이 가능한 것을 볼 수 있습니다.

해당 방법은 method의 사용을 조심히 할 필요가 있을 때 사용하면 좋을 것 같습니다.

## 5. 예약어와 같은 이미 지정된 변수 / 함수의 이름과의 충돌을 피하기 위해

👍알아두면 유용해요~~

앞에서도 언급했듯이 프로그래밍을 할 때 네이밍은 무엇보다도 중요할 수 있습니다. 그러나 Python에서는 예약어(if, for)과 같은 이미 python이 설치될 때 정해진 이름들이 있습니다. 또한 프로그래밍을 하다 보면 비슷한 역할아지만 변수명을 다르게 사용해야 할 때 네이밍 하기가 굉장히 힘들 수 있습니다. 이때 이름 뒤에 언더스코어를 하나 붙이면 사용이 가능합니다.

```python
my_name = 'Gil Dong Hong'
print(my_name)
    >>  Gil Dong Hong

my_name_ = '홍길동'
print(my_name_)
    >>  홍길동
```

```python
list_ = [1, 2, 3, 4, 5]
print(list_)
    >>  [1, 2, 3, 4, 5]
```

---

🖥️ 아래 나오는 두 가지 사용법은 특히 github에서 다른 사람의 코드를 볼 때 또는 프로젝트 수준의 큰 코딩을 할 때 굉장히 중요한 것 같습니다.

## 6. 네임 맨글링(Name Mangling)

### 6.1 Private 속성 추가하기

```python
class Resume():
    def __init__(self):
        self.name = "홍길동"
        self.age = "20"
        self.hobby = "축구"

person = Resume()
print(person.name)
    >>  홍길동
print(person.age)
    >>  20
print(person.hobby)
    >>  축구
```

Private 속성을 추가하지 않았기 때문에 class 밖에서도 접근이 가능합니다. 그러나 이런 경우 class의 변수 값이 본인의 의도와 상관없이 바뀔 수 있으므로 이를 방지하기 위해서는 private 속성을 추가해야 합니다. Python에서는 언더스코어를 변수명 앞에 두개 붙여줍니다.

```python
class Resume():
    def __init__(self):
        self.name = "홍길동"
        self.age = "20"
        self.__hobby = "축구"

person = Resume()
print(person.name)
    >>  홍길동
print(person.age)
    >>  20
print(person.hobby)
    >>  AttributeError: 'Resume' object has no attribute '__hobby'
```

그러면 AttributeError가 발생하면서 접근이 불가능하게 됩니다.

그런데 접근을 해야 할 경우가 있을 수 있습니다. 이럴땐 `property decorator`를 사용해 private 속성을 가진 변수도 접근이 가능하게 할 수 있습니다.

```python
class Resume():
    def __init__(self):
        self.name = "홍길동"
        self.age = "20"
        self.__hobby = "축구"

    @property
    def hobby(self):
        return self.__hobby

person = Resume()
print(person.name)
    >>  홍길동
print(person.age)
    >>  20
print(person.hobby)
    >>  축구
```

위의 방법 말고도 아래와 같이 클래스 이름을 통해 접근하는 방법도 있습니다.

> `ObjectName._ClassName__PrivateVariableName`

```python
class Resume():
    def __init__(self):
        self.name = "홍길동"
        self.age = "20"
        self.__hobby = "축구"

    @property
    def hobby(self):
        return self.__hobby

person = Resume()
print(person.__dict__)
    >>  {'name': '홍길동', 'age': '20', '_Resume__hobby': '축구'}
print(person._Resume__hobby)
    >>  축구
```

### 6.2 Overriding

> Overriding : 상위 클래스의 method와 이름이 같은 함수를 하위 클래스에 재정의 하는 것을 말합니다.

```python
class Resume():
    def __init__(self):
        self.name = "홍길동"
        self.age = "20"
        self.hobby = "축구"

class English_resume(Resume):
    def __init__(self):
        super().__init__()
        self.name = "Gil-Dong Hong"
        self.age = "20"
        self.hobby = "Soccer"

english_person = English_resume()
print(english_person.name)
    >>  Gil-Dong Hong
print(english_person.age)
    >>  20
print(english_person.hobby)
    >>  Soccer
```

위 코드는 정상적인 오버라이딩이 수행되는 코드이다. 아래 코드에서 언더스코어를 사용하면 어떻게 될지 한 번 살펴보겠습니다.

```python
class Resume():
    def __init__(self):
        self.name = "홍길동"
        self.age = "20"
        self.__hobby = "축구"

class English_resume(Resume):
    def __init__(self):
        super().__init__()
        self.name = "Gil-Dong Hong"
        self.age = "20"
        self.__hobby = "Soccer"

english_person = English_resume()
print(english_person.name)
    >>  Gil-Dong Hong
print(english_person.age)
    >>  20
print(english_person.__hobby)
    >>  AttributeError: 'English_resume' object has no attribute '__hobby'
```

## 7. 매직 메소드(Magic Method)

> 매직 메소드 : 메소드 앞 뒤에 언더스코어를 두 개씩 붙인것을 매직 메소드라고 하며 여러가지 Built-in 함수들이 처리할 연산을 정의한 것을 말합니다.

대표적인 매직 메소드는 `__init__, __add__, ___str__`등이 있습니다. 그 중에서 `__add__`를 사용하는 방법은 다음과 같습니다.

```python
class Resume():
    def __init__(self):
        self.name = "홍길동"
        self.age = "20"
        self.hobby = "축구"

    def __add__(self, other):
        return int(self.age) + int(other.age)

person1 = Resume()
person2 = Resume()

print(person1 + person2)
    >>  40
```

Python에서 정의할 수 있는 매직 메소드는 필요에 따라 공식문서나 여러 자료들을 찾아가며 프로그래밍 하실 수 있을 것 같습니다.

---

※ Reference

- [https://losskatsu.github.io/programming/py-underscore/#8-%EC%95%9E%EB%92%A4-%EC%96%B8%EB%8D%94%EB%B0%94-2%EA%B0%9C%EC%94%A9-%EB%A7%A4%EC%A7%81-%EB%A9%94%EC%86%8C%EB%93%9C](https://losskatsu.github.io/programming/py-underscore/#8-%EC%95%9E%EB%92%A4-%EC%96%B8%EB%8D%94%EB%B0%94-2%EA%B0%9C%EC%94%A9-%EB%A7%A4%EC%A7%81-%EB%A9%94%EC%86%8C%EB%93%9C)
- [https://corikachu.github.io/articles/python/python-magic-method](https://corikachu.github.io/articles/python/python-magic-method)
