---
layout: post
title: "[Generative Model] DDPM : Denoising Diffusion Probabilistic Models"
author: kjy
date: 2024-01-03 16:00:00 +09:00
categories: [Deep Learning, Paper Reading]
tags: [Deep Learning, Paper Reading, Computer Vision, Generative Model]
comments: true
toc: true
math: true
---

해당 포스트에서는 [DDPM : Denoising Diffusion Probabilistic Models](https://arxiv.org/abs/2006.11239) 논문을 함께 읽어가도록 하겠습니다. 문장마다 해석을 하기 보다는 각 문단에서 중요한 부분을 요약하는 식으로 진행하겠습니다.

## Abstract

![](../../assets/img/Paper_Reading/DDPM/ddpm_1.jpg){: width="400" .left}

🔎 해당 논문은 diffusion probability models을 사용해 높은 퀄리티의 이미지를 합성했음을 보여줍니다.

💡 [Nonequilibrium thermodynamics](https://ko.wikipedia.org/wiki/%EB%B9%84%ED%8F%89%ED%98%95%EC%97%B4%EC%97%AD%ED%95%99)

❓ Diffusion probabilistic models와 denoising score matching의 새로운 연결

❓ Generalization of autoregressive decoding

![](../../assets/img/Paper_Reading/blank.png){: .normal}

## 1. Introduction

![](../../assets/img/Paper_Reading/DDPM/ddpm_2.jpg){: width="400" .left}

🔎 Generative models : `GANs`, `Autoregressive models`, `flows`, `VAEs`, `Energy-based modeling`, `Score matching`

![](../../assets/img/Paper_Reading/blank.png){: .normal}

![](../../assets/img/Paper_Reading/DDPM/ddpm_3.jpg){: width="400" .left}

🔎 해당 논문에서는 diffusion probabilistic models(diffusion model)의 progress를 제안합니다.

🔎 Diffusion model은 유한한 시간이후에 data와 일치하는 samples을 생성하기 위해 `variational inference`를 사용하여 훈련된 `parameterized Markov chain`입니다.

💡 Variational inference : Posteroir distribution $p(\theta\|x)$를 다루기 쉬운 매개변수 $\theta$를 조정하여 확률분포 $q(\theta)$로 approximation 하는 것

💡 Parameterized Markov chain : Markov property를 가진 이산확률과정

💡 Markov property : 특정 상태의 확률은 오직 과거의 상태에 의존

🔎 `Reverse a diffusion process`를 통해 학습 : <u>원래의 signal(image)이 파괴될 때까지 sampling의 반대방향으로 점진적으로 data에 noise를 추가하는 markov chain</u>을 통해 학습

❓ Sampling의 방향?

![](../../assets/img/Paper_Reading/blank.png){: .normal}

![](../../assets/img/Paper_Reading/DDPM/ddpm_4.jpg){: width="400" .left}

🔎 기존 연구 : Diffusion model이 정의하고 학습하기에는 수월하지만 좋은 결과를 얻지는 못했습니다.

🔎 해당 논문에서 보여주는 결과

1️⃣ Diffusion model의 특정 매개변수화가 훈련 중 multiple noise level에 대한 denoising score matching과 sampling 중 annealed Langevin dynamics와 등가임을 발견

2️⃣ 특정 매개변수화를 통해 얻은 좋은 품질의 결과

3️⃣ Log likelihoods : Energy based models & score matching < Diffusion model < Other likelihood-based models

4️⃣ Diffusion model의 많은 lossless codelengths가 이미지 세부 정보를 설명하는데 사용

5️⃣ Sampling procedure of diffusion models : Type of progressive decoding

❓ 아직 해결되지 않은 부분이 있으나 뒤에 설명이 있을 것 같아 일단은 넘어가겠습니다.

![](../../assets/img/Paper_Reading/blank.png){: .normal}

## 2. Background

![](../../assets/img/Paper_Reading/DDPM/ddpm_5.jpg){: width="400" .left}

1️⃣ Reverse Process : Standard normal distribution에서 학습 data로 denoising 해가는 과정이며 학습의 대상이라고 볼 수 있습니다.

2️⃣ Forward Process(Diffusion process) : Data에서 noise를 추가해 standard normal distribution으로 가는 과정입니다.

🔎 $\beta_t$는 학습할 수도 있고 상수로 사용할 수 있습니다.

❓ $\beta_t$가 작을 때 왜 reverse process, forward process 모두 동일한 기능적인 형태를 갖는 것일까요?

❓ Forward process는 임의의 시간 $t$에서 $\mathbf{x}_t$ sampling을 허용한다?

![](../../assets/img/Paper_Reading/blank.png){: .normal}

### 2.1 Reverse Process

$$
\begin{align}
p_\theta(\mathbf{x}_{0:T})
&= p_\theta(\mathbf{x}_0|\mathbf{x}_{1:T})\cdot p_\theta(\mathbf{x}_{1:T}) \tag{1} \\
&= p_\theta(\mathbf{x}_0|\mathbf{x}_{1:T})\cdot p_\theta(\mathbf{x}_1|\mathbf{x}_{2:T})\cdot p_\theta(\mathbf{x}_{2:T}) \tag{2} \\
& \qquad \vdots \\
&= p_\theta(\mathbf{x}_0|\mathbf{x}_{1:T})\cdot p_\theta(\mathbf{x}_1|\mathbf{x}_{2:T})\cdots p_\theta(\mathbf{x}_{T-1}|\mathbf{x}_T)\cdot P_\theta(\mathbf{x}_T) \tag{3} \\
&= p_\theta(\mathbf{x}_0|\mathbf{x}_1)\cdot p_\theta(\mathbf{x}_1|\mathbf{x}_2)\cdots p_\theta(\mathbf{x}_{T-1}|\mathbf{x}_T)\cdot p_\theta(\mathbf{x}_T) \tag{4} \\
&= p_\theta(\mathbf{x}_0|\mathbf{x}_1)\cdot p_\theta(\mathbf{x}_1|\mathbf{x}_2)\cdots p_\theta(\mathbf{x}_{T-1}|\mathbf{x}_T)\cdot p(\mathbf{x}_T) \tag{5} \\
&= p(\mathbf{x}_T)\cdot \prod_{t=1}^T p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t) \tag{6}
\end{align}
$$

#### 2.1.1 Formula (1), (2), (3)

먼저 표기법에 대해 알아보면 $p_\theta(\mathbf{x}\_{0:T})$는 매개변수 $\theta, \mathbf{x}\_0, \mathbf{x}\_1, \cdots, \mathbf{x}\_T$의 결합확률에서 $\theta$를 고정시켰을 때의 주변확률이라고 할 수 있습니다.

아래에 기술되어 있는 공식(chain rule)을 사용하면 `(1)`, `(2)`, `(3)`을 쉽게 이해할 수 있습니다.

$$
\begin{align} P(A, B, C) &= P(A \cap B \cap C) \\
&= P(A \cap (B \cap C)) \\
&= P(A | B \cap C)P(B \cap C) \\
&= P(A | B, C)P(B, C)
\end{align}
$$

#### 2.1.2 Formula (4)

Reverse process는 markov property를 만족하기 때문에 markov process에 의해서 `(4)`처럼 `(3)`을 변경할 수 있습니다.

#### 2.1.3 Formula (5)

Timestep $T$에서의 $\mathbf{x}_T$는 $\theta$와 상관없이 $\mathcal{N}(0, I)$이기 때문에 $\theta$를 지워 `(5)`처럼 나타낼 수 있습니다.

#### 2.1.4 Formula (6)

`(5)`를 일반화 시키면 `(6)`으로 정리할 수 있습니다.

### 2.2 Forward Process (Diffusion Process)

$$
\begin{align}
q(\mathbf{x}_{1:T}|\mathbf{x}_0)
&= q(\mathbf{x}_{2:T}|\mathbf{x}_0, \mathbf{x}_1)\cdot q(\mathbf{x}_1|\mathbf{x}_0) \tag{1}\\
&= q(\mathbf{x}_{3:T}|\mathbf{x}_{0:2})\cdot q(\mathbf{x}_2|\mathbf{x}_0, \mathbf{x}_1)\cdot q(\mathbf{x}_1|\mathbf{x}_0) \tag{2}\\
& \qquad \vdots \\
&= q(\mathbf{x}_T|\mathbf{x}_{0:T-1})\cdot q(\mathbf{x}_{T-1}|\mathbf{x}_{0:T-2})\cdots q(\mathbf{x}_1|\mathbf{x}_0) \tag{3}\\
&= q(\mathbf{x}_T|\mathbf{x}_{T-1})\cdot q(\mathbf{x}_{T-1}|\mathbf{x}_{T-2})\cdots q(\mathbf{x}_1|\mathbf{x}_0) \tag{4}\\
&= \prod_{t=1}^T q(\mathbf{x}_t|\mathbf{x}_{t-1}) \tag{5}
\end{align}
$$
