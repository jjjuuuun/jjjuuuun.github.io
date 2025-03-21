---
layout: post
title: "[Python] collections - defaultdict"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, collections]
tags: [Python]
comments: true
toc: true
---

Python의 [Standard Library에서 지원하는 collections의 defaultdict](https://docs.python.org/3/library/collections.html#collections.defaultdict)에 대해 알아보려고 합니다.

---

1. defaultdict

   - Dict type의 값에 기본 값을 지정, 신규 값 생성 시 사용하는 방법입니다.

2. Python의 built-in dict class

   - Key 값이 없는 경우 KeyError를 보여줍니다.

     ```python
     dict_ex = {}
     print(dict_ex['e'])
         >>  KeyError: 'e'
     ```

3. defaultdict를 사용

   - 기본 값으로 `int`를 지정해 줌으로써 0으로 기본 값을 지정해주는 것을 알 수 있습니다.

     ```python
     from collections import defaultdict

     dict_ex = defaultdict(int)
     print(dict_ex['e'])
         >> 0
     ```

4. defaultdict의 여러가지 경우

   - 필요에 따라 기본 값을 설정해 줄 수 있습니다.

     ```python
     from collections import defaultdict

     dict_ex = defaultdict(object)
     dict_ex = defaultdict(str)
     dict_ex = defaultdict(list)
     dict_ex = defaultdict(set)
     dict_ex = defaultdict(tuple)
     ```
