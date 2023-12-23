---
layout: post
title: "[fast.ai] fast.ai Overview"
author: kjy
date: 2023-12-23 18:10:00 +9:00
categories: [Deep Learning, fast.ai]
tags: [Deep Learning, fast.ai]
comments: true
toc: true
math: true
---

## 1. fast.ai 설치하기

[fast.ai 공식 홈페이지](https://docs.fast.ai/)를 참고하시면 설치하실 수 있습니다. 다만 코랩을 사용하시는 경우에는 별도의 설치 필요 없이 바로 사용이 가능합니다.

## 2. fast.ai에 대해 알아보기

fast.ai는 PyTorch에 기반을 둔 딥러닝 라이브러리입니다.

- High-level Components
  - SOTA 결과를 빠르게 확인할 수 있도록 실무자들에게 제공
- Low-level Components
  - 혼합 및 매칭하여 새로운 접근 방식을 구축할 수 있도록 연구자들에게 제공

## 3. fast.ai의 목표

fast.ai는 사용 편의성, 유연성, 성능 면에서 실질적인 타협 없이 high-level components와 low-level components를 모두 제공하는 것을 목표로 합니다.

High-level components와 low-level components 모두 제공이 가능한 이유는 fast.ai가 deep learning과 data processing 기술의 공통적인 패턴을 분리된 추상화의 관점에서 표현한 <u>layered architecture</u>이기 때문입니다.

🔎 Layered Architecture

> 소프트웨어 개발에서 흔히 사용되는 architecture의 개념입니다.
>
> 비슷한 개념으로 코드를 작성할 때 응집도, 결합도를 생각해 작성해야 합니다. 왜냐하면 모듈화를 할 때 결합도는 낮을수록 응집도는 높을수록 재사용성과 유지보수성이 좋아지기 때문에 이상적인 모듈이라고 할 수 있습니다.
>
> 여기서 layer를 모듈보다 큰 개념으로 생각하면 될 것 같습니다. 각 layer를 관심사에 따라 분리하고 각 layer의 응집도와 결합도를 낮춰서 application의 재사용성과 유지보수성을 높이기 위해 layered architecture를 사용합니다.

<br>
High-level components와 low-level components를 모두 제공하기 위해 합성 가능한 building block을 low-level API 계층 위에 구축하여 high-level API의 일부를 재작성하거나 새로운 기능을 추가할 수 있도록 했습니다.

다만, 새로운 기능을 만들기 위해서 사용자는 low-level components를 어떻게 사용하는지 배워야 합니다.

# 4. fast.ai의 대표적인 기능들

Python의 역동성과 PyTorch의 유연성을 활용하여 간결하고 명확하게 추상화를 표현할 수 있다고 합니다.

1. Tensor를 위한 의미형 계층구조와 함께 파이썬을 위한 새로운 형태의 <u>dispatch system</u>(🔎 주로 작업이나 리소스를 효율적으로 관리하고 조정하기 위한 시스템)
2. Pure Python으로 확장 가능한 GPU 최적화 computer vision library
3. Optimization algorithm을 4 ~ 5줄의 코드로 구현할 수 있도록 modern optimizer의 공통 기능을 두 개의 기본 조각으로 반영하는 optimizer
4. 데이터, 모델 또는 옵티마이저 모든 부분에 액세스하여 교육 중 언제든지 변경할 수 있는 새로운 양방향 callback system
5. New data block API

# 5. fast.ai에 대한 요약

fast.ai는 간단한 코드로 SOTA 모델을 학습 및 평가를 해볼 수 있는 딥러닝 라이브러리라고 할 수 있습니다. 그렇기 때문에 간단하게 SOTA 모델을 사용해보고 싶을 때 사용할 수 있는 유용한 라이브러리이지 않을까 합니다.

만약 새로운 기능이 필요하다면 만들 수 있지만 직접 기능을 만들기 위해서는 low-level components에 대한 이해가 필요로 합니다. 그렇기 때문에 fast.ai를 통해 간단하게 프로토타입을 만들고 이후 작업은 PyTorch나 Tensorflow를 통해 튜닝해가는 것이 좋지 않을까 합니다.

# ※ Reference

- [Layered Architecture](https://ksh-coding.tistory.com/92)
- [fast.ai](https://docs.fast.ai/)
