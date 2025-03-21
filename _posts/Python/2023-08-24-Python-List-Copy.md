---
layout: post
title: "[Python] List - Copy"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, List]
tags: [Python]
comments: true
toc: true
---

Python의 List자료형에 대한 얇은 복사와 깊은 복사에 대해 알아보려고 합니다. 해당 자료는 **<U>Do it! 자료구조와 함께 배우는 알고리즘 입문 파이썬편</U>** 을 참고하였습니다.

---

## 1. Python의 변수

Python에서 데이터, 함수, 클래스, 모듈, 패키지 등을 모두 객체(Object)로 취급합니다. 따라서 객체는 data type을 가지며 메모리를 차지합니다. 이러한 특징 때문에 Python의 변수는 값을 갖지 않는다는 특징이 있습니다.

정리하자면, 변수는 객체를 참조하는 객체에 연결된 이름에 불과하며 모든 객체는 메모리를 차지하고, 자료형뿐만 아니라 식별 번호(identity)를 가집니다.

![](https://ifh.cc/g/s6Na2o.png){: class="align-center"}

## 2. Shallow Copy

Shallow Copy라 불리는 이유에 대해서 알아보겠습니다.

Shallow Copy는 원소 수준에서만 복사가 일어납니다. 즉, `list_ex_copy`는 `list_ex[0]`, `list_ex[1]`, `list_ex[2]`를 복사하고 `list_ex_copy[i]`는 `list_ex[i][j]`를 참조합니다. 따라서 `list_ex_copy[i][j]`를 변경하면 `list_ex[i][j]` 또한 변경되지만 `list_ex_copy[i]`를 변경한다고 해서 `list_ex[i]`가 변경되지는 않습니다. 이점을 유의 해야 합니다.

![](https://ifh.cc/g/xwmxwK.png){: class="align-center"}

```python
list_ex = [[1,2,3], [4,5,6], [7,8,9]]

list_ex_copy = list_ex.copy()
list_ex_copy[0][1] = 0
list_ex_copy[1] = [10,11,12]

print(list_ex_copy)
    >>  [[1, 0, 3], [10, 11, 12], [7, 8, 9]]

print(list_ex)
    >>  [[1, 0, 3], [4, 5, 6], [7, 8, 9]]
```

## 3. Deep Copy

Deep Copy는 Shallow Copy와 다르게 구성 원소 수준으로 복사를 진행하는 것을 말합니다. 구성 원소 수준이란 2차원 리스트에서 `list_ex[0][1]`과 같이 원소의 원소를 의미합니다. 따라서 `list_ex_copy[i][j]`를 변경한다고 해서 `list_ex[i][j]`의 값이 변경 되지 않습니다.

![](https://ifh.cc/g/NroYXj.png){: class="align-center"}

```python
from copy import deepcopy

list_ex = [[1,2,3], [4,5,6], [7,8,9]]
list_ex_copy = deepcopy(list_ex)
list_ex_copy[0][1] = 0
list_ex_copy[1] = [10,11,12]

print(list_ex_copy)
    >>  [[1, 0, 3], [10, 11, 12], [7, 8, 9]]

print(list_ex)
    >>  [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

## 4. 정리 💭

1차원 리스트를 복사하는 경우에는 Shallow Copy나 Deep Copy와 상관없이 copy본이 변경되어도 원본이 변경되지 않지만 2차원 이상의 리스트를 복사하는 경우에는 주의 할 필요가 있습니다. Shallow Copy는 간접 참조, Deep Copy는 직접 참조라고 할 수도 았습니다.
