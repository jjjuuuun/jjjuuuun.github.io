---
layout: post
title: "[Python] 문자열 함수 - 대소문자 변환"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, String]
tags: [Python]
comments: true
toc: true
---

Python의 [Standard Library에서 지원하는 문자열 함수 중 대소문자 변환과 관련된 함수](https://docs.python.org/3/library/stdtypes.html#text-sequence-type-str)에 대해 알아볼 것이다.

---

1. str.capitalize()

   - 문장 첫 단어의 첫 문자만 대문자로 바꾸고 나머지는 소문자로 바꾸는 함수

   ```python
   string1 = "3python hello wORld"
   string2 = "Python hello wORld"

   print(string1.capitalize())
       >>  3python hello world
   print(string2.capitalize())
       >>  Python hello world
   ```

2. str.title()

   - 문장 모든 단어의 첫 문자만 대문자로 바꾸고 나머지는 소문자로 바꾸는 함수
   - string1에서 볼 수 있듯이 숫자는 첫 문자로 인식하지 않는다.

   ```python
   string1 = "3python hello wORld"
   string1_1 = "333python hello wORld"
   string2 = "!!!python hello wORld"
   string3 = "Python hello wORld"

   print(string1.title())
       >>  3Python Hello World
   print(string1_1.title())
       >>  333Python Hello World
   print(string2.title())
       >>  !!!Python Hello World
   print(string3.title())
       >>  Python Hello World
   ```

3. str.swapcase()

   - 대문자는 소문자로, 소문자는 대문자로 치환하는 함수

   ```python
   string1 = "3python hello wORld"

   print(string1.swapcase())
       >>  3PYTHON HELLO WorLD
   ```

4. str.upper()

   - 모든 문자를 대문자로 치환하는 함수

   ```python
   string1 = "3python hello wORld"

   print(string1.upper())
       >>  3PYTHON HELLO WORLD
   ```

5. str.lower()

   - 모든 문자를 소문자로 치환하는 함수

   ```python
   string1 = "3python hello wORld"

   print(string1.lower())
       >>  3python hello world
   ```
