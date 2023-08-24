---
layout: post
title: "[PyTorch] Tensor의 연속성 - contiguous"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Pytorch, Tensor]
tags: [Pytorch]
comments: true
toc: true
---

## 1. Tensor.contiguous

Contiguous를 번역하면 연속성이라는 뜻입니다. 그렇다면 Tensor의 연속성을 보장한다는 뜻 같은데 Tensor의 연속성은 또 무엇일지 의문이 들 수 있습니다. 정답을 먼저 말하자면 Tensor의 연속성은 Tensor에 있는 데이터의 순서가 연속적이라는 것을 말합니다. 글로 설명하기 보다는 아래의 코드를 통해 `Tensor.contiguous()`를 알아보도록 하겠습니다.

---

## 2. Tensor가 연속성을 보장하는 경우

먼저 tensor 하나를 생성하고 해당 tensor가 연속성을 가지고 있는지 알아보도록 하겠습니다.

```python
A = torch.randn(2, 3)

print(A)
    >>  tensor([[ 0.3312,  0.2217, -1.1354],
                [-0.2663, -1.2885, -0.5531]])

print(f'Tensor A is contiguous >> {A.is_contiguous()}')
    >>  Tensor A is contiguous >> True
```

위의 코드에서 확인하실 수도 있듯이 Tensor A는 연속성을 가지고 있습니다.

그렇다면 먼저 Tensor A의 데이터 순서를 알아보겠습니다.

```python
print(f'Stride of tensor A when dimension is 0 >> {A.stride(dim = 0)}')
    >>  Stride of tensor A when dimension is 0 >> 3

print(f'Stride of tensor A when dimension is 1 >> {A.stride(dim = 1)}')
    >>  Stride of tensor A when dimension is 0 >> 1
```

Tensor A의 데이터가 어떤 순서로 있는지 확인하기 위해 `Tensor.stride()`를 통해 확인하였습니다. `Tensor.stride()`는 dimension 방향으로 현재 요소에서 다음 요소로 이동하는데 필요한 stride를 알려줍니다. 결과에 따르면 세로방향(dimension = 0)에 있는 요소로 이동하기 위해서는 3번의 이동이 필요하고 가로방향(dimension = 1)에 있는 요소로 이동하기 위해서는 1번의 이동이 필요합니다.

정리하자면 Tensor A의 데이터 순서는 `A[0][0] → A[0][1] → A[0][2] → A[1][0] → A[1][1] → A[1][2]`입니다. 해당 순서는 `A[0] → A[1]`로 요약할 수 있고 데이터의 순서가 연속적이라고 할 수 있습니다.

---

## 3. Tensor가 연속성을 보장하지 않는 경우

이와 반대로 연속성을 보장하지 않는 경우도 있을까 싶지만 PyTorch에서는 연속성을 보장하지 않는 경우도 있습니다. 오늘 말하고자 하는 `Tensor.view()`도 그런 함수 중 하나입니다. 그 외에도 `torch.narrow()`, `torch.transpose()`, `Tensor.expand()` 등이 있습니다. 이번에는 연속성을 보장하지 않는 함수를 통해 연속성이 보장하지 않을 때의 데이터 순서를 한 번 살펴 보겠습니다.

먼저 연속성을 보장하지 않는 함수를 적용하고 실제로 연속성을 가지고 있지 않은지 확인해 보겠습니다.

```python
A = torch.randn(2, 3)
print(A)
    >>  tensor([[ 0.4735,  1.9021,  0.7869],
                [-0.3108,  1.4994,  2.4527]])

A.transpose_(0, 1) # 연속성을 보장하지 않는 함수를 적용
print(A)
    >>  tensor([[ 0.4735, -0.3108],
                [ 1.9021,  1.4994],
                [ 0.7869,  2.4527]])

print(f'Tensor A is contiguous >> {A.is_contiguous()}', end = "\n\n")
    >>  Tensor A is contiguous >> False
```

역시 Tensor A는 연속성을 보장하지 않은 것을 확인할 수 있었습니다. 그럼 이번에는 `Tensor.stride()`를 통해 데이터 순서를 알아보겠습니다.

```Python
print(f'Stride of tensor A when dimension is 0 >> {A.stride(dim = 0)}')
    >>  Stride of tensor A when dimension is 0 >> 1

print(f'Stride of tensor A when dimension is 1 >> {A.stride(dim = 1)}')
    >>  Stride of tensor A when dimension is 1 >> 3
```

위의 코드를 고려하여 현재 Tensor A의 데이터 순서를 생각해보면 `A[0][0] → A[1][0] → A[0][1] → A[1][1] → A[0][2] → A[1][2]`입니다.

현재 Tensor A는 연속성을 보장하고 있는 것 같나요? 현재 Tensor A는 데이터의 저장순서가 엉켜있으므로 연속성을 보장하지 않고 있습니다.

⚠️ 이렇게 contiguous가 보장되지 않는 Tensor를 사용할 때는 다음과 같은 에러 메세지를 만날 수 있으니 주의 하셔야 합니다.

```python
A = torch.randn(2, 3)
A.transpose_(0, 1)
A.view(2, 3)
    >>  "RuntimeError: view size is not compatible with input tensor's size and stride (at least one dimension spans across two contiguous subspaces). Use .reshape(...) instead."
```

---

## 4. 연속성을 가지지 않은 Tensor가 연속성을 가지도록 하는 방법

이렇게 연속성이 보장되지 않은 경우 연속성을 보장하기 위해 사용하는 것이 `Tensor.contiguous()`입니다. `Tensor.contiguous()`는 기존의 tensor를 copy하여 연속성을 보장하도록 합니다.

```python
A = torch.randn(2, 3)
print(A)
    >>  tensor([[-0.0246, -0.2377, -0.9833],
                [ 0.6629, -0.5296, -0.1188]])

for i in range(2):
    for j in range(3):
        print(A[i][j].data_ptr(), end = '\t')
    print()

    >>  33110848	33110852	33110856
        33110860	33110864	33110868

A.transpose_(0, 1)
print(A)
    >>  tensor([[-0.0246,  0.6629],
                [-0.2377, -0.5296],
                [-0.9833, -0.1188]])

for i in range(3):
    for j in range(2):
        print(A[i][j].data_ptr(), end = '\t')
    print()

    >>  33110848	33110860
        33110852	33110864
        33110856	33110868

B = A.contiguous().view(2, 3)
print(B)
    >>  tensor([[-0.0246,  0.6629, -0.2377],
                [-0.5296, -0.9833, -0.1188]])

for i in range(2):
    for j in range(3):
        print(B[i][j].data_ptr(), end = '\t')
    print()

    >>  101649920	101649924	101649928
        101649932	101649936	101649940
```

위의 코드를 보시면 Tensor A의 주소가 바뀌지 않다가 연속성을 보장하기 위해 `Tensor.contiguous()`를 적용한 순간 메모리의 주소가 바뀝니다. 이것은 연속성을 보장하기위해 copy를 하였다는 것을 의미합니다.

💭 메모리 용량이 부족하지 않다면 연속성을 보장하는 함수들을 위주로 사용하는 것이 좋다고 생각합니다.

---

## ※ Reference

- [https://jimmy-ai.tistory.com/122](https://jimmy-ai.tistory.com/122)
- [https://titania7777.tistory.com/3](https://titania7777.tistory.com/3)
