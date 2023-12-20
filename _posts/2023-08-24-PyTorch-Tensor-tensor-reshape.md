---
layout: post
title: "[PyTorch] Tensor의 shape을 바꾸는 방법 - view, reshape"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Deep Learning, Pytorch]
tags: [Deep Learning, Pytorch, Tensor]
comments: true
toc: true
---

PyTorch에서 tensor의 크기를 유지한채 shape을 바꾸는 함수에는 `Tensor.view()`, `torch.reshpae()`가 있습니다. 이번 포스트에서는 두 함수의 사용법과 차이점을 알아보겠습니다.

---

## 1. Tensor.view

`Tensor.view()`의 경우 새로운 Tensor를 생성하는 것이 아니라 기존의 Tensor에서 메타데이터만 수정하여 return 해줍니다. 그러다보니 Input tensor와 output tensor 중 하나의 tensor에서 수정이 일어나면 두 tensor 모두 수정되지만 연속성은 보장되지 않을 수 있습니다.

우선, 메타데이터가 어떻게 수정되었는지 확인해보겠습니다.

```python
A = torch.zeros(3, 2)

for i in range(3):
    for j in range(2):
        print(A[i][j].data_ptr(), end = '\t')
    print()

    >>  112727872	112727876
        112727880	112727884
        112727888	112727892

B = A.view(2, 3)

for i in range(2):
    for j in range(3):
        print(B[i][j].data_ptr(), end = '\t')
    print()

    >>  112727872	112727876	112727880
        112727884	112727888	112727892
```

Tensor A가 가지고 있던 메모리 주소는 그대로 사용하되 모양만 바꿔준 것을 Tensor B를 통해 확인할 수 있습니다. 두 Tensor는 메모리 주소를 공유하기 때문에 두 Tensor 중 하나의 요소를 변경하면 나머지 하나도 변경 될 것을 알 수 있습니다.

위의 코드에서는 연속성이 깨지지 않았지만 `Tensor.view()`는 연속성을 보장하지 않으므로 조심하셔야 합니다.

---

## 2. torch.reshape

`torch.reshape()`의 경우 tensor의 shape을 바꿀 때 연속성이 깨진다면 copy를 하고 그렇지 않다면 `Tensor.view()`와 같이 작동합니다.

먼저 연속성이 깨지지 않아 `torch.reshape()`이 `Tensor.view()`와 같은 방식으로 작동하는 것을 코드를 통해 알아보겠습니다.

```python
A = torch.randn(2, 3)

for i in range(2):
    for j in range(3):
        print(A[i][j].data_ptr(), end = '\t')
    print()

    >>  112729792	112729796	112729800
        112729804	112729808	112729812

B = A.reshape(3, 2)

for i in range(3):
    for j in range(2):
        print(B[i][j].data_ptr(), end = '\t')
    print()

    >>  112729792	112729796
        112729800	112729804
        112729808	112729812
```

`Tensor.view()`와 같이 메모리가 공유되는 것을 확인하실 수 있습니다. 그렇다면 이번에는 연속성을 보장하지 않도록 한 다음 `torch.reshape()`을 사용해 보도록 하겠습니다.

```python
A = torch.randn(2, 3)

for i in range(2):
    for j in range(3):
        print(A[i][j].data_ptr(), end = '\t')
    print()

    >>  112709184	112709188	112709192
        112709196	112709200	112709204

A.transpose_(0, 1) # 연속성이 깨지는 순간

for i in range(3):
    for j in range(2):
        print(A[i][j].data_ptr(), end = '\t')
    print()

    >>  112709184	112709196
        112709188	112709200
        112709192	112709204

B = A.reshape(2, 3) # copy 해버림

for i in range(2):
    for j in range(3):
        print(B[i][j].data_ptr(), end = '\t')
    print()

    >>  112715008	112715012	112715016
        112715020	112715024	112715028
```

위의 코드에서도 볼 수 있듯이 `torch.reshape()`을 할 때 연속성이 보장되어 있지 않다보니 copy를 하는 것을 메모리 주소가 바뀐 점에서 알 수 있습니다.

💭 개인적인 의견으로는 메모리 용량이 부족하지 않다면 `torch.reshape()`을 사용하는 것이 연속성 오류를 안일으키는 측면에서 `Tensor.view()`보다 더 낫다고 생각합니다.

---
