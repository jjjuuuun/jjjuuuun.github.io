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

![](../../assets/img/Paper_Reading/DDPM/ddpm_8.jpg){: .normal}

🔎 $\beta\_t$는 reparameterization을 통해 학습될 수 있고 hyperparameter로 두고 학습할 수 있습니다.

🔎 $\beta\_t$가 매우 작을 때 diffusion process와 reverse process는 동일한 형태를 가지고 있기 때문에 reverse process의 표현성은 diffusion process의 gaussian conditionals에 의해서 부분적으로 보장됩니다. ($\because$ [이전 연구](https://arxiv.org/abs/1503.03585))

🔎 Diffusion process에서 주목할만 한 점은 diffusion process를 임의의 timestep $t$에서 closed form으로 나타낼 수 있다는 점입니다.

🔎 어떻게 closed form으로 나타낼 수 있는지 아래 증명과 함께 알아보겠습니다.

#### 2.4.1 Reparameterziation Trick

딥러닝에서 필수적인 계산이라 함은 바로 chain rule에 의한 backpropagation입니다.

Backpropagation을 사용하기 위해서는 미분이 필수적인데 어떠한 확률분포를 따르는 확률변수 $Z$가 backpropagation의 대상일 때 미분이 불가능해 나온 것이 바로 reparameterization trick입니다.

정리를 해보면 확률변수 $Z$를 standard normal distribution에서 sampling 하는 과정이 미분 가능하도록 만들어 backpropagation을 가능하게 만드는 것이 reparameterization trick이며 아래와 같이 나타낼 수 있습니다.

$$Z \sim \mathcal{N}(\mu, \sigma^2) \Longrightarrow Z = \mu + \sigma\cdot\epsilon (\epsilon \sim \mathcal{N}(0, \mathrm{I}))$$

#### 2.4.2 Diffusion Proecess with reprameterization trick

먼저 diffusion proecess의 sampling 공식을 다시 한 번 살펴보겠습니다.

$$
\begin{align}

& \text{Diffusion process} : &q(\mathbf{x}_t|\mathbf{x}_{t-1}) := \mathcal{N}(\mathbf{x}_t; \sqrt{1-\beta_t}\mathbf{x}_{t-1}, \beta_t\mathrm{I}) \\ \\

& \text{Sampling} : &\mathbf{x}_t \sim \mathcal{N}(\sqrt{1-\beta_t}\mathbf{x}_{t-1}, \beta_t\mathrm{I})

\end{align}
$$

이제 reparameterization trick을 활용해 timestep $t$에서 diffusion process를 closed form으로 나타내보도록 하겠습니다.

$$
\begin{align}

\mathbf{x}_t

&= \sqrt{1 - \beta_t}\mathbf{x}_{t-1} + \sqrt{\beta_t}\epsilon_{t-1} (\epsilon_{t-1} \sim \mathcal{N}(0, \mathrm{I})) \qquad & \because \text{Reparameterization Trick} \\

&= \sqrt{\alpha_t}{\color{Green}\mathbf{x}_{t-1}} + \sqrt{1-\alpha_t} \epsilon_{t-1} \qquad & \because \alpha_t := 1-\beta_t \\

&= \sqrt{\alpha_t}({\color{Green}\sqrt{\alpha_{t-1}}{\mathbf{x}_{t-2} + \sqrt{1-\alpha_{t-1}} \epsilon_{t-2}}}) + \sqrt{1-\alpha_t} \epsilon_{t-1} \qquad & \\

&= \sqrt{\alpha_{t}\cdot \alpha_{t-1}}\mathbf{x}_{t-2} + {\color{Red}\sqrt{\alpha_t(1-\alpha_{t-1})}\epsilon_{t-2} + \sqrt{1-\alpha_t} \epsilon_{t-1}} \qquad & \\

&= \sqrt{\alpha_{t}\cdot \alpha_{t-1}}\mathbf{x}_{t-2} + {\color{Red}\sqrt{\alpha_t(1-\alpha_{t-1}) + 1 - \alpha_t}\bar{\epsilon}_{t-2}} \qquad & \because \epsilon_1 \sim \mathcal{N}(0, \sigma_1^2\mathrm{I})\quad \& \quad \epsilon_2 \sim \mathcal{N}(0, \sigma_2^2\mathrm{I}) \Longrightarrow \bar{\epsilon} := \epsilon_1 + \epsilon_2 \sim \mathcal{N}(0, (\sigma_1^2 + \sigma_2^2)\mathrm{I}) \\

&= \sqrt{\alpha_{t}\cdot \alpha_{t-1}}\mathbf{x}_{t-2} + {\color{Red}\sqrt{1 - \alpha_t\cdot\alpha_{t-1}}\bar{\epsilon}_{t-2}} \qquad & \\

&= \cdots \qquad & \\

&= \sqrt{\bar{\alpha}_t}\mathbf{x}_0 + \sqrt{1 - \bar{\alpha}_t}\epsilon \qquad & \because \bar{\alpha}_t := \alpha_t\cdot\alpha_{t-1}\cdots\alpha_1 \\

\end{align}
$$

### 2.5 Rewriting $L$

![](../../assets/img/Paper_Reading/DDPM/ddpm_9.jpg){: .normal}

위에서 정의한 [optimization function $L$](#233-optimization-function-구하기)을 변경함으로써 효율적인 학습이 가능합니다.

어떻게 효율적으로 변경하는지 차례차례 살펴보겠습니다.

$$
\begin{align}
L

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\sum_{t=1}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_t|\mathbf{x}_{t-1})}\right] \qquad & \tag{1}\\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\sum_{t=2}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{\color{Orange}q(\mathbf{x}_t|\mathbf{x}_{t-1})} - \log \frac{p_\theta(\mathbf{x}_{0}|\mathbf{x}_1)}{q(\mathbf{x}_1|\mathbf{x}_{0})}\right] \qquad & \because t = 1 \text{분리} \tag{2}\\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\sum_{t=2}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{\color{Orange}q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0})}\cdot {\color{Orange}\frac{q(\mathbf{x}_{t-1}|\mathbf{x}_{0})}{q(\mathbf{x}_t|\mathbf{x}_{0})}} - \log \frac{p_\theta(\mathbf{x}_{0}|\mathbf{x}_1)}{q(\mathbf{x}_1|\mathbf{x}_{0})}\right] \qquad & \because \text{Section 2.5.1} \tag{3}\\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\sum_{t=2}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0})}-{\color{Purple}\sum_{t=2}^T\log \frac{q(\mathbf{x}_{t-1}|\mathbf{x}_{0})}{q(\mathbf{x}_t|\mathbf{x}_{0})}} - \log \frac{p_\theta(\mathbf{x}_{0}|\mathbf{x}_1)}{q(\mathbf{x}_1|\mathbf{x}_{0})}\right] \qquad & \because \log\ a\cdot b = \log\ a + \log\ b \tag{4}\\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\sum_{t=2}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0})}-\right({\color{Purple}\log \frac{q(\mathbf{x}_{1}|\mathbf{x}_{0})}{q(\mathbf{x}_{2}|\mathbf{x}_{0})} + \log \frac{q(\mathbf{x}_{2}|\mathbf{x}_{0})}{q(\mathbf{x}_{3}|\mathbf{x}_{0})} \cdots + \log \frac{q(\mathbf{x}_{T-1}|\mathbf{x}_{0})}{q(\mathbf{x}_{T}|\mathbf{x}_{0})}}\left) - \log \frac{p_\theta(\mathbf{x}_{0}|\mathbf{x}_1)}{q(\mathbf{x}_1|\mathbf{x}_{0})}\right] \qquad & \tag{5}\\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\sum_{t=2}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0})}-{\color{Purple}\log \frac{q(\mathbf{x}_{1}|\mathbf{x}_{0})}{q(\mathbf{x}_{T}|\mathbf{x}_{0})}} - \log \frac{p_\theta(\mathbf{x}_{0}|\mathbf{x}_1)}{q(\mathbf{x}_1|\mathbf{x}_{0})}\right] \qquad & \because \log \frac{a}{b}+\log \frac{b}{c} = \log \frac{a}{c} \tag{6}\\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\sum_{t=2}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0})}-\log \frac{\color{Red}q(\mathbf{x}_{1}|\mathbf{x}_{0})}{q(\mathbf{x}_{T}|\mathbf{x}_{0})} - \log \frac{p_\theta(\mathbf{x}_{0}|\mathbf{x}_1)}{\color{Red}q(\mathbf{x}_1|\mathbf{x}_{0})}\right] \qquad & \tag{7}\\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log\ p(\mathbf{x}_T) -\sum_{t=2}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0})} - \log \frac{p_\theta(\mathbf{x}_{0}|\mathbf{x}_1)}{\color{Blue}q(\mathbf{x}_{T}|\mathbf{x}_{0})}\right] \qquad & \because \log \frac{a}{b}+\log \frac{b}{c} = \log \frac{a}{c} \tag{8}\\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log \frac{p(\mathbf{x}_T)}{\color{Blue}q(\mathbf{x}_{T}|\mathbf{x}_{0})} -\sum_{t=2}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0})} - \log\ p_\theta(\mathbf{x}_{0}|\mathbf{x}_1)\right] \qquad & \tag{9}\\

&= {\color{Red}\mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log \frac{p(\mathbf{x}_T)}{q(\mathbf{x}_{T}|\mathbf{x}_{0})}\right]} + \sum_{t=2}^T{\color{Green}\mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0})}\right]} + {\color{Blue}\mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[- \log\ p_\theta(\mathbf{x}_{0}|\mathbf{x}_1)\right]} \qquad & \tag{10}\\

&= {\color{Red}D_{KL}(q(\mathbf{x}_T|\mathbf{x}_0)||p(\mathbf{x}_T))} + \sum_{t=2}^T {\color{Green}D_{KL}(q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0}) || p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t))} + {\color{Blue}\mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[- \log\ p_\theta(\mathbf{x}_{0}|\mathbf{x}_1)\right]} \qquad & \because D_{KL}(Q|P) = \mathbb{E}_{\mathbf{x}\ \sim\ Q(\mathbf{x})}\left[-\log \frac{P(\mathbf{x})}{Q(\mathbf{x})}\right]\tag{11}\\

&= {\color{Red}L_T} + \sum_{t=2}^T {\color{Green}L_{t-1}} + {\color{Blue}L_0} \qquad & \tag{12}\\

&= {\color{Red}L_T} + {\color{Green}L_{1:T-1}} + {\color{Blue}L_0} \qquad & \tag{13}\\

\end{align}
$$

#### 2.5.1 Formula (2), (3) : Rewriting $q(\mathbf{x}\_t|\mathbf{x}\_{t-1})$

$q(\mathbf{x}\_{t}\|\mathbf{x}\_{t-1})$에 condition으로 다루기 쉬운 $\mathbf{x}\_{0}$을 추가함으로써 KL divergence를 통해 직접적으로 $p\_{\theta}(\mathbf{x}\_{t-1}\|\mathbf{x}\_{t})$를 비교할 수 있게 됩니다.

Optimization function $L$을 간단히 하기 위해 $q(\mathbf{x}\_{t}\|\mathbf{x}\_{t-1})$을 간단히 하면 아래와 같습니다.

$$
\begin{align}

{\color{Orange}q(\mathbf{x}_t|\mathbf{x}_{t-1})}

&= q(\mathbf{x}_t|\mathbf{x}_{t-1}, \mathbf{x}_{0}) \qquad & \\

&= \frac{q(\mathbf{x}_t, \mathbf{x}_{t-1}, \mathbf{x}_{0})}{q(\mathbf{x}_{t-1}, \mathbf{x}_{0})} \qquad & \\

&= \frac{q(\mathbf{x}_t, \mathbf{x}_{t-1}, \mathbf{x}_{0})}{\color{Red}q(\mathbf{x}_{t-1}, \mathbf{x}_{0})}\cdot \frac{q(\mathbf{x}_t, \mathbf{x}_{0})}{\color{Purple}q(\mathbf{x}_t, \mathbf{x}_{0})} \qquad & \\

&= \frac{q(\mathbf{x}_t, \mathbf{x}_{t-1}, \mathbf{x}_{0})}{\color{Purple}q(\mathbf{x}_t, \mathbf{x}_{0})}\cdot \frac{\color{Blue}q(\mathbf{x}_t, \mathbf{x}_{0})}{\color{Red}q(\mathbf{x}_{t-1}, \mathbf{x}_{0})}  \qquad & \\

&= \frac{q(\mathbf{x}_t, \mathbf{x}_{t-1}, \mathbf{x}_{0})}{q(\mathbf{x}_t, \mathbf{x}_{0})}\cdot \frac{\color{Blue}q(\mathbf{x}_t|\mathbf{x}_{0})\cdot q(\mathbf{x}_{0})}{\color{Red}q(\mathbf{x}_{t-1}|\mathbf{x}_{0})\cdot q(\mathbf{x}_{0})} \qquad & \\

&= {\color{Green}\frac{q(\mathbf{x}_t, \mathbf{x}_{t-1}, \mathbf{x}_{0})}{q(\mathbf{x}_t, \mathbf{x}_{0})}}\cdot \frac{\color{Blue}q(\mathbf{x}_t|\mathbf{x}_{0})}{\color{Red}q(\mathbf{x}_{t-1}|\mathbf{x}_{0})} \qquad & \\

&= {\color{Green}q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0})}\cdot \frac{q(\mathbf{x}_t|\mathbf{x}_{0})}{q(\mathbf{x}_{t-1}|\mathbf{x}_{0})} \qquad & \\

\end{align}
$$

### 2.6 What probability distribution $q(\mathbf{x}\_{t}\|\mathbf{x}\_{t-1}, \mathbf{x}\_{0})$ follows

![](../../assets/img/Paper_Reading/DDPM/ddpm_10.jpg){: .normal}

$$
\begin{align}

q(\mathbf{x}_{t-1}|\mathbf{x}_{t}, \mathbf{x}_{0})

&= {\color{Red}q(\mathbf{x}_t|\mathbf{x}_{t-1})}\cdot\frac{\color{Blue}q(\mathbf{x}_{t-1}|\mathbf{x}_{0})}{\color{Green}q(\mathbf{x}_{t}|\mathbf{x}_{0})} \qquad & \because \text{Section 2.5.1}\\

&= {\color{Red}\frac{1}{\sqrt{2\pi\beta_{t}}}\exp\left(-\frac{(\mathbf{x}_{t}-\sqrt{1-\beta_{t}}\mathbf{x}_{t-1})^2}{2\beta_{t}}\right)} \cdot {\color{Blue}\frac{1}{\sqrt{2\pi(1-\bar{\alpha}_{t-1})}}\exp\left(-\frac{(\mathbf{x}_{t-1}-\sqrt{\bar{\alpha}_{t-1}}\mathbf{x}_{0})^2}{2(1-\bar{\alpha}_{t-1})}\right)} \cdot {\color{Green}\sqrt{2\pi(1-\bar{\alpha}_{t})}\exp\left(\frac{(\mathbf{x}_{t}-\sqrt{\bar{\alpha}_{t}}\mathbf{x}_{0})^2}{2(1-\bar{\alpha}_{t})}\right)}\qquad & \because x \sim \mathcal{N}(\mu, \sigma^2) \Longrightarrow f(x) = \frac{1}{\sqrt{2\pi\sigma^2}}\exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right) \\

&\propto \exp\left(-\frac{(\mathbf{x}_{t}-\sqrt{1-\beta_{t}}\mathbf{x}_{t-1})^2}{2\beta_{t}}\right) \cdot \exp\left(-\frac{(\mathbf{x}_{t-1}-\sqrt{\bar{\alpha}_{t-1}}\mathbf{x}_{0})^2}{2(1-\bar{\alpha}_{t-1})}\right) \cdot \exp\left(\frac{(\mathbf{x}_{t}-\sqrt{\bar{\alpha}_{t}}\mathbf{x}_{0})^2}{2(1-\bar{\alpha}_{t})}\right) \qquad & \because \text{상수 제거} \\

&= \exp\left(-\frac{1}{2}\frac{(\mathbf{x}_{t}-\sqrt{1-\beta_{t}}{\color{Orange}\mathbf{x}_{t-1}})^2}{\beta_{t}} + \frac{({\color{Orange}\mathbf{x}_{t-1}}-\sqrt{\bar{\alpha}_{t-1}}\mathbf{x}_{0})^2}{(1-\bar{\alpha}_{t-1})} - \frac{(\mathbf{x}_{t}-\sqrt{\bar{\alpha}_{t}}\mathbf{x}_{0})^2}{(1-\bar{\alpha}_{t})}\right) \qquad & \because \exp\ a \cdot \exp\ b = \exp (a+b) \\

&= \exp\bigg(-\frac{1}{2}\Big({\color{Red}(\frac{\alpha_{t}}{\beta_{t}} + \frac{1}{1-\bar{\alpha}_{t-1}})}{\color{Orange}\mathbf{x}_{t-1}^2} - {\color{Blue} (\frac{2\sqrt{\alpha_{t}}}{\beta_{t}}\mathbf{x}_{t} + \frac{2\sqrt{\bar{\alpha}_{t-1}}}{1-\bar{\alpha}_{t-1}}\mathbf{x}_{0})}{\color{Orange}\mathbf{x}_{t-1}} + C(\mathbf{x}_{t}, \mathbf{x}_{0})\Big) \bigg)  \qquad & \because \text{Condition으로 }\mathbf{x}_{t}\text{와 }\mathbf{x}_{0}\text{가 주어졌을 때 }\mathbf{x}_{t-1}\text{의 확률분포를 찾고자 함} \\

&= \exp\bigg(-\frac{1}{2}\Big({\color{Red}a}\mathbf{x}_{t-1}^2 - {\color{Blue}b}\mathbf{x}_{t-1} + C(\mathbf{x}_{t}, \mathbf{x}_{0})\Big)\bigg) \qquad & \\

&= \exp\bigg(-\frac{\color{Red}a}{2}\Big(\mathbf{x}_{t-1} - \frac{\color{Blue}b}{\color{Red}a}\Big)^2 \bigg) + C^\prime(\mathbf{x}_{t}, \mathbf{x}_{0}) \qquad & \\

&\propto \frac{1}{\sqrt{2\pi\frac{1}{\color{Red}a}}}\exp\left(-\frac{(\mathbf{x}_{t-1}-\frac{\color{Blue}b}{\color{Red}a})^2}{2\frac{1}{\color{Red}a}}\right) + C^{''}(\mathbf{x}_{t}, \mathbf{x}_{0}) \qquad & \\

\end{align}
$$

위의 식 $q(\mathbf{x}\_{t-1}\|\mathbf{x}\_{t}, \mathbf{x}\_{0})$을 정리해보면 평균이 ${\color{Blue}b}/{\color{Red}a}$이고 분산이 $1/{\color{Red}a}$인 gaussian distribution임을 알 수 있습니다. 논문에 있는 식으로 정리를 해보면 아래와 같이 정리할 수 있습니다.

$$
\begin{align}

&q(\mathbf{x}_{t-1}|\mathbf{x}_{t}, \mathbf{x}_{0}) = \mathcal{N}(\mathbf{x}_{t-1};\tilde{\mu}_t(\mathbf{x}_t, \mathbf{x}_0), \tilde{\beta}_t\mathrm{I}) \\

&\text{where}\quad \tilde{\beta} := \frac{1}{\color{Red}a} = \frac{1 - \bar{\alpha}_{t-1}}{1-\bar{\alpha}_t}\cdot\beta_t\quad \text{and}\quad

\tilde{\mu}_t(\mathbf{x}_{t}, \mathbf{x}_{0}) := {\color{Blue}b}/{\color{Red}a} = \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1-\bar{\alpha}_t}\mathbf{x}_0 + \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}\mathbf{x}_t

\end{align}
$$

결과적으로 optimization function $L$의 모든 $D_{KL}$은 gaussian distribution 비교이므로 high variance Monte Carlo estimates 대신 closed form을 사용하여 Rao-Blackwellized 방식으로 계산할 수 있습니다.

## 3. Diffusion models and denoising autoencoders

![](../../assets/img/Paper_Reading/DDPM/ddpm_11.jpg){: width="400" .left}

🔎 Diffusion models은 제약이 있는 latent variable models의 한 종류로 보일 수 있지만 구현에서 <u>많은 자유도</u>를 허용합니다.($\beta\_t$, model architecture, Gaussian distributionparameterization)

🔎 해당 논문의 저자들이 선택한 것들을 앞으로 설명합니다.

1️⃣ Section 3.2 : New connection between diffusion models and decnoising score matching

2️⃣ Section 3.4 : Weighted variational bound objective for diffusion models

3️⃣ Section 4 : Model design & results

![](../../assets/img/Paper_Reading/blank.png){: .normal}

### 3.1 Forward process and $L\_T$

![](../../assets/img/Paper_Reading/DDPM/ddpm_12.jpg){: .normal}

🔎 [Section 2.5](#25-rewriting)에서의 $L\_T$를 살펴보면 아래와 같습니다.

$$
\begin{align}

L_T

&= D_{KL}(q(\mathbf{x}_T|\mathbf{x}_0)||p(\mathbf{x}_T)) \qquad & \\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log \frac{p(\mathbf{x}_T)}{q(\mathbf{x}_{T}|\mathbf{x}_{0})}\right] \qquad & \\

&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log \frac{\epsilon}{\sqrt{\bar{\alpha}_T}\mathbf{x}_0 + \sqrt{1-\bar{\alpha}_T}\epsilon}\right] \qquad & \because p(\mathbf{x}_T) = \mathcal{N}(\mathbf{x}_T; \mathrm{0}, \mathrm{I})\quad \& \quad q(\mathbf{x}_{T}|\mathbf{x}_{0}) = \mathcal{N}(\mathbf{x}_T; \sqrt{\bar{\alpha}_T}\mathbf{x}_0, (1-\bar{\alpha}_T)\mathrm{I})\\

\end{align}
$$

🔎 $\beta\_t$가 학습가능한 파라미터임을 무시하고 상수로 사용합니다. ([Section 4]())

🔎 따라서 posterior $q$는 학습 파리미터를 가지고 있지 않습니다.

⭐ 그렇기 때문에 $L\_T$는 학습하지 않고 무시합니다.

### 3.2 Reverse process and $L\_{1:T-1}$

![](../../assets/img/Paper_Reading/DDPM/ddpm_13.jpg){: .normal}

🔎 [Section 2.5](#25-rewriting)에서의 $L\_{1:T-1}$를 살펴보면 아래와 같습니다.

$$
\begin{align}

L_{1:T-1}

&= \sum_{t=2}^T L_{t-1} \qquad & \\

&= \sum_{t=2}^TD_{KL}(q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0}) || p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)) \qquad & \\

&= \sum_{t=2}^T\mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[-\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0})}\right] \qquad & \\

\end{align}
$$

먼저 $p\_\theta(\mathbf{x}\_{t-1}\|\mathbf{x}\_t) = \mathcal{N}(\mathbf{x}\_{t-1}; \mu\_\theta(\mathbf{x}\_t, t), \Sigma\_\theta(\mathbf{x}\_t, t))$ for $1 < t \le T$에 대해서 논해보고자 합니다.

1. $\Sigma\_\theta(\mathbf{x}\_t, t)$

   > 해당 논문의 저자들은 먼저 $\Sigma\_\theta(\mathbf{x}\_t, t)$를 학습하지 않아도 되는 time dependent constants인 $\sigma\_t^2\mathrm{I}$로 설정했습니다.  
   > 이때 실험적으로 `1`번 : $\sigma\_t^2 = \beta\_t$과 `2`번 : $\sigma\_t^2 = \tilde{\beta}\_t = \frac{1 - \bar{\alpha}\_{t-1}}{1-\bar{\alpha}\_t}\cdot\beta\_t \cdots 2$의 결과가 비슷함을 발견했습니다.
   > `1`번과 `2`번은 reverse process에 대한 upper bound와 lower bound에 해당합니다.([이전 연구](https://arxiv.org/abs/1503.03585))

2. $\mu\_\theta(\mathbf{x}\_t, t)$

   > 알고 있는 정보들을 정리해보면 다음과 같습니다.
   >
   > > $p\_\theta(\mathbf{x}\_{t-1}\|\mathbf{x}\_t) = \mathcal{N}(\mathbf{x}\_{t-1}; \mu\_\theta(\mathbf{x}\_t, t), \sigma\_t^2\mathrm{I}) = \mathcal{N}(\mathbf{x}\_{t-1}; \mu\_\theta(\mathbf{x}\_t, t), \tilde{\beta}\_t\mathrm{I})$  
   > > $q(\mathbf{x}\_{t-1}\|\mathbf{x}\_{t}, \mathbf{x}\_{0}) = \mathcal{N}(\mathbf{x}\_{t-1};\tilde{\mu}\_t(\mathbf{x}\_t, \mathbf{x}\_0), \tilde{\beta}\_t\mathrm{I})$
   >
   > 분산이 같은 두 가우시안 분포의 KL-Divergence를 구하면 아래와 같이 정리 가능합니다.
   >
   > > $L\_{t-1} = \mathbb{E}\_{\mathbf{x}\_T\ \sim\ q(\mathbf{x}\_T\|\mathbf{x}\_0)}\left[\frac{1}{2\sigma\_t^2}\lVert\tilde{\mu}\_t(\mathbf{x}\_t, \mathbf{x}\_0) - \mu\_\theta(\mathbf{x}\_t, t)\rVert^2\right] + C$  
   > > $\qquad \text{where C is a constant that does not depend on }\theta$
   >
   > 즉, $\mu\_\theta$의 가장 간단한 parameterization은 forward process posteriro mean인 $\tilde{\mu}\_\theta$를 예측하는 모델임을 알 수 있습니다.
   >
   > $L\_{t-1}$을 reparameterization trick을 사용해 변경하면 아래와 같습니다.
   >
   > > $$\begin{align}L_{t-1}&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\lVert\tilde{\mu}_t(\mathbf{x}_t, \mathbf{x}_0) - \mu_\theta(\mathbf{x}_t, t)\rVert^2\right] + C \qquad & \\&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1-\bar{\alpha}_t}\mathbf{x}_0 + \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}\mathbf{x}_t - \mu_\theta(\mathbf{x}_t, t)\Bigg\rVert^2\right] + C \qquad & \because \tilde{\mu}_t(\mathbf{x}_t, \mathbf{x}_0) = \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1-\bar{\alpha}_t}\mathbf{x}_0 + \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}\mathbf{x}_t \\&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\frac{ {\color{Red}\sqrt{\bar{\alpha}_{t-1}}} \beta_t}{1-\bar{\alpha}_t}\cdot\frac{\mathbf{x}_t - \sqrt{1 - \bar{\alpha}_t}\epsilon}{\color{Red}\sqrt{\bar{\alpha}_t}} + \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}\mathbf{x}_t - \mu_\theta(\mathbf{x}_t, t)\Bigg\rVert^2\right] + C \qquad & \because \mathbf{x}_t = \sqrt{\bar{\alpha}_t}\mathbf{x}_0 + \sqrt{1 - \bar{\alpha}_t}\epsilon \Longrightarrow \mathbf{x}_0 = \frac{\mathbf{x}_t - \sqrt{1 - \bar{\alpha}_t}\epsilon}{\sqrt{\bar{\alpha}_t}}  \\&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\frac{\beta_t}{1-\bar{\alpha}_t}\cdot\frac{ {\color{Blue}\mathbf{x}_t} - \sqrt{1 - \bar{\alpha}_t}\epsilon}{\color{Red}\sqrt{\alpha_t}} + \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}{\color{Blue}\mathbf{x}_t} - \mu_\theta(\mathbf{x}_t, t)\Bigg\rVert^2\right] + C \qquad & \\&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\left(\frac{\beta_t}{(1-\bar{\alpha}_t) \cdot \sqrt{\alpha_t}} + \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}\right){\color{Blue}\mathbf{x}_t} - \frac{\beta_t\cdot {\color{Green}\sqrt{1 - \bar{\alpha}_t}}\epsilon}{ {\color{Green}(1-\bar{\alpha}_t)} \cdot \sqrt{\alpha_t}} - \mu_\theta(\mathbf{x}_t, t)\Bigg\rVert^2\right] + C \qquad & \\&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\left(\frac{\beta_t}{(1-\bar{\alpha}_t) \cdot {\color{Orange}\sqrt{\alpha_t}}} + \frac{ {\color{Orange}\sqrt{\alpha_t}}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}\right)\mathbf{x}_t - \frac{\beta_t\cdot\epsilon}{ {\color{Green}\sqrt{1 - \bar{\alpha}_t}} \cdot {\color{Orange}\sqrt{\alpha_t}}} - \mu_\theta(\mathbf{x}_t, t)\Bigg\rVert^2\right] + C \qquad & \\&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\frac{1}{ {\color{Orange}\sqrt{\alpha_t}}}\Bigg(\bigg(\frac{ {\color{Purple}\beta_t}}{1-\bar{\alpha}_t} + \frac{ {\color{Orange}\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}\bigg)\mathbf{x}_t - \frac{\beta_t\cdot\epsilon}{ \sqrt{1 - \bar{\alpha}_t}}\Bigg) - \mu_\theta(\mathbf{x}_t, t)\Bigg\rVert^2\right] + C \qquad & \\&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\frac{1}{ \sqrt{\alpha_t}}\Bigg(\bigg(\frac{ {\color{Purple}1-\alpha_t}}{1-\bar{\alpha}_t} + \frac{ {\color{Brown}\alpha_t(1-\bar{\alpha}_{t-1})}}{1-\bar{\alpha}_t}\bigg)\mathbf{x}_t - \frac{\beta_t\cdot\epsilon}{ \sqrt{1 - \bar{\alpha}_t}}\Bigg) - \mu_\theta(\mathbf{x}_t, t)\Bigg\rVert^2\right] + C \qquad & \\&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\frac{1}{ \sqrt{\alpha_t}}\Bigg(\bigg(\frac{ 1-\alpha_t}{1-\bar{\alpha}_t} + \frac{ {\color{Brown}\alpha_t-\bar{\alpha}_{t}}}{1-\bar{\alpha}_t}\bigg)\mathbf{x}_t - \frac{\beta_t\cdot\epsilon}{ \sqrt{1 - \bar{\alpha}_t}}\Bigg) - \mu_\theta(\mathbf{x}_t, t)\Bigg\rVert^2\right] + C \qquad & \\&= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\frac{1}{ \sqrt{\alpha_t}}\Bigg(\mathbf{x}_t - \frac{\beta_t}{ \sqrt{1 - \bar{\alpha}_t}}\cdot\epsilon\Bigg) - \mu_\theta(\mathbf{x}_t, t)\Bigg\rVert^2\right] + C \qquad & \\ &\propto \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\frac{1}{ \sqrt{\alpha_t}}\Bigg(\mathbf{x}_t - \frac{\beta_t}{ \sqrt{1 - \bar{\alpha}_t}}\cdot\epsilon\Bigg) - \mu_\theta(\mathbf{x}_t, t)\Bigg\rVert^2\right] \qquad & \\\end{align}$$
   >
   > 변경된 $L\_{t-1}$에서 $\beta\_t$가 상수이고 $\mathbf{x}\_t$가 주어진 값이기 때문에 $\epsilon$을 예측하는 function approximator인 $\epsilon\_\theta$를 통해 최소화 할 수 있습니다. 즉 아래와 같이 $L\_{t-1}$을 다시 표현할 수 있습니다.
   >
   > > $$\begin{align}L_{t-1} &\propto \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\frac{1}{ \sqrt{\alpha_t}}\Bigg(\mathbf{x}_t - \frac{\beta_t}{ \sqrt{1 - \bar{\alpha}_t}}\cdot\epsilon\Bigg) - \mu_\theta(\mathbf{x}_t, t)\Bigg\rVert^2\right] \qquad & \\ &= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{1}{2\sigma_t^2}\Bigg\lVert\frac{1}{ \sqrt{\alpha_t}}\Bigg(\mathbf{x}_t - \frac{\beta_t}{ \sqrt{1 - \bar{\alpha}_t}}\cdot\epsilon\Bigg) - \frac{1}{ \sqrt{\alpha_t}}\Bigg(\mathbf{x}_t - \frac{\beta_t}{ \sqrt{1 - \bar{\alpha}_t}}\cdot\epsilon_\theta\Bigg)\Bigg\rVert^2\right] \qquad & \\ &= \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[\frac{\beta_t^2}{2\sigma_t^2\cdot\alpha_t\cdot(1-\bar{\alpha}_t)}\lVert\epsilon - \epsilon_\theta\rVert^2\right] \qquad & \\ \end{align}$$

진행중입니다....
