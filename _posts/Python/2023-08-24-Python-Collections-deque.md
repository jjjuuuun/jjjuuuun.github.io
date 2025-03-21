---
layout: post
title: "[Python] collections - deque"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, collections]
tags: [Python]
comments: true
toc: true
---

Python의 [Standard Library에서 지원하는 collections의 deque](https://docs.python.org/3/library/collections.html#collections.deque)에 대해 알아보려고 합니다.

---

1. deque

   - List에 비해 효율적이고 빠른 자료 저장 방식을 지원하여 속도가 빠릅니다.
   - linked list의 특성을 지원합니다.

2. deque을 활용한 stack 구현

   - stack

     - LIFO(Last In First Out) 구조를 이루는 data type입니다.

       ```python
       from collections import deque

       stack = deque()
       stack.append(1)
       stack.append(2)
       stack.append(3)
       stack.pop()
       stack.pop()
       stack.pop()

           >>  stack = [1] # 삽입 연산 (Push)
           >>  stack = [1, 2] # 삽입 연산 (Push)
           >>  stack = [1, 2, 3] # 삽입 연산 (Push)
           >>  stack = [1, 2] # 삭제 연산 (Pop)
           >>  stack = [1] # 삭제 연산 (Pop)
           >>  stack = [] # 삭제 연산 (Pop)
       ```

3. deque을 활용한 queue 구현

   - Queue

     - FIFO(First In First Out) 구조를 이루는 data type입니다.

       ```python
       from collections import deque

       stack = deque()
       stack.append(1)
       stack.append(2)
       stack.append(3)
       stack.pop(0)
       stack.pop(0)
       stack.pop(0)

           >>  stack = [1] # 삽입 연산 (Enqueue)
           >>  stack = [1, 2] # 삽입 연산 (Enqueue)
           >>  stack = [1, 2, 3] # 삽입 연산 (Enqueue)
           >>  stack = [2, 3] # 삭제 연산 (Dequeue)
           >>  stack = [3] # 삭제 연산 (Dequeue)
           >>  stack = [] # 삭제 연산 (Dequeue)
       ```

4. deque의 linked list 특성

   - rotate

     ```python
     from collections import deque

     deque_ex = deque([1,2,3,4,5,6])
     deque_ex.rotate(2)
     print(deque_ex)
         >>  deque([5, 6, 1, 2, 3, 4])
     deque_ex.rotate(-2)
     print(deque_ex)
         >>  deque([1, 2, 3, 4, 5, 6])
     ```

   - reverse

     ```python
     from collections import deque

     deque_ex = deque([1,2,3,4,5,6])
     print(deque(reversed(deque_ex)))
         >>  deque([6, 5, 4, 3, 2, 1])
     ```
