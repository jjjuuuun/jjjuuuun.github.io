---
layout: post
title: "[Python] List - Comprehension"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, List]
tags: [Python]
comments: true
toc: true
---

List를 선언할 때 빠르고 파이썬 답게 사용하기 위한 List Comprehension에 대해 알아보려고 합니다.

---

## 1. List Comprehension

일반적으로 List Comprehension을 사용하면 for문과 append를 사용해 list를 선언하는 것보다 속도면에서 조금이나마 빠르다는 장점이 있습니다. 그렇다면 list comprehension을 어떻게 구현하는 것인지 알아보겠습니다.

우선, list를 for문과 append로 구현해보겠습니다.

```python
list_for = []
for i in range(10):
    list_for.append(i)
print(list_for)
    >>  [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

위의 for문과 append로 작성한 코드를 list comprehension으로 한 번 작성해보겠습니다.

```python
list_comprehension = [i for i in range(10)]
print(list_comprehension)
    >>  [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

List comprehension을 사용할 때는 빈 list를 선언함과 동시에 원소가 할당이 되는 것을 알 수 있습니다.

처음 list comprehension을 알게 되면 list 안에 for 구문을 어떻게 사용해야 하는지 헷갈릴 수 있습니다. 아래의 설명을 통해 for 구문을 어떤 순서로 작성해야 할지 알아보겠습니다. 먼저 이중 for문을 어떻게 list comprehension으로 작성하면 좋을 지 알아보겠습니다.

## 2. 이중 for문을 list comprehension으로 표현하는 방법

우선, list를 이중 for문과 append로 구현해보겠습니다.

```python
list_for = []
for i in range(2):
    for j in range(3):
        list_for.append((i,j))
print(list_for)
    >>  [(0, 0), (0, 1), (0, 2), (1, 0), (1, 1), (1, 2)]
```

다음은 list comprehension을 통해 list를 구현해 보겠습니다.

```python
list_comprehension = [(i, j) for i in range(2) for j in range(3)]
print(list_comprehension)
    >>  [(0, 0), (0, 1), (0, 2), (1, 0), (1, 1), (1, 2)]
```

For문과 append로 구현한 list에서 for문의 순서가 ⬇️으로 진행이 됬다면 list comprehension에서 for문의 순서는 ➡️으로 진행이 됩니다.

여기서 만약 if문이 추가 된다면 어떻게 작성해야 할지 알아보겠습니다.

## 3. List Comprehension과 if문 같이 사용하는 방법

If문이 포함된 list comprehension을 구현할 때는 else문이 있느냐 없느냐에 따라 작성 방법이 바뀝니다. 먼저 if문만 있는 경우에 대해 알아보겠습니다.

우선, list를 이중 for문, append, if문으로 구현해보겠습니다.

```python
list_for = []
for i in range(2):
    for j in range(3):
        if i != j:
            list_for.append((i,j))
print(list_for)
    >>  [(0, 1), (0, 2), (1, 0), (1, 2)]
```

다음은 list comprehension을 통해 list를 구현해 보겠습니다.

```python
list_comprehension = [(i, j) for i in range(2) for j in range(3) if i != j]
print(list_comprehension)
    >>  [(0, 1), (0, 2), (1, 0), (1, 2)]
```

기본적인 이중 for문을 list comprehension으로 구현할 때와 같습니다. for문, append, if문으로 구현한 list의 for문과 if문의 순서가 ⬇️으로 진행이 됬다면 list comprehension으로 구현된 list의 for문과 if문의 순서는 ➡️입니다.

그러나 else문이 포함되면 if-else문의 순서가 달라집니다. 어떻게 달라지는지 살펴보겠습니다.

우선, list를 이중 for문, append, if-else문으로 구현해보겠습니다.

```python
list_comprehension = []
for i in range(2):
    for j in range(3):
        if i != j:
            list_comprehension.append((i,j))
        else:
            list_comprehension.append(('same', 'same'))
print(list_comprehension)
    >> [('same', 'same'), (0, 1), (0, 2), (1, 0), ('same', 'same'), (1, 2)]
```

다음은 list comprehension을 통해 list를 구현해 보겠습니다.

```python
list_comprehension = [(i, j) if i != j else ('same', 'same')for i in range(2) for j in range(3)]
print(list_comprehension)
    >>  [('same', 'same'), (0, 1), (0, 2), (1, 0), ('same', 'same'), (1, 2)]
```

If-else문이 for문 앞으로 튀어 나온 것을 알 수 있습니다.

🔎 삼중 for문도 list comprehension으로 구현할 수는 있지만 코드의 가독성과 삼중 for문을 굳이 list comprehension으로 작성해야 할 이유가 없는 한 기본적으로는 이중 for문까지만 list comprehension으로 작성하는 것 같습니다.

---
