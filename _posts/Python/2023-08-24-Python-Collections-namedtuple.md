---
layout: post
title: "[Python] collections - namedtuple"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, collections]
tags: [Python]
comments: true
toc: true
---

Python의 [Standard Library에서 지원하는 collections의 namedtuple](https://docs.python.org/3/library/collections.html#collections.namedtuple)에 대해 알아보려고 합니다.

---

1. namedtuple

   - Tuple 형태로 data 구조체를 저장하는 방법입니다.
   - C 언어에서의 structure와 비슷합니다.
   - `namedtuple(typename, field_names, *, rename=False, defaults=None, module=None)`

2. namedtuple 선언

   - rename

     - field_names가 중복된 경우 자동으로 rename해주는 옵션입니다.
     - True로 설정하지 않았는데 filed_names가 중복된 경우 에러가 발생합니다.

       ```python
       from collections import namedtuple

       Point = namedtuple('Point', ['x', 'y'])
       Point = namedtuple('Point', 'x y')
       Point = namedtuple('Point', 'x, y')
       Point = namedtuple('Point', 'x y x z', rename=True)
       ```

3. namedtuple 기본적인 사용

   ```python
   from collections import namedtuple

   Point = namedtuple('Point', ['x', 'y'])
   p = Point(11, y=22)
   print(p)
       >>  11
   print(p[0])
       >>  22
   print(p.y)
       >>  11

   a, b = p
   print(a)
       >>  11
   ```

4. namedtuple method

   - \_make()

     - 새로운 객체를 생성합니다.

       ```python
       from collections import namedtuple

       Point = namedtuple('Point', ['x', 'y'])
       ppoint = [11, 22]
       p = Point._make(ppoint)
       print(p)
           >>  Point(x=11, y=22)
       ```

   - \_asdict()

     - dict type으로 반환해줍니다.

       ```python
       from collections import namedtuple

       Point = namedtuple('Point', ['x', 'y'])
       p = Point(x=11, y=22)
       p_dict = p._asdict()
       print(p_dict)
           >>  {'x': 11, 'y': 22}
       ```

   - \_replace()

     - 값을 변경시키고 새로운 namedtuple을 반환해줍니다.

       ```python
       from collections import namedtuple

       Point = namedtuple('Point', ['x', 'y'])
       p = Point(x=11, y=22)
       new_p = p._replace(x=33)
       print(new_p)
           >>  Point(x=33, y=22)
       ```

   - \_fields()

     - field names를 tuple로 반환해줍니다.

       ```python
       from collections import namedtuple

       Point = namedtuple('Point', ['x', 'y'])
       p = Point(x=11, y=22)
       print(p._fields)
           >>  ('x', 'y')
       ```
