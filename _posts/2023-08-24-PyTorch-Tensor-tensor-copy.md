---
layout: post
title: "[PyTorch] PyTorch에서 tensor를 copy하는 방법"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Pytorch, Tensor]
tags: [Pytorch]
comments: true
toc: true
---

PyTorch에서 사용할 수 있는 copy 방법에 대해서 알아보려고 합니다. PyTorch에서 tensor를 복사하는 방법에는 `torch.clone()`과 `torch.detach()`가 있습니다.

---

## 1. torch.clone

`torch.clone()`은 input tensor를 copy함과 동시에 autograd history를 같이 가져옵니다.

먼저 `requires_grad = False`인 경우를 먼저 살펴보겠습니다.

```python
A = torch.tensor([0., 1.])
B = A.clone()

print(A, A.requires_grad)
    >>  tensor([0., 1.]) False

print(B, B.requires_grad)
    >>  tensor([0., 1.]) False
```

A가 `requires_grad가 False`이니깐 A를 copy한 B 또한 `requires_grad`가 `False`인 것을 확인할 수 있습니다.

그렇다면 이번엔 `requires_grad`가 `True`인 경우를 살펴보도록 하겠습니다.

```python
A = torch.tensor([0., 1.], requires_grad = True)
B = A.clone()

print(A, A.requires_grad)
    >>  tensor([0., 1.], requires_grad=True) True

print(B, B.requires_grad)
    >>  tensor([0., 1.], grad_fn=<CloneBackward0>) True
```

예상하신대로 A의 `requires_grad`가 `True`이다보니 A를 copy한 B 또한 `requires_grad`가 `True`인 것을 확인할 수 있습니다.

---

## 2. torch.detach

`torch.detach()`는 data를 복사하기 보다는 storage를 공유하는 방식을 취하고 있기 때문에 input과 output이 서로 영향을 미칩니다. 이 때 autograd history는 가져오지 않습니다.

먼저 data type은 그대로 가져오는지 확인해 보겠습니다.

```python
A = torch.tensor([0, 1])
B = A.detach()

print(A, A.dtype)
    >>  tensor([0, 1]) torch.int64

print(B, B.dtype)
    >>  tensor([0, 1]) torch.int64
```

위의 코드 결과에서도 볼 수 있듯이 `torch.detach()`를 할 때 data type도 그대로 유지가 되는 것을 알 수 있습니다.

이번에는 autograd history를 가져오는지 확인해 보겠습니다.

```python
A = torch.tensor([0., 1.], requires_grad=True)
B = A.detach()

print(A, A.requires_grad)
    >>  tensor([0., 1.], requires_grad=True) True

print(B, B.requires_grad)
    >>  tensor([0., 1.]) False
```

B의 `requires_grad`가 `False`이므로 A의 autograd를 가져오지 않았음을 알 수 있습니다.

그렇다면 이번에는 `torch.detach()`를 할 때 storage를 공유하는지 한 번 알아보겠습니다.

```python
A = torch.tensor([0., 1.], requires_grad=True)
B = A.detach()

for i in range(2):
    print(A[i].data_ptr(), end = '\t')

    >>  114949632	114949636


for i in range(2):
    print(B[i].data_ptr(), end = '\t')

    >>  114949632	114949636
```

위의 코드에서 살펴볼 수 있듯이 A와 B는 메모리 공간을 공유하고 있습니다. 그렇다면 B[0]의 값을 변경했을 때 A[0] 값이 변경되야 할 텐데 이 부분을 코드로 살펴보겠습니다.

```python
A = torch.tensor([0., 1.], requires_grad=True)
B = A.detach()

B[0] = 3

print(A, A.requires_grad)
    >>  tensor([3., 1.], requires_grad=True) True

print(B, B.requires_grad)
    >>  tensor([3., 1.]) False
```

B 요소의 값을 변경하니 A 요소의 값이 변경된 것을 알 수 있었습니다.

그렇다면 거꾸로 A 요소의 값을 위에와 같이 변경하면 B 요소의 값이 변경된 것을 확인하기 전에 에러 메세지가 뜰 것입니다. 그 이유는 A의 경우 leaf Tensor라서 in-place operation이 수행이 되지 않습니다. (leaf Tensor가 궁금하신 분은 포스트 맨 아래에서 확인하실 수 있습니다.)

따라서 in-place operation이 아닌 operation을 적용하여 값을 변경 시켜봐야 합니다. 그러나 in-place operation이 아닌 operation을 수행하게 되면 A와 A를 copy한 B와의 관계를 알아보는 테스트를 진행할 수 없습니다.

이러한 이유 때문에 B를 변경하여 `torch.detach()`의 storage 공유를 확인하였습니다.

---

### 🔎 leaf Tensor란?

- `requires_grad`가 False인 모든 Tensor는 leaf Tensor라고 말합니다. 또한 처음 생성된 Tensor를 leaf Tensor라고 합니다.
- 정리하자면, `requires_grad`가 False인 경우는 어떠한 연산(operation)을 수행하더라도 해당 Tensor는 leaf Tensor이며 `requires_grad`가 True로 Tensor가 처음 생성 되었을 때는 leaf Tensor이지만 해당 Tensor에 추가적인 연산(operation)을 수행하게 되면 해당 Tensor는 더 이상 leaf Tensor가 아니게 됩니다.
- leaf Tensor인 경우에만 `backward()` 호출시 grad 값을 저장하는데 만약 leaf Tensor가 아닌 Tensor에 대해서 grad 값을 계산하고 저장하고 싶다면 `retain_grad()`를 사용하면 됩니다.
