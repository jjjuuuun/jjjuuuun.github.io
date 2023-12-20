---
layout: post
title: "[PyTorch] PyTorch에서 tensor를 만드는 방법"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Deep Learning, Pytorch]
tags: [Deep Learning, Pytorch, Tensor]
comments: true
toc: true
---

PyTorch에서는 `torch.Tensor` type으로 tensor를 표현하고 있습니다. 그리고 PyTorch에서는 tensor를 생성하는 여러가지 방법이 있는데 오늘 포스트에서 이 방법들을 알아보려고 합니다.

[PyTorch document](https://pytorch.org/docs/stable/tensors.html#torch.Tensor)에서 소개하는 방법은 크게 5가지 방법이 있습니다. 오늘은 그 분류에 따라 설명해보도록 하겠습니다.

---

## 1. torch.Tensor

첫 번째로는 `torch.Tensor()`를 통해 <U>직접</U> tensor를 생성하는 방법입니다. 여기서 <U>직접</U>이라고 말한 이유는 다음과 같습니다. 이후 설명되는 tensor를 만드는 방법들은 `torch.Tensor `type의 tensor를 만드는 동시에 서로 다른 역할이 존재하기 때문에 그 역할에 맞는 추가적인 조작이 추가적으로 들어가기 때문입니다.

`torch.Tensor()`를 통해 어떻게 tensor를 생성하는지 알아보겠습니다.

```python
A = torch.Tensor([5])
    >>  tensor([5.])

B = torch.Tensor()
    >>  tensor([])
```

`torch.Tensor()`는 사실 `torch.FloatTensor`의 alias입니다. 그렇기 때문에 `torch.Tensor()`는 data type이 기본적으로 32-bit floating point인 tensor를 반환해줍니다.

PyTorch에서는 tensor를 `torch.Tensor` type으로 표현하기 때문에 앞으로 나올 `torch.tensor()`, `torch.*`, `torch.*_like`, `torch.new_*`으로 생성하는 tensor의 data type은 특별한 언급이 없으면 32-bit floating point type으로 tensor가 만들어집니다.

※ PyTorch에서 지원하는 data type이 궁금하신 분들은 [공식 문서](https://pytorch.org/docs/stable/tensors.html#data-types)에서 확인해주세요.

※ Default tensor data type을 바꾸는 방법이 궁금하신 분들은 [공식 문서](https://pytorch.org/docs/stable/generated/torch.set_default_tensor_type.html#torch.set_default_tensor_type)를 참고해 주세요.

---

그렇다면 앞서 말했던 torch.Tensor는 tensor를 직접 만들어내는 방법이고 직접 만들어내지 않는 방법에는 무엇이 있을지 알아보겠습니다.

## 2. torch.tensor()

`torch.tensor()`는 기존 데이터를 tensor로 변경하는 방법입니다.

```python
print(torch.tensor([0, 1]))
    >>  tensor([0, 1])

print(torch.tensor(3.14159))
    >>  tensor(3.1416)

print(torch.tensor([]))
    >>  tensor([])
```

위의 코드에서도 볼 수 있듯이 `torch.tensor()`는 기존의 데이터를 그대로 tensor로 만들어줍니다. 여기서 집중해서 보셔야 할 부분이 있습니다. 바로 tensor의 data type입니다. 이전의 `torch.Tensor()`는 입력으로 어떤 data type이 들어오든 상관없이 Float type으로 바꾸어 tensor를 만들어 주었습니다. 하지만 `torch.tensor()`는 기존 데이터를 그대로 tensor로 변경하기 때문에 data type 또한 기존 데이터와 같게 유지 됩니다.

하지만 기존 데이터가 비어 있어 반환된 tensor 안에 값이 들어있지 않은 경우 즉, 기존 데이터 없이 tensor를 만들었을 경우에는 `torch.Tensor()`와 마찬가지로 Float type인 빈 tensor를 만들어 줍니다. (tensor의 data type은 기본적으로 float type이라고 앞서 이야기한 것과 일맥상통합니다.)

---

## 3. torch.\*

`torch.*`는 특정 크기의 tensor를 만드는 방법입니다.

`torch.*`의 예시로는 `torch.ones()`, `torch.zeros()`, `torch.eye()`, `torch.empty()`등이 있습니다. 이렇게 tensor를 생성하는 방법의 특징은 만들고자 하는 tensor의 shape을 넣어주어야 합니다.

```python
print(torch.ones(2, 3))
    >>  tensor([[1., 1., 1.],
                [1., 1., 1.]])

print(torch.zeros(5))
    >>  tensor([0., 0., 0., 0., 0.])

print(torch.eye(2))
    >>  tensor([[1., 0.],
                [0., 1.]])

print(torch.eye(2, 3))
    >>  tensor([[1., 0., 0.],
                [0., 1., 0.]])

print(torch.empty((2, 3)))
    >>  tensor([[-1.2047e+09,  4.5891e-41,  6.2698e-38],
                [ 0.0000e+00,  3.2461e-28,  4.5891e-41]])
```

---

## 4.torch.\*\_like

`torch.*_like`는 다른 tensor의 크기로 tensor를 만드는 방법입니다.

`torch.*_like`의 예시로는 `torch.ones_like()`, `torch.zeros_like()`등이 있습니다.

```python
input = torch.empty(2, 3)

print(torch.ones_like(input))
    >>  tensor([[1., 1., 1.],
                [1., 1., 1.]])

print(torch.zeros_like(input))
    >>  tensor([[0., 0., 0.],
                [0., 0., 0.]])
```

---

## 5. Tensor.new\_\*

`Tensor.new_*`는 다른 tesnor와 같은 data type을 가지고 있지만 크기가 다른 tensor를 만드는 방법입니다.

`Tensor.new_*`의 예시로는 `Tensor.new_full()`, `Tensor.new_empty()`, `Tensor.new_ones()`, `Tensor.new_zeors()`, `Tensor.new_tensor()`가 있습니다.

`Tensor.new_tensor()`의 경우 다른 `Tensor.new_*`와 다르게 바꾸고 싶은 data를 직접 넣어주어야 하는 것을 헷갈리면 안됩니다.

```python
tensor = torch.tensor((), dtype=torch.int32)

print(tensor.new_full((2, 3), 10))
    >>  tensor([[10, 10, 10],
                [10, 10, 10]], dtype=torch.int32)

print(tensor.new_ones((2, 3)))
    >>  tensor([[1, 1, 1],
                [1, 1, 1]], dtype=torch.int32)

print(tensor.new_zeros((2, 3)))
    >>  tensor([[0, 0, 0],
                [0, 0, 0]], dtype=torch.int32)

print(tensor.new_empty((2, 3)))
    >>  tensor([[-829449120,      32749,  103637808],
                [         0,         32,          0]], dtype=torch.int32)

print(tensor.new_tensor([[0, 1], [2, 3]]))
    >>  tensor([[0, 1],
                [2, 3]], dtype=torch.int32)
```
