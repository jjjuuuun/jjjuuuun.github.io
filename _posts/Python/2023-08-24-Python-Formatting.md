---
layout: post
title: "[Python] Formatting"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Python, Formatting]
tags: [Python]
comments: true
toc: true
---

Python에서 지원하는 다양한 형태의 formatting에 대해 알아보려고 합니다.

---

현재 Python3에서 지원하는 formatting 방법에는 % string, formt함수 사용, fstring이 있습니다.

## 1. % string을 사용해 formatting 하는 방법

% string은 python 초기에 사용되던 방법으로 C언어에서 사용하는 방법과 유사하기 때문에 파이썬 답지 않게 굉장히 불편한 점이 있습니다. 예를들면 %d, %s과 같은 출력 포맷을 정확히 작성해 주어야 하기 때문에 dynamic typing을 지원해주는 파이썬답지 않은 formatting입니다. 따라서 간단하게만 살펴보고 지나가겠습니다.

- 기본적인 형식

  ```python
  a = 3; b = "three"

  print("%d, %s" %(a, b))
      >>  3, three
  print("%f." %5.243)
      >>  5.243000.
  ```

- 소수점의 자리를 조절 하고 싶을 때

  ```python
  print("%.5f." %5.243)
      >>  5.24300.
  ```

- 전체 자릿수를 조절 하고 싶을 때

  ```python
  print("%10.5f." %5.243)
      >>     5.24300.
  ```

## 2. format 함수를 사용해 formatting 하는 방법

% string 이후 좀 더 편리하게 파이썬답게 사용하기 위해 나온 formatting 방법이 format 함수를 사용하는 것입니다. format 함수에 대해 알아보겠습니다.

- 기본적인 사용 방법

  ```python
  print("{0}".format(3))
      >>  3
  ```

  ```python
  a = 3; b = "three"

  print("{0}, {1}".format(a, b))
      >>  3, three
  ```

  🔎 { } 괄호 안에 꼭 숫자를 명시를 안해도 실행이 됩니다. 그러나 명시를 해주지 않는 경우 format 함수에 들어오는 인자가 순서대로 들어가니 format 함수에 인자를 작성하실 때 주의해서 순서대로 입력해주셔야 합니다.

  ```python
  print("{}, {}".format(a, b))
      >>  3, three
  ```

- 소수점을 다룰 때

  ```python
  print("{0:.5f}.".format(5.243))
      >>  5.24300.
  print("{:.5f}.".format(5.243))
      >>  5.24300.
  ```

- 텍스트 정렬과 너비 지정할 때

  ✅ 졍렬을 진행하는 옵션

  | 옵션 |    의미     |
  | :--: | :---------: |
  | "<"  |  왼쪽 정렬  |
  | ">"  | 오른쪽 정렬 |
  | "^"  | 가운데 정렬 |

  ⚠️ 출력하려는 출력값의 길이보다 큰 field가 정의되어 있지 않으면 해당 옵션을 사용해도 결과값은 변하지 않습니다. field를 지정하는 방법은 option뒤에 혹은 콜른(:) 뒤에 field의 길이를 입력해주시면 됩니다.

  ```python
  print("{:10.5f}.".format(5.243))
      >>     5.24300.
  print("{:>10.5f}.".format(5.243))
      >>     5.24300.
  print("{:<10.5f}.".format(5.243))
      >>  5.24300   .
  print("{:^10.5f}.".format(5.243))
      >>   5.24300  .
  print("{:+^10.5f}.".format(5.243))
      >>  +5.24300++.
  ```

## 3. f-string를 사용해 formatting 하는 방법

f-string은 파이썬 버전 3.6에 추가되어 현재 가장 많이 사용되고 있는 formatting 방법입니다. 위에 두 개보다 편리하고 가독성 또한 올라가는 장점 때문에 가장 많이 사용되지 않나 싶습니다. f-string 사용 방법은 format함수 사용법과 크게 다르지 않습니다. format 함수에서 함수에 넣던 real value를 f-string에선 { } 안에 직접 넣어주시면 됩니다. 따라서 예시만 살펴보고 넘어가겠습니다.

```python
a = 3; b = "three"

print(f"{a}, {b}")
    >>  3, three
```

```python
print(f"{5.243:.5f}.")
    >>  5.24300.
print(f"{5.243:10.5f}.")
    >>     5.24300.
```

```python
print(f"{5.243:>10.5f}.")
    >>     5.24300.
print(f"{5.243:<10.5f}.")
    >>  5.24300   .
print(f"{5.243:^10.5f}.")
    >>   5.24300  .
print(f"{5.243:+^10.5f}.")
    >>  +5.24300++.
```

---

※ Reference

- [format 함수](https://docs.python.org/ko/3/library/string.html#format-string-syntax)

- [f-string](https://docs.python.org/ko/3/reference/lexical_analysis.html#f-strings)
