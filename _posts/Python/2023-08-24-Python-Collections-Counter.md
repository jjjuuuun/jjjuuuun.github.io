---
layout: post
title: "[Python] collections - Counter"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, collections]
tags: [Python]
comments: true
toc: true
math: true
---

Python의 [Standard Library에서 지원하는 collections의 Counter](https://docs.python.org/3.8/library/collections.html#collections.Counter)에 대해 알아볼 것이다.

---

1. Counter

   - 개수를 세어 내림차순으로 Counter 객체를 반환해준다.

   ```python
   A = 'france faa'
   B = 'french fee'

   A_Counter = Counter(A)
   B_Counter = Counter(B)

   print(A_Counter)
       >>  Counter({'a': 3, 'f': 2, 'r': 1, 'n': 1, 'c': 1, 'e': 1, ' ': 1})
   print(B_Counter)
       >>  Counter({'e': 3, 'f': 2, 'r': 1, 'n': 1, 'c': 1, 'h': 1, ' ': 1})
   ```

2. Counter.most_common(num)

   - 가장 많은 원소를 가지고 있는 상위 요소 num개를 반환한다.
   - 인자를 주지 않으면 전체를 출력한다.

   ```python
   print(A_Counter.most_common(1))
       >>  [('a', 3)]
   print(B_Counter.most_common(2))
       >>  [('e', 3), ('f', 2)]
   print(A_Counter.most_common())
       >>  [('a', 3), ('f', 2), ('r', 1), ('n', 1), ('c', 1), ('e', 1), (' ', 1)]
   print(B_Counter.most_common())
       >>  [('e', 3), ('f', 2), ('r', 1), ('n', 1), ('c', 1), ('h', 1), (' ', 1)]
   ```

3. +, - 연산

   - $+$ 연산
     - 두 Counter 객체를 그대로 더한 것을 의미한다.
   - $-$ 연산
     - 두 Counter 객체를 그대로 뺀 것을 의미한다.
     - A_Counter에서 B_Counter를 뺄 때 A_Counter에 존재하는 값에 대해서만 연산을 수행한다.

   ```python
   print(A_Counter + B_Counter)
       >>  Counter({'f': 4, 'e': 4, 'a': 3, 'r': 2, 'n': 2, 'c': 2, ' ': 2, 'h': 1})
   print(A_Counter - B_Counter)
       >>  Counter({'a': 3})
   print(B_Counter - A_Counter)
       >>  Counter({'e': 2, 'h': 1})
   ```

4. Counter.subtract(Counter)

   - $-$연산과 다르게 존재하지 않아도 연산이 가능하다
   - 따라서 마이너스 값이 존재할 수 있다.
   - 주의 : 객체 정보가 직접 바뀐다.

   ```python
   # A_Counter의 정보가 바뀜
   A_Counter.subtract(B_Counter)
   print(A_Counter)
       >>  Counter({'a': 3, 'f': 0, 'r': 0, 'n': 0, 'c': 0, ' ': 0, 'h': -1, 'e': -2})

   # B_Counter의 정보가 바뀜
   B_Counter.subtract(A_Counter)
   print(B_Counter)
       >>  Counter({'e': 2, 'h': 1, 'f': 0, 'r': 0, 'n': 0, 'c': 0, ' ': 0, 'a': -3})
   ```

5. 합집합과 교집합

   - Counter 객체는 합집합과 교집합을 지원한다.

   ```python
   print(A_Counter | B_Counter)
       >>  Counter({'a': 3, 'e': 3, 'f': 2, 'r': 1, 'n': 1, 'c': 1, ' ': 1, 'h': 1})
   print(A_Counter & B_Counter)
       >>  Counter({'f': 2, 'r': 1, 'n': 1, 'c': 1, 'e': 1, ' ': 1})
   ```

6. Counter.elements()

   - 원소를 개수만큼 iterable 객체로 반환해준다.

   ```python
   print(list(A_Counter.elements()))
       >>  ['f', 'f', 'r', 'a', 'a', 'a', 'n', 'c', 'e', ' ']
   print(list(B_Counter.elements()))
       >>  ['f', 'f', 'r', 'e', 'e', 'e', 'n', 'c', 'h', ' ']
   ```
