---
layout: post
title: "[Model Compression] Model Compression Overview"
author: kjy
date: 2023-12-20 13:04:00 +9:00
categories: [Deep Learning, Model Compression]
tags: [Deep Learning, Model Compression]
comments: true
toc: true
math: true
---

최근에 경량화와 관련된 이야기가 많아 한 번 공부를 해보려고 합니다. [네이버 부스트코스에 좋은 강의](https://www.boostcourse.org/ai302/lecture/1474495?isDesc=false)가 있어 해당 강의를 참고하여 공부하므로 해당 포스트에 정리되는 내용 또한 부스트 코스를 참고하였음을 알려드립니다.

이번 포스트에서는 경량화의 목적과 대략적인 분류에 대해 말씀드리고 넘어가도록 하겠습니다.

---

## 경량화의 목적

1. On device AI

   요즘 AI는 모든 곳에 사용된다고 봐도 무방할 정도로 넓게 사용되고 있습니다. 그 중에서도 생활에 밀접한 곳에서 AI의 활용도가 높아지고 있는데 그 예시로 스마트 폰, 스마트 워치, IoT를 말할 수 있을 것 같습니다. 이처럼 생활에 밀접한 디바이스에는 공통적인 특징이 있습니다. 바로 제한된 크기입니다. 들고 다녀야 하고 제한된 공간에 존재해야 하기 때문에 크기가 제한될 수 밖에 없습니다. 생활적인 측면에서는 좋을 수 있지만 AI 측면에서는 좋은 것만은 아닙니다.

   데이터가 많을수록 모델이 클수록 좋은 성능을 보인다는 것은 모두들 알고 계실 것 같습니다. 그러나 위에서 말씀드린 디바이스는 크기가 제한되어 있어 배터리의 제한, 저장 공간의 제한 등이 따를 수 밖에 없습니다. 따라서 큰 모델을 디바이스에서 사용하기에는 무리가 따릅니다. 이러한 필요에 의해서 경량화가 점점 중요해지고 있습니다.

2. AI on cloud (or server)

   한정된 자원으로 더 적은 latency와 더 큰 throughput이 가능하다면 더 큰 이득을 볼 수 있습니다.

   1번에서 말한 디바이스와 관련된 문제가 하드웨어적인 문제 때문에 경량화가 필요하다면 두 번째는 현실적인 문제 때문에 경량화가 필요한 것 같습니다.

   Cloud 또는 Server를 사용하는 데에도 비용이 들기 때문에 더 적은 latency와 더 큰 throughput이 가능하다면 더 큰 이득을 볼 수 있기 때문에 경량화가 필요합니다.

➡️ 1번과 2번에서 말씀드린 경량화가 필요한 이유가 결국은 AI의 연산량이 많기 때문이라고 정리할 수 있을 것 같습니다. 따라서 computation cost를 줄이는 것이 경량화의 목적이라고 할 수 있습니다.

## 경량화 대표적인 종류

Computation cost를 줄이기 위해서는 최적화 되지 않은 부분을 최적화 함으로써 cost를 줄일 수 있고 직접적으로 불필요한 계산을 제거함으로써 cost를 줄일 수 있습니다.

이처럼 경량화에는 다양한 방법이 있고 현재까지도 연구중인 것으로 알고 있습니다. 이중에서 대표적인 종류를 두 가지 관점에서 살펴보고자 합니다.

1. 네트워크 구조 관점

   - [Efficient Architecture Design]()
     - 더 효율적인 블록 모듈을 디자인 하는 것을 말합니다.
     - AutoML을 사용해서 디자인 할 수 있는데 사람의 직관보다 상회하는 모듈을 찾아낼 수 있습니다.
   - [Network Pruning]()
     - 가지치기를 의미합니다. 학습한 모델이 있을 때 중요도가 낮은 파라미터를 제거해서 사이즈를 줄여보고자 하는 것을 의미합니다.
     - 이 분야는 좋은 중요도를 정의하고 찾는 것이 주요 연구 토픽 중 하나입니다.
     - 크게 두 가지 세부 분야로 나뉘는데 구조적 가지치기와 비구조적 가지치기로 나뉘어집니다.
   - Knowledge Distillation
     - 지식 증류는 학습된 큰 모델(Teacher Model)이 존재할 때, 작은 모델(Student Model)에 지식을 전달하는 접근법입니다.
     - 두 모델의 soft outputs를 사용해서 지식을 전달합니다.
   - Matrix / Tensor Decomposition
     - 학습된 모델이 있을 때 가중치들을 더 작은 단위의 벡터나 텐서의 곱으로 표현하는 방법입니다.
     - 결국, 하나의 텐서를 작은 텐서들의 조합으로 표현하는 것을 말합니다.

2. Hardware 관점
   - [Network Quantization]()
     - 일반적인 float32 데이터 타입의 network의 연산과정을 그 보다 작은 크기의 데이터 타입으로 변환하여 연산을 수행하는 것을 의미합니다.
   - [Network Compiling]()
     - 타겟 하드웨어가 정해져 있을 때, 모델이 더 효과적으로 수행될 수 있도록 네트워크를 컴파일링 하는 것을 의미합니다.

➡️ 위의 경량화 분야중에서 앞으로 Matrix Pruning, Knowledge Distillation을 제외한 총 4가지를 살펴보고자 합니다.

## ※ Reference

[https://www.boostcourse.org/ai302/lecture/1474495?isDesc=false](https://www.boostcourse.org/ai302/lecture/1474495?isDesc=false)
