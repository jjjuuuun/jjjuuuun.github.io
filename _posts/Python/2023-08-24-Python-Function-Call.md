---
layout: post
title: "[Python] Function - 함수에 parameter를 전달하는 방식"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, Function]
tags: [Python]
comments: true
toc: true
---

C언어나 C++언어를 배우신분들에게는 call by value와 call by reference는 굉장히 익숙한 용어가 아닐까 싶습니다. 이번 포스트에서는 call by value와 call by reference를 살펴봄과 동시에 파이썬에서는 어떤 방식을 채택해 사용하고 있는지 알아보겠습니다.

---

## 1. 값에 의한 호출(Call by Value)

Call by value는 말 그대로 값만을 부른다는 의미입니다. 쉽게 파이썬 코드와 함께 알아보겠습니다.

⚠️ 파이썬은 해당 방식을 사용하지 않으므로 Call by value가 무엇인지에 대해서 이해하기위한 용도로만 봐주셔야 합니다.

```python
def naming(person_name):
    print(person_name)
        >>  홍길동
    person_name = "길동홍"
    print(person_name)
        >>  길동홍

name = "홍길동"

print(name)
    >>  홍길동

naming(name)

print(name)
    >>  홍길동
```

위의 코드에서 `naming`이라는 함수를 호출(call)하기 전에는 `name`이라는 변수는 <U>"홍길동"</U>으로 출력이 됩니다. 그리고나서 `naming`을 호출해주면 `name`이라는 변수에 있던 <U>"홍길동"</U>을 `person_name`으로 넘겨주면 `person_name`은 <U>"홍길동"</U>를 받습니다. 이때 call by value는 값을 copy하여 받으므로 `person_name`을 코드에서와 같이 변경하더라도 `name`의 값은 변경되지 않습니다.

## 2. 참조에 의한 호출(Call by Reference)

그렇다면 call by reference는 어떻게 실행되는지 감이 오시나요?? 결과부터 말씀드리면 name의 값도 변경이 됩니다.

⚠️ 이 또한 call by reference가 무엇인지 이해하기 위한 용도로만 봐주셔야 합니다.

```python
def naming(person_name):
    print(person_name)
        >>  홍길동
    person_name = "길동홍"
    print(person_name)
        >>  길동홍

name = "홍길동"

print(name)
    >>  홍길동

naming(name)

print(name)
    >>  길동홍
```

Call by reference는 값을 copy하여 넘겨주는 것이 아니라 주소값 자체를 넘겨주기 때문에 값이 변경되는 것입니다. 자세히 말하자면 참조하고 있는 주소의 값을 변경시키기 때문에 해당 주소를 참조하고 있는 변수들의 값들이 모두 바뀌게 되는 것입니다.

## 3. 객체 참조에 의한 호출(Call by Object Reference 또는 Call by assignment)

Call by Object Reference는 파이썬에서 사용하고 있는 방법입니다. 해당 방법은 call by value와 call by reference의 특징들을 섞어 놓은 방법입니다.

함수가 인자로 전달받은 것을 그대로 사용하는 경우에는 call by reference 처럼 원래의 값도 변경이 됩니다. 하지만 해당 변수명으로 새로운 객체를 만들어버리면 그 때부터는 call by value와 같이 원래의 값이 변경되지 않습니다.

이는 파이썬에서는 모든 것을 객체로 보기 때문이라고 할 수 있습니다. 모든 것을 객체로 보기 때문에 객체의 주소가 함수로 넘어가 해당 객체의 값을 변경 했을 경우에는 기존의 변수도 같은 주소를 가지고 있는 객체이기 때문에 변경이 되지만 같은 이름의 새로운 객체를 만들게 되면 더 이상 기존의 객체의 주소와 같지 않기 때문에 기존의 변수는 변경되지 않습니다.

```python
def call_by_object_reference(object_ex_reference):
    object_ex_reference.append(2)
    print(f'reference : {object_ex_reference}')
        >>  reference : [0, 1, 2]
    print(f'object_ex : {object_ex}')
        >>  object_ex : [0, 1, 2]
    object_ex_reference = [10, 9, 8]
    print(f'new_obejct : {object_ex_reference}')
        >>  new_obejct : [10, 9, 8]
    print(f'object_ex : {object_ex}')
        >>  object_ex : [0, 1, 2]


object_ex = [0,1]
call_by_object_reference(object_ex)
```
