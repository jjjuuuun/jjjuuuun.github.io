---
layout: post
title: "[Python] Function - Python 함수의 특징"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, Function]
tags: [Python]
comments: true
toc: true
---

Python의 함수가 가지고 있는 특징에 대해 알아보고자 합니다.

---

## 1. 일급 함수

Python에서 함수는 변수나 데이터 구조에 할당이 가능한 객체입니다. 따라서 parameter로 전달이 가능하며 return 값으로도 사용이 가능한 특징이 있습니다.

사실 이러한 특징을 모르고 계셨다고 하더라도 우리는 이미 변수로서 사용한 경험이 있습니다.

아래 코드와 같이 map함수를 사용할 때 `int`를 변수로서 사용하거나 sys.stdin.readline을 변수로 받아 사용한 경우 모두 일급 함수 특징에 해당 됩니다.

```python
list_ = ['1', '2', '3', '4', '5']
print(list(map(int, list_)))
    >>  [1, 2, 3, 4, 5]
```

```python
import sys

input = sys.stdin.readline
list_ = input()
```

## 2. Inner Function

Inner Function이란 함수 내에 또 다른 함수가 정의 되어 있는 것을 말합니다.

```python
def outer_function(msg):
    def inner_function():
        print(msg)
    inner_function()

outer_function("Inner Function 입니다.")
    >>  Inner Function 입니다.
```

Inner Function을 언급할 때 closures를 같이 언급합니다. closures란 inner function을 return 값으로 반환하는 것을 말합니다.

```python
def outer_function(msg):
    def inner_function():
        print(msg)
    return inner_function()

outer_function("Inner Function 입니다.")
    >>  Inner Function 입니다.
```

## 3. Decorator Function

Decorator는 복잡한 클로저 함수를 간단하게 만들때 사용합니다. Decorator를 사용한 코드가 어떻게 돌아가는지 하나씩 살펴보겠습니다.

```python
def make_star(func):
    print(f'04번 make_star.func는 무엇인가 >> {func}')
    def inner(*args, **kwargs):
        print('06번')
        func(*args, **kwargs)
        print('10번')
    print(f'05번 make_stac.inner는 무엇인가 >> {inner}')
    return inner

def make_percent(func):
    print(f'02번 make_percent.func는 무엇인가 >> {func}')
    def inner(*args, **kwargs):
        print('07번')
        func(*args, **kwargs)
        print('09번')
    print(f'03번 make_percent.inner는 무엇인가 >> {inner}')
    return inner

print('01번')
@make_star
@make_percent
def printer(msg):
    print(f'08번 {msg}')

printer("Decorator Function")
    >>  01번
    >>  02번 make_percent.func는 무엇인가 >> <function printer at 0x000002336EDA50D0>
    >>  03번 make_percent.inner는 무엇인가 >> <function make_percent.<locals>.inner at 0x000002336EDA5160>
    >>  04번 make_star.func는 무엇인가 >> <function make_percent.<locals>.inner at 0x000002336EDA5160>
    >>  05번 make_stac.inner는 무엇인가 >> <function make_star.<locals>.inner at 0x000002336EDA51F0>
    >>  06번
    >>  07번
    >>  08번 Decorator Function
    >>  09번
    >>  10번
```
