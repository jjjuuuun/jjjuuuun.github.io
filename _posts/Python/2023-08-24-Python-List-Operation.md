---
layout: post
title: "[Python] List"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, List]
tags: [Python]
comments: true
toc: true
---

Python의 [Standard Library에서 지원하는 List(Mutable Sequence Type)](https://docs.python.org/3.8/library/stdtypes.html#sequence-types-list-tuple-range)에 대해 알아보려고 합니다.

---

1. List

   - Sequence Data type이면서 다양한 data type을 포함할 수 있는 여러 데이터들의 집합을 의미하며 list 자료형은 수정이 가능한 mutable type입니다.

     ```python
     list_ex = [False, 1, '2', [3, 4], (5, 6), {7:8, 9:10}]
     print(list_ex)
         >> [False, 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}]
     ```

2. Indexing과 Slicing

   - **`list_ex[i] = x`**
     - list_ex의 i번째 원소를 x로 대체합니다.
       ```python
       list_ex[0] = 0
       print(list_ex)
           >> [0, 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}]
       ```
   - **`list_ex[i:j] = t`**
     - list_ex의 i번째 원소부터 j번째 원소를 iterable t의 원소들로 대체 합니다.
     - 실제 계산될 때는 i번 부터 j-1까지 변경됩니다.(index 시작은 0이라는 점을 고려해야 합니다.)
       ```python
       list_ex[1:3] = [True, [2, 3]]
       print(list_ex)
           >>  [0, True, [2, 3], [3, 4], (5, 6), {7: 8, 9: 10}]
       ```
   - **`list_ex[i:j:k] = t`**
     - list_ex의 i번째 원소부터 j번째 원소 중 k번째마다 iterable t의 원소들로 대체 합니다.
     - 위의 slicing에서 step이 추가 된 것을 제외하고는 같습니다.
       ```python
       list_ex[0:6:2] = ['1', '2', '3']
       print(list_ex)
           >>  ['1', True, '2', [3, 4], '3', {7: 8, 9: 10}]
       ```

3. 추가 연산

   - **`list_ex.append(x)`**

     - `list_ex.append(x)` **==** `list_ex[len(list_ex):len(list_ex)] = [x]`

       ```python
       list_ex[len(list_ex):len(list_ex)] = [11]
       print(list_ex)
           >>  [False, 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}, 11]
       ```

       ```python
       list_ex.append(12)
       print(list_ex)
           >>  [False, 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}, 11, 12]
       ```

       ```python
       list_ex[len(list_ex):len(list_ex)] = [11, 12, 13]
       print(list_ex)
           >>  [False, 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}, 11, 12, 13]
       ```

   - **`list_ex.extend(t)`**

     - `list_ex.extend(t)` **==** `list_ex += t</U> == <U>list_ex[len(list_ex):len(list_ex)] = t`

       ```python
       list_ex += [14, 15, 16]
       print(list_ex)
           >>  [False, 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}, 11, 12, 13, 14, 15, 16]
       ```

       ```python
       list_ex.extend([17, 18, 19])
       print(list_ex)
           >>  [False, 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}, 11, 12, 13, 14, 15, 16, 17, 18, 19]
       ```

4. 삽입 연산

   - **`list_ex.insert(i, x)`**

     - `list_ex.insert(i, x)` **==** `s[i:i] = [x]`
     - i번째부터 len(list_ex)번째의 원소들을 한 칸씩 뒤로 미루고 index i번째 자리에 x를 집어넣는 연산이 수행됩니다.

       ```python
       list_ex.insert(0, False)
       print(list_ex)
           >>  [False, False, 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}]
       ```

       ```python
       list_ex[2:2] = ['1']
       print(list_ex)
           >>  [False, False, '1', 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}]
       ```

5. 삭제 연산

   - **`del list_ex[i:j]`**
     - `del list_ex[i:j]` **==** `list_ex[i:j] = []`
       ```python
       del list_ex[0:1]
       print(list_ex)
           >>  [1, '2', [3, 4], (5, 6), {7: 8, 9: 10}]
       ```
       ```python
       list_ex[0:1] = []
       print(list_ex)
           >>  ['2', [3, 4], (5, 6), {7: 8, 9: 10}]
       ```
   - **`del list_ex[i:j:k]`**
     ```python
     del list_ex[0:4:2]
     print(list_ex)
         >>  [[3, 4], {7: 8, 9: 10}]
     ```
   - **`list_ex.clear()`**

     - `list_ex.clear()` **==** `del list_ex[:]`

       ```python
       list_ex = [False, 1, '2', [3, 4], (5, 6), {7:8, 9:10}]

       list_ex.clear()
       print(list_ex)
           >>  []
       ```

       ```python
       list_ex = [False, 1, '2', [3, 4], (5, 6), {7:8, 9:10}]

       del list_ex[:]
       print(list_ex)
           >>  []
       ```

   - **`list_ex.pop()`** or **`list_ex.pop(i)`**

     - i가 있으면 i번째 원소를 반환하고 list_ex에서 제거해줍니다.
     - i가 없으면 default 값은 맨 마지막 원소입니다.

       ```python
       list_ex = [1, 2, 2, 3, 4, 2, 5, 6, 7]

       list_ex.pop()
       print(list_ex)
           >>  [1, 2, 2, 3, 4, 2, 5, 6]
       ```

       ```python
       list_ex.pop(3)
       print(list_ex)
           >>  [1, 2, 2, 4, 2, 5, 6]
       ```

   - **`list_ex.remove(x)`**

     - list_ex에 여러 개의 x가 있는 경우 맨 처음 나오는 x를 제거해줍니다.

       ```python
       list_ex.remove(2)
       print(list_ex)
           >>  [1, 2, 4, 2, 5, 6]
       ```

6. 기타

   - **`list_ex.reverse()`**

     - `list_ex.reverse()` **==** `list_ex[::-1]`
     - 그러나 list_ex[::-1]과 완전히 같지는 않습니다. reverse()연산의 경우 list_ex 자체를 변경시키는 반면에 list_ex[::-1] 연산의 경우 list_ex 자체를 변경시키지는 않습니다.

       ```python
       list_ex.reverse()
       print(list_ex)
           >>  [{7: 8, 9: 10}, (5, 6), [3, 4], '2', 1, False]
       ```

       ```python
       print(list_ex[::-1])
           >>  [False, 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}]
       ```

   - **`list_ex.copy()`**

     - `list_ex.copy()` **==** `list_ex[:]`
     - copy() 연산의 경우 shallow copy를 지원합니다. shallow copy의 반댓말로는 deep copy가 있는데 해당 설명은 다음 포스트로 설명을 미루고자 합니다. >> [Shallow Copy VS Deep Copy](https://jjjuuuun.github.io/2023-03-07-Python-List-Copy/)

       ```python
       list_ex_copy = list_ex.copy()
       print(list_ex_copy)
           >>  [False, 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}]
       ```

       ```python
       list_ex_copy = list_ex[:]
       print(list_ex_copy)
           >>  [False, 1, '2', [3, 4], (5, 6), {7: 8, 9: 10}]
       ```
