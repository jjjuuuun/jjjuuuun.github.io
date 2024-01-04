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

🔎 Diffusion model은 유한한 시간이후에 data와 일치하는 samples을 생성하기 위해 [`variational inference`](#231-variational-inference)를 사용하여 훈련된 `parameterized Markov chain`입니다.

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

### 2.1 Reverse Process

![](../../assets/img/Paper_Reading/DDPM/ddpm_5.jpg){: .normal}

🔎 Reverse Process : Standard normal distribution에서 학습 data로 denoising 해가는 과정이며 학습의 대상이라고 볼 수 있습니다.

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

![](../../assets/img/Paper_Reading/DDPM/ddpm_6.jpg){: .normal}

🔎 Forward Process(Diffusion process) : Data에서 noise를 추가해 standard normal distribution으로 가는 과정입니다.

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

#### 2.2.1 Formula (1), (2), (3)

아래에 기술되어 있는 공식(chain rule)을 사용하면 `(1)`, `(2)`, `(3)`을 쉽게 이해할 수 있습니다.

$$
\begin{align}
P(A, B, C | D)
&= P(A \cap B \cap C | D) \\
&= \frac{P(A \cap B \cap C \cap D)}{P(D)} \\
&= P(A \cap B | C \cap D)\cdot \frac{P(C \cap D)}{P(D)} \\
&= P(A, B | C, D)\cdot P(C | D) \\
\end{align}
$$

#### 2.2.2 Formula (4)

Forward process는 markov property를 만족하기 때문에 markov process에 의해서 `(4)`처럼 `(3)`을 변경할 수 있습니다.

#### 2.2.3 Formula (5)

`(4)`를 일반화 시키면 `(5)`으로 정리할 수 있습니다.

### 2.3 Optimization Function

![](../../assets/img/Paper_Reading/DDPM/ddpm_7.jpg){: .normal}

🔎 NLL(Negative Log Likelihood)의 variational bound를 사용해 최적화를 수행합니다.

💡 Log Likelihood를 Negative Log Likelihood로 변경한 이유 : 우도를 최대화하는 문제를 최소화하는 문제로 바꾸기 위해

#### 2.3.1 Variational Inference

1. Variational transform

   - 비선형 함수 $f(x)$를 다루기 쉬운 선형 함수 $g(x)$로 바꾸는 것을 variational transform이라 합니다.
   - Variational transform의 한 예로 $f(x) = \log$함수를 일차함수로 변경해보겠습니다.
     > - $\log$ 함수를 일차함수로 나타낸다는 것은 $\log$함수 위에 있는 모든 점을 일차함수로 표현할 수 있음을 뜻합니다.
     > - $\log$ 함수의 접선은 $g(x) = \lambda x + b(\lambda)$로 나타낼 수 있을 것입니다.
     > - 이 때 일차함수인 접선의 절편은 기울기 $\lambda$에 따라 달라지기 때문에 $b(\lambda)$로 표현됩니다.
     > - 접점을 찾는 공식은 아래와 같이 나타낼 수 있습니다.
     >
     > $$\begin{align}y = \underset{x}{\text{min}} \{g(x) - f(x)\}\end{align}$$
     >
     > - 이제 접점 구해보겠습니다.(간단하게 $\log$함수이 밑이 $10$이라 가정)
     >
     > $$\begin{align} \frac{d}{dx} \{ g(x) - f(x) \} &=& 0 \\ \frac{d}{dx} \{ \lambda x + b(\lambda) - \ln\ x\} &=& 0 \\ \lambda - \frac{1}{x} &=& 0 \\ x &=& \frac{1}{\lambda}  \end{align}$$
     >
     > - 정리를 해보면 $x = \frac{1}{\lambda}$인 모든 $x$좌표에서 $\log$ 함수를 일차함수로 표현할 수 있습니다.
   - 그러나 variationa transform에서 중요한 특징이 하나 있습니다.
   - 그것은 바로 함수가 convex하거나 concave해야만 가능합니다.
     > 만약 비선형 함수 $f(x)$가 convex하지 않고 concave하지도 않다면 접선 $g(x)$에는 두 개 이상의 접점이 생기기 때문입니다.
   - 이러한 특징 때문에 duality를 통해 variational tranform이 가능합니다.
     > - 만약 convex하지 않거나 concave하지 않은 함수가 있다면 $\log$를 사용해 concave한 함수를 만들어 사용할 수 있습니다.
     > - Deep Learning을 공부하다보면 $\log$를 사용하는 이유에는 곱을 합으로 변경해주는 이유도 있지만 concave한 특징을 사용해 variational transform하는 이유도 있다는 것을 기억하면 좋을 것 같습니다.

2. Variational Inference

   - Variational inference는 variational transform을 사용해 Posteroir distribution $p(\theta\|x)$를 다루기 쉬운 매개변수 $\theta$를 조정하여 확률분포 $q(\theta)$로 approximation 하는 것을 말합니다.

#### 2.3.2 Variational Inference at DDPM

DDPM에서 variational inference을 사용하는 공식을 통해 살펴보겠습니다.

$$
\begin{align}
\mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p_\theta(\mathbf{x}_0)\right]

&= \int(-\log\ p_\theta(\mathbf{x}_0))\cdot q(\mathbf{x}_T|\mathbf{x}_0)\ d\mathbf{x}_T \qquad & \because \text{Monte Carlo Integration} \\

&= \int(-\log\ \frac{p_\theta(\mathbf{x}_{0:T})}{p_\theta(\mathbf{x}_{1:T}|\mathbf{x}_0)}) \cdot q(\mathbf{x}_T|\mathbf{x}_0)\ d\mathbf{x}_T \qquad & \because \text{Chain Rule} \\

&= \int(-\log\ \frac{p_\theta(\mathbf{x}_{0:T})}{p_\theta(\mathbf{x}_{1:T}|\mathbf{x}_0)} \cdot \frac{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)})\cdot q(\mathbf{x}_T|\mathbf{x}_0)\ d\mathbf{x}_T \qquad & \\

&= \int(-\log\ \frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)} \cdot \frac{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}{p_\theta(\mathbf{x}_{1:T}|\mathbf{x}_0)})\cdot q(\mathbf{x}_T|\mathbf{x}_0)\ d\mathbf{x}_T \qquad & \\

&= \int(-\log\ \frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)})\cdot q(\mathbf{x}_T|\mathbf{x}_0)\ d\mathbf{x}_T + \int(-\log\ \frac{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}{p_\theta(\mathbf{x}_{1:T}|\mathbf{x}_0)})\cdot q(\mathbf{x}_T|\mathbf{x}_0)\ d\mathbf{x}_T \qquad & \because \log\ a\cdot b = \log\ a + \log\ b \\

&= \int(-\log\ \frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)})\cdot q(\mathbf{x}_T|\mathbf{x}_0)\ d\mathbf{x}_T - \int(\log\ \frac{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}{p_\theta(\mathbf{x}_{1:T}|\mathbf{x}_0)})\cdot q(\mathbf{x}_T|\mathbf{x}_0)\ d\mathbf{x}_T \qquad & \\

&= \int(-\log\ \frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)})\cdot q(\mathbf{x}_T|\mathbf{x}_0)\ d\mathbf{x}_T - D_{KL}(q(\mathbf{x}_{1:T}|\mathbf{x}_0)\ ||\ p_\theta(\mathbf{x}_{1:T}|\mathbf{x}_0)) \qquad & \because q\text{를 통해 } p_\theta\text{를 찾으려고 함}\ (\text{Variational Inference}) \\

&\le \int(-\log\ \frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)})\cdot q(\mathbf{x}_T|\mathbf{x}_0)\ d\mathbf{x}_T\ \qquad & \because D_{KL} \ge 0 \\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ \frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}\right] \qquad & \\

&= ELBO \qquad & \\
\end{align}
$$

#### 2.3.3 Optimization Function 구하기

[Section 2.1](#21-reverse-process)과 [section 2.2](#22-forward-process-diffusion-process)를 사용해 위에서 variational inference를 통해 찾은 ELBO term을 아래와 같이 풀어 쓸 수 있습니다.

$$
\begin{align}
\mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p_\theta(\mathbf{x}_0)\right]

&\le \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ \frac{p(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}\right] \qquad & \\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ \frac{p(\mathbf{x}_T)\cdot \prod_{t=1}^T p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{\prod_{t=1}^T q(\mathbf{x}_t|\mathbf{x}_{t-1})}\right] \qquad & \because \text{Section 2.1}\ \&\ \text{Section2.2} \\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\log\prod_{t=1}^T \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_t|\mathbf{x}_{t-1})}\right] \qquad & \because \log\ a\cdot b = \log\ a + \log\ b \\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\left( \log\frac{p_\theta(\mathbf{x}_0|\mathbf{x}_1)}{q(\mathbf{x}_1|\mathbf{x}_0)} + \log\frac{p_\theta(\mathbf{x}_1|\mathbf{x}_2)}{q(\mathbf{x}_2|\mathbf{x}_1)} + \cdots + \log\frac{p_\theta(\mathbf{x}_{T-1}|\mathbf{x}_T)}{q(\mathbf{x}_T|\mathbf{x}_{T-1})} \right)\right] \qquad & \because \log\ a\cdot b = \log\ a + \log\ b \\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\sum_{t=1}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_t|\mathbf{x}_{t-1})}\right] \qquad & \\

&=: L \qquad & \\

\end{align}
$$

지금까지 많은 공식들이 나와 헷갈릴 수 있으니 지금까지 증명했던 공식을 간단히 나타내면 아래와 같이 나타낼 수 있습니다.

$$\mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p_\theta(\mathbf{x}_0)\right] \le \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ \frac{p(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}\right] = \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\sum_{t=1}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_t|\mathbf{x}_{t-1})}\right] =: L$$

### 2.4 Variance schedule $\beta\_t$

...진행 중입니다.
