---
layout: post
title: "[Generative Model] DDIM : Denoising Diffusion Implicit Models"
author: kjy
date: 2024-02-28 22:30:00 +09:00
categories: [Deep Learning, Paper Reading]
tags: [Deep Learning, Paper Reading, Computer Vision, Generative Model]
comments: true
toc: true
math: true
---

해당 포스트에서는 [DDIM : Denoising Diffusion Implicit Models](https://arxiv.org/abs/2010.02502) 논문을 함께 읽어가도록 하겠습니다. 문장마다 해석을 하기 보다는 각 문단에서 중요한 부분을 요약하는 식으로 진행하겠습니다.

Emoji의 의미는 아래와 같습니다.

🔎 : 논문에 있는 내용

💡 : 논문에 적혀있지 않은 사전지식

💭 : 저의 생각

⭐ : 중요한 내용

✅ : 나중에 확인이 필요한 내용

## Abstract

![](../../assets/img/Paper_Reading/DDIM/ddim_1.jpg){: .normal}

🔎 DDPM의 단점 : Sample을 생성하기위해 많은 step의 Markov chain 과정을 거쳐야 합니다.

🔎 DDIM : DDPM과 같은 objective function을 만드는 non-Markovian diffusion process를 통해 학습

> 🔎 10 ~ 50배 정도 DDPM보다 빠릅니다.  
> 🔎 Semantically meaningful image를 latent space에서 바로 interpolation을 통해 만들어낼 수 있습니다.  
> 🔎 매우 낮은 reconstruction error를 관찰했습니다.

## 1. Introduction

![](../../assets/img/Paper_Reading/DDIM/ddim_2.jpg){: .normal}

🔎 Generative model : GANs, VAE, Autoregressive models, normalizaing flows

🔎 그 중 GANs이 가장 좋은 품질을 만들어냈지만 학습 안정화를 하기 위해 매우 신중한 optimization과 architecture 선택이 필요합니다.

🔎 최근에는 iterative generative models에 대한 연구가 진행(DDPM, NCSN)

🔎 Iterative generative model은 GANs과 견줄만한 품질을 생성했으며 GANs에 비해 학습이 쉽다는 장점이 있습니다.

🔎 그러나 높은 품질의 sample을 생성하기 위해서는 많은 iteration이 필요해 GANs보다 매우 느리다는 단점이 있습니다.

![](../../assets/img/Paper_Reading/DDIM/ddim_3.jpg){: .normal}

🔎 DDPM과 GANs의 gap을 줄이기위해 DDIM을 제시합니다.

🔎 DDIM은 DDPM의 objective function과 같은 objective function을 통해 학습을 진행합니다. ([Section 3](#3-variational-inference-for-non-markovian-forward-process))

🔎 Diffusion process에서 DDPM과 달리 non-Markovian을 사용합니다. ([Section 4.1]())

🔎 Reverse process는 그대로 Markov chains을 사용하는데 짧은 Markov chains 사용합니다. ([Section 4.2]())

🔎 DDPM보다 나은 결과 ([Section 5]())

> 1. 더 나은 품질의 sample을 더 빠른 속도로 생성
> 2. DDIM으로 생성한 sample은 consistency를 가지고 있어 똑같은 initial latent variable과 다양한 길이의 Markov chain을 통해 생성된 sample은 비슷한 feature를 가지고 있습니다.
> 3. Consistency 때문에 initial latent variable을 조작해서 의미있는 이미지를 만들어낼 수 있습니다.

## 2. Background

![](../../assets/img/Paper_Reading/DDIM/ddim_4.jpg){: .normal}

💡 해당 파트는 DDPM에 있는 내용을 복습하는 부분이므로 넘어가도록 하겠습니다.

> - [Paper Reading DDPM](https://jjjuuuun.github.io/posts/Paper-Reading-DDPM/)
> - [Paper Review DDPM](https://jjjuuuun.github.io/posts/Paper-Review-DDPM/)

💡 다만, notation이 다른 부분이 있어 그 부분만 말씀드리고 넘어가겠습니다. (DDIM $\Longrightarrow$ DDPM)

> - $\alpha\_t$ $\longrightarrow$ $\bar{\alpha}\_t$
> - ELBO term을 최대화하는 $\theta$를 찾는 문제 $\longrightarrow$ Negative ELBO term을 최소화하는 $\theta$를 찾는 문제
> - $\gamma\_t$ : 시간에 따른 가중치(계수) $\longrightarrow$ $\gamma\_t = 1$

## 3. Variational Inference For Non-Markovian Forward Process

![](../../assets/img/Paper_Reading/DDIM/ddim_5.jpg){: .normal}

🔎 Generative model은 reverse of inference process을 approximate

- Inference process : Diffusion process / Forward process
- Reverse of inference process : Denoising process / Reverse process

🔎 Generative model이 image를 생성하기 위해 필요한 iteration 수를 줄이기 위해 inference process를 다시 생각할 필요가 있습니다.

🔎 DDPM의 objective인 $L\_\gamma$는 marginals $q(\mathbf{x}\_t\|\mathbf{x}\_0)$에만 의존하고 joints $q(\mathbf{x}\_{1:T}\|\mathbf{x}\_0)$에는 직접 의존하지 않습니다.

$$
\begin{align}
L_\gamma(\epsilon_\theta)
:=& \sum^T_{t=1}\gamma_t\mathbb{E}_{\mathbf{x}_0\sim q(\mathbf{x}_0), \epsilon_T \sim \mathcal{N}(\mathrm{0}, \mathrm{I})}\Big[ \lVert \epsilon^{(t)}_\theta {\color{Red}\mathbf{x}_t}\ -\ \epsilon_t \rVert^2_2 \Big] & \qquad \\
=& \sum^T_{t=1}\gamma_t\mathbb{E}_{\mathbf{x}_0\sim q(\mathbf{x}_0), \epsilon_T \sim \mathcal{N}(\mathrm{0}, \mathrm{I})}\Big[ \lVert \epsilon^{(t)}_\theta (\sqrt{\alpha_t}{\color{Red}\mathbf{x}_0}\ +\ \sqrt{1-\alpha_t}\epsilon_t)\ -\ \epsilon_t \rVert^2_2 \Big] & \qquad \because \mathbf{x}_t = \sqrt{\alpha_t}\mathbf{x}_0\ +\ \sqrt{1-\alpha_t}\epsilon_t
\end{align}
$$

🔎 같은 marginals $q(\mathbf{x}\_t\|\mathbf{x}\_0)$을 가지는 joints $q(\mathbf{x}\_{1:T}\|\mathbf{x}\_0)$가 많기 때문에 Non-Markovian inference process를 탐색하여 새로운 generative process로 이어지도록 합니다.

> 💭 $\mathbf{x}\_t$를 만들 수 있는 $q(\mathbf{x}\_{1:T}\|\mathbf{x}\_0)$는 많은데 그 중 Non-Markovian을 사용해서 특정 $\mathbf{x}\_t$를 만들도록 해 새로운 generative process라 정의한다는 이야기인 것 같습니다.

🔎 새로운 generative process 또한 DDPM과 동일한 objective function을 사용할 수 있습니다.

🔎 또한 새로운 generative process를 만드는 가장 중요한 개념인 Non-Markovian이 Gaussian case를 넘어서도 적용됨을 보여주었습니다.

### 3.1 Non-Markovian Forward Processes

![](../../assets/img/Paper_Reading/DDIM/ddim_6.jpg){: .normal}

🔎 우선, inference process를 Non-Markovian으로 정의해보고자 합니다.

$$
\begin{align}
& q_\sigma(\mathbf{x}_{1:T}|\mathbf{x}_0) := q_\sigma(\mathbf{x}_T|\mathbf{x}_0)\prod^T_{t=2}q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) \\

& \qquad \text{where} \\

& \qquad \qquad q_\sigma(\mathbf{x}_T|\mathbf{x}_0) = \mathcal{N}(\sqrt{\alpha_T}\mathbf{x}_0, (1-\alpha_T)\mathrm{I}) \\

& \qquad \qquad q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) = \mathcal{N}\big(\sqrt{\alpha_{t-1}}\mathbf{x}_0 + \sqrt{1-\alpha_{t-1} - \sigma^2_t}\cdot \frac{\mathbf{x}_t - \sqrt{\alpha_t}\mathbf{x}_0}{\sqrt{1-\alpha_t}}, \sigma^2_t\mathrm{I}\big) \quad (t > 1)

\end{align}
$$

🔎 위에서 정의한 inference process가 모든 $t$에서 성립함을 귀납법을 통해 증명해야 합니다.

1. $q\_\sigma(\mathbf{x}\_T\|\mathbf{x}\_0) = \mathcal{N}(\sqrt{\alpha\_T}\mathbf{x}\_0, (1-\alpha\_T)\mathrm{I})$으로부터 $q\_\sigma(\mathbf{x}\_t\|\mathbf{x}\_0)$와 $q\_\sigma(\mathbf{x}\_{t-1}\|\mathbf{x}\_0)$을 정의

   $$
   \begin{align}
       q_\sigma(\mathbf{x}_t|\mathbf{x}_0)\ =&\ \mathcal{N}(\sqrt{\alpha_t}\mathbf{x}_0, (1-\alpha_t)\mathrm{I}) \\
       q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_0)\ =&\ \mathcal{N}(\sqrt{\alpha_{t-1}}\mathbf{x}_0, (1-\alpha_{t-1})\mathrm{I})
   \end{align}
   $$

2. $t$로부터 $t-1$를 유도할 수 있으면 모든 $t$에서 성립함을 증명

   - Bishop 2.115번 공식을 먼저 알아봐야 합니다.

     - $p(\mathbf{x})$가 Gaussian이고 $p(\mathbf{y}\|\mathbf{x})$ 또한 Gaussian distribution일 때 $p(\mathbf{y})$는 Gaussian

   - Bishop 2.115번 공식과 위에서 정의한 내용을 연결 시켜보도록 하겠습니다.

     $$
     \begin{align}
         &p(\mathbf{x}) &\iff& \qquad q_{\sigma}(\mathbf{x}_t|\mathbf{x}_0)& \\
         &p(\mathbf{y}|\mathbf{x}) &\iff& \qquad q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) \\
         &p(\mathbf{y}) &\iff& \qquad q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_0)& \\
     \end{align}
     $$

   - 위에서 $q\_{\sigma}(\mathbf{x}\_t\|\mathbf{x}\_0)$와 $q\_\sigma(\mathbf{x}\_{t-1}\|\mathbf{x}\_t, \mathbf{x}\_0)$를 Gaussian으로 정의했기 때문에 Bishop 2.115번 공식에 따라서 $q\_\sigma(\mathbf{x}\_{t-1}\|\mathbf{x}\_0)$는 Gaussian 이므로 아래와 같이 표기가 가능합니다.

     $$
     \begin{align}
         & q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_0) = \mathcal{N}(\mu_{t-1}, \Sigma_{t-1}) \\ \\

           & \qquad \text{where} \\

           & \qquad \qquad q_\sigma(\mathbf{x}_t|\mathbf{x}_0)\ =\ \mathcal{N}(\sqrt{\alpha_t}\mathbf{x}_0, (1-\alpha_t)\mathrm{I}) \\

           & \qquad \qquad q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) = \mathcal{N}\big(\sqrt{\alpha_{t-1}}\mathbf{x}_0 + \sqrt{1-\alpha_{t-1} - \sigma^2_t}\cdot \frac{\mathbf{x}_t - \sqrt{\alpha_t}\mathbf{x}_0}{\sqrt{1-\alpha_t}}, \sigma^2_t\mathrm{I}\big) \quad (t > 1)
     \end{align}
     $$

   - $q\_\sigma(\mathbf{x}\_{t-1}\|\mathbf{x}\_t, \mathbf{x}\_0)$에 $\mathbf{x}\_t$를 대입해서 $q\_\sigma(\mathbf{x}\_{t-1}\|\mathbf{x}\_0)$의 평균과 분산을 구합니다.

     - 위에서 정의한 $q\_\sigma(\mathbf{x}\_t\|\mathbf{x}\_0)\ =\ \mathcal{N}(\sqrt{\alpha\_t}\mathbf{x}\_0, (1-\alpha\_t)\mathrm{I})$로 부터 reparameterization trick을 통해 $\mathbf{x}\_t$를 구합니다.

     $$
     \begin{align}
         \mathbf{x}_t = \sqrt{\alpha_t}\mathbf{x}_0\ +\ \sqrt{1-\alpha_t}\epsilon_t
     \end{align}
     $$

     - $q\_\sigma(\mathbf{x}\_{t-1}\|\mathbf{x}\_t, \mathbf{x}\_0)$에 $\mathbf{x}\_t$ 대입하여 평균 파트와 분산 파트를 분리하여 계산하면 아래와 같습니다.

     $$
     \begin{align}
         \mu_{t-1}
         &= \sqrt{\alpha_{t-1}}\mathbf{x}_0 + \sqrt{1-\alpha_{t-1} - \sigma^2_t}\cdot \frac{\mathbf{x}_t - \sqrt{\alpha_t}\mathbf{x}_0}{\sqrt{1-\alpha_t}} \\
         &= \sqrt{\alpha_{t-1}}\mathbf{x}_0 + \sqrt{1-\alpha_{t-1} - \sigma^2_t}\cdot \frac{\sqrt{\alpha_t}\mathbf{x}_0 - \sqrt{\alpha_t}\mathbf{x}_0}{\sqrt{1-\alpha_t}} \\
         &= \sqrt{\alpha_{t-1}}\mathbf{x}_0 \\

           \Sigma_{t-1}
         &= \sigma^2_t\mathrm{I}\ +\ (1-\alpha_{t-1} - \sigma^2_t) \cdot \frac{1-\alpha_t}{1-\alpha_t} \mathrm{I} \\
         &= (1-\alpha_{t-1})\mathrm{I}
     \end{align}
     $$

     - 따라서 $q\_\sigma(\mathbf{x}\_{t-1}\|\mathbf{x}\_0)\ =\ \mathcal{N}(\sqrt{\alpha\_{t-1}}\mathbf{x}\_0, (1-\alpha\_{t-1})\mathrm{I})$ 입니다.

   - $t$로부터 $t-1$를 유도하였으므로 귀납법에 의해서 모든 $t$에서 성립함을 증명할 수 있었습니다.

3. 위에서 찾은 $q\_{\sigma}(\mathbf{x}\_t\|\mathbf{x}\_0)$, $q\_\sigma(\mathbf{x}\_{t-1}\|\mathbf{x}\_0)$, $q\_\sigma(\mathbf{x}\_{t-1}\|\mathbf{x}\_t, \mathbf{x}\_0)$을 Bayes' rule을 통해 $q\_\sigma(\mathbf{x}\_{t}\|\mathbf{x}\_{t-1}, \mathbf{x}\_0)$을 구할 수 있습니다.

   $$
   q_\sigma(\mathbf{x}_{t}|\mathbf{x}_{t-1}, \mathbf{x}_0) = q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_{t}, \mathbf{x}_0) \times \frac{q_{\sigma}(\mathbf{x}_t|\mathbf{x}_0)}{q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_0)}
   $$

4. 3번에서 찾은 Bayes' rule을 통해 inference process를 정의할 수 있습니다.

   $$
   q_\sigma(\mathbf{x}_{1:T}|\mathbf{x}_0) := q_\sigma(\mathbf{x}_T|\mathbf{x}_0)\prod^T_{t=2}q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0)
   $$

![](../../assets/img/Paper_Reading/DDIM/ddim_7.jpg){: .normal}

🔎 DDPM의 diffusion process와 달리 DDIM의 forward process(inference process)는 더 이상 Markovian이 아닙니다.

- 더 이상 forward process가 Markov process가 아니므로 diffusion process라고 부르지 않습니다. 따라서 [Section 3](#3-variational-inference-for-non-markovian-forward-process)에서 정리한 내용에 약간의 수정이 필요합니다.

  - Inference process : ~~Diffusion process~~ / Forward process
  - Reverse of inference process : ~~Denoising process~~ / Reverse process / <u>Generative process</u>

🔎 $\sigma$의 크기는 forward process가 stochastic 정도를 통제합니다.

- $\sigma \rightarrow 0$ : 임의의 $t$에 대해 $\mathbf{x}\_{0}$와 $\mathbf{x}\_{t}$를 발견하는 한 하나의 $\mathbf{x}\_{t-1}$를 찾을 수 있는 극단적인 경우에 도달합니다.

  > 💭 DDPM에서는 다양한 $\mathbf{x}\_{t}$가 나타날 수 있는데 DDIM에서는 Non-Markovian을 통해 나올 수 있는 $\mathbf{x}\_{t}$의 경우의 수를 조절하는 것 같습니다. 이것을 조절하는 파라미터가 $\sigma$인 것 같습니다.
  >
  > 💭 그 중에서 $\sigma$가 0일 때 $\mathbf{x}\_{t}$의 경우의 수를 하나로 고정하는 것 같습니다.
  >
  > 💭 이처럼 $\sigma$를 조절해서 이미지를 만들어가는 과정을 reverse process 대신 generative process라 부르는 것 같습니다.

### 3.2 Generative Process And Unified Vairational Inference Objective

![](../../assets/img/Paper_Reading/DDIM/ddim_8.jpg){: .normal}

🔎 훈련 가능한 generative process $p\_{\theta}(\mathbf{x}\_{0:T})$에서 각 $p\_{\theta}^{(t)}(\mathbf{x}\_{t-1}\|\mathbf{x}\_{t})$는 $q\_{\sigma}(\mathbf{x}\_{t-1}\|\mathbf{x}\_{t}, \mathbf{x}\_{0})$을 활용합니다.

🔎 $p\_{\theta}^{(t)}(\mathbf{x}\_{t-1}\|\mathbf{x}\_{t})$를 구하는 방법을 알아보겠습니다.

1. $\mathbf{x}\_{0} \sim q(\mathbf{x}\_{0})$이고 $\epsilon\_{t} \sim \mathcal{N}(\mathrm{0}, \mathrm{I})$ 일 때 앞에서 정의한 식을 reparameterization trick을 활용해 $\mathbf{x}\_t$를 구합니다.

   $$
    \mathbf{x}_t = \sqrt{\alpha_t}\mathbf{x}_0\ +\ \sqrt{1-\alpha_t}\epsilon_t \qquad \because q_{\sigma}(\mathbf{x}_t|\mathbf{x}_0) = \mathcal{N}(\sqrt{\alpha_t}\mathbf{x}_0, (1 -\alpha_t)\mathrm{I})
   $$

2. 모델 $\epsilon\_{\theta}^{t}(\mathbf{x}\_{t})$은 $\mathbf{x}\_{t}$로부터 $\epsilon\_{t}$를 예측하고자 합니다.

   $$
        L_\gamma(\epsilon_\theta) := \sum^T_{t=1}\gamma_t\mathbb{E}_{\mathbf{x}_0\sim q(\mathbf{x}_0), \epsilon_T \sim \mathcal{N}(\mathrm{0}, \mathrm{I})}\Big[ \lVert \epsilon^{(t)}_\theta (\mathbf{x}_{t})\ -\ \epsilon_t \rVert^2_2 \Big]
   $$

3. 모델 $\epsilon\_{\theta}^{t}(\mathbf{x}\_{t})$을 $\mathbf{x}\_t = \sqrt{\alpha\_t}\mathbf{x}\_0\ +\ \sqrt{1-\alpha\_t}\epsilon\_t$에서 $\epsilon\_t$ 대신 대입하고 해당 식을 $\mathbf{x}\_0$으로 정리하면 아래와 같습니다.

   $$
       \begin{align}
       f_{\theta}^{(t)}(\mathbf{x}_{t})
       &=\ \hat{\mathbf{x}}_0 \\
       &=\ (\mathbf{x}_t - \sqrt{1-\alpha_{t}}\cdot \epsilon_{\theta}^{t}(\mathbf{x}_{t})) / \sqrt{\alpha_{t}}
       \end{align}
   $$

4. 따라서 아래와 같이 나타낼 수 있습니다.

   $$
   \begin{align}
   q_{\sigma}(\mathbf{x}_{t-1}|\mathbf{x}_{t}, \hat{\mathbf{x}}_{0})
   &= q_{\sigma}(\mathbf{x}_{t-1}|\mathbf{x}_{t}, f_\theta^{(t)}(\mathbf{x}_t)) \\
   &= \mathcal{N}\big(\sqrt{\alpha_{t-1}}\mathbf{x}_0 + \sqrt{1-\alpha_{t-1} - \sigma^2_t}\cdot \frac{\mathbf{x}_t - \sqrt{\alpha_t}f_{\theta}^{(t)}(\mathbf{x}_{t})}{\sqrt{1-\alpha_t}}, \sigma^2_t\mathrm{I}\big)
   \end{align}
   $$

5. 이제 generative process를 정의하겠습니다.

   - $p\_{\theta}(\mathbf{x}\_{T}) = \mathcal{N}(\mathrm{0}, \mathrm{I})$로 고정
   - Generative process가 모든 곳에서 작동하도록 하기 위해 $t=1$일 때 Gaussian noise를 추가합니다.
     - 위에서 inference process를 정의할 때 $t$가 1보다 클 때 $q\_{\sigma}(\mathbf{x}\_{t-1}\|\mathbf{x}\_{t}, \mathbf{x}\_{0})$을 정의했기 때문에 $t=1$일 때 따로 정의가 필요했음
     - $p\_{\theta}^{(1)}(\mathbf{x}\_{0}\|\mathbf{x}\_{1}) = \mathcal{N}(f\_\theta^{(1)}(\mathbf{x}\_1),\ \sigma\_1^2\mathrm{I})$
   - 마지막으로 generative process를 정의하면 아래와 같습니다.

     $$
         p_{\theta}^{(t)}(\mathbf{x}_{t-1}|\mathbf{x}_{t})=
         \begin{cases}
         \mathcal{N}(f_\theta^{(1)}(\mathbf{x}_1),\ \sigma_1^2\mathrm{I}), & \text{if } t = 1 \\
         q_{\sigma}(\mathbf{x}_{t-1}|\mathbf{x}_{t}, f_\theta^{(t)}(\mathbf{x}_t)), & \text{otherwise}
         \end{cases}
     $$

![](../../assets/img/Paper_Reading/DDIM/ddim_9.jpg){: .normal}

🔎 해당 논문에서는 아래와 같이 objective function을 정의했습니다.

$$
\begin{align}
J_{\sigma} :=& \ \mathbb{E}_{\mathbf{x}_{0:T}\sim q_{\sigma}(\mathbf{x}_{0:T})}[\log q_\sigma(\mathbf{x}_{1:T}|\mathbf{x}_{0}) - \log p_{\theta}(\mathbf{x}_{0:T})]  & \qquad \\
=&\ \mathbb{E}_{\mathbf{x}_{0:T}\sim q_{\sigma}(\mathbf{x}_{0:T})}\Bigg[\bigg(\log q_\sigma(\mathbf{x}_T|\mathbf{x}_0)\prod^T_{t=2}q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t,\mathbf{x}_0) \bigg) - \bigg(\log p_\theta(\mathbf{x}_T)\prod^T_{t=1}p_\theta^{(t)}(\mathbf{x}_{t-1}|\mathbf{x}_t) \bigg)\Bigg]  & \qquad \\
=&
\ \mathbb{E}_{\mathbf{x}_{0:T}\sim q_{\sigma}(\mathbf{x}_{0:T})}\Bigg[ \log q_\sigma(\mathbf{x}_T|\mathbf{x}_0) + \sum^T_{t=2}\log q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_{t}, \mathbf{x
}_{0}) - \sum^T_{t=1} \log p_\theta^{(t)} (\mathbf{x}_{t-1}|\mathbf{x}_t) - \log p_\theta(\mathbf{x}_T) \Bigg]  & \qquad \\
=&
\ \mathbb{E}_{\mathbf{x}_{0:T}\sim q_{\sigma}(\mathbf{x}_{0:T})}\Bigg[ \log q_\sigma(\mathbf{x}_T|\mathbf{x}_0) + \sum^T_{t=2}\log \frac{q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0)}{p^{(t)}_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)} - \log p^{(1)}_\theta(\mathbf{x}_0|\mathbf{x}_1) - \log p_\theta(\mathbf{x}_T) \Bigg]  & \qquad \\
\equiv&
\ \mathbb{E}_{\mathbf{x}_{0:T}\sim q_{\sigma}(\mathbf{x}_{0:T})}\Bigg[ \sum^T_{t=2}\log \frac{q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0)}{p^{(t)}_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)} - \log p^{(1)}_\theta(\mathbf{x}_0|\mathbf{x}_1)\Bigg] & \qquad \because \text{학습에 필요하지 않은 부분 삭제}\\
=&
\ \sum^T_{t=2}\mathbb{E}_{\mathbf{x}_{0:T}\sim q_{\sigma}(\mathbf{x}_{0:T})}\Bigg[\log \frac{q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0)}{p^{(t)}_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}\Bigg] - \mathbb{E}_{\mathbf{x}_{0:T}\sim q_{\sigma}(\mathbf{x}_{0:T})}\big[\log p^{(1)}_\theta(\mathbf{x}_0|\mathbf{x}_1)\big] \\
=&
\ \sum^T_{t=2}D_{KL}(q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t,\mathbf{x}_0)\ ||\ p^{(t)}_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)) - \mathbb{E}_{\mathbf{x}_{0:T}\sim q_{\sigma}(\mathbf{x}_{0:T})}\big[\log p^{(1)}_\theta(\mathbf{x}_0|\mathbf{x}_1)\big]
\end{align}
$$

- 위의 objective function에서 $t > 1$일 때를 살펴보겠습니다.

  $$
  \begin{align}
  D_{KL}(q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t,\mathbf{x}_0)\ ||\ p^{(t)}_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t))
  &= D_{KL}(q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t,\mathbf{x}_0)\ ||\ q_{\sigma}(\mathbf{x}_{t-1}|\mathbf{x}_{t}, f_\theta^{(t)}(\mathbf{x}_t))) & \qquad \\
  &= \mathbb{E}_{\mathbf{x}_{0},\ \mathbf{x}_{t}\ \sim\ q_{\sigma}(\mathbf{x}_{0},\ \mathbf{x}_{t})} \Bigg[\frac{1}{2\sigma^2_t}\lVert \mathbf{x}_0 - f^{(t)}_\theta(\mathbf{x}_t)\rVert^2_2\Bigg] & \qquad \because \text{분산이 같은 두 Gaussian distribution} \\
  &= \mathbb{E}_{\mathbf{x}_{0}\ \sim\ q_{\sigma}(\mathbf{x}_{0}),\ \epsilon\ \sim \mathcal{N}(\mathrm{0},\ \mathrm{I}),\ \mathbf{x}_t=\sqrt{\alpha_t}\mathbf{x}_0 + \sqrt{1-\alpha_t}\epsilon_t} \Bigg[\frac{1}{2\sigma^2_t}\bigg\lVert \frac{\mathbf{x}_t - \sqrt{1-\alpha_t}\epsilon_t}{\sqrt{\alpha_t}} - \frac{\mathbf{x}_t - \sqrt{1-\alpha_t}\epsilon^{(t)}_\theta(\mathbf{x}_t)}{\sqrt{\alpha_t}} \bigg\rVert^2_2  \Bigg] \\
  &= \mathbb{E}_{\mathbf{x}_{0}\ \sim\ q_{\sigma}(\mathbf{x}_{0}),\ \epsilon\ \sim \mathcal{N}(\mathrm{0},\ \mathrm{I}),\ \mathbf{x}_t=\sqrt{\alpha_t}\mathbf{x}_0 + \sqrt{1-\alpha_t}\epsilon_t}\Bigg[\frac{1-\alpha_t}{2\sigma^2_t\alpha_t}\bigg\lVert  \epsilon_t\ -\ \epsilon^{(t)}_\theta(\mathbf{x}_t)\bigg\rVert^2_2  \Bigg]
  \end{align}
  $$

- 이제는 objective function에서 $t=1$일 때를 살펴보겠습니다.

  $$
  \begin{align}
  \mathbb{E}_{\mathbf{x}_{0},\ \mathbf{x}_{1}\ \sim\ q_{\sigma}(\mathbf{x}_{0},\ \mathbf{x}_{1})}\big[-\log p^{(1)}_\theta(\mathbf{x}_0|\mathbf{x}_1)\big]
  &\equiv \mathbb{E}_{\mathbf{x}_{0},\ \mathbf{x}_{1}\ \sim\ q_{\sigma}(\mathbf{x}_{0},\ \mathbf{x}_{1})}\Bigg[\frac{1}{2\sigma^2_1}\bigg\lVert\mathbf{x}_0 - f^{(1)}_\theta(\mathbf{x}_1) \bigg\rVert^2_2\Bigg] \\
  &= \mathbb{E}_{\mathbf{x}_{0}\ \sim\ q_{\sigma}(\mathbf{x}_{0}),\ \epsilon\ \sim \mathcal{N}(\mathrm{0},\ \mathrm{I}),\ \mathbf{x}_1=\sqrt{\alpha_1}\mathbf{x}_0 + \sqrt{1-\alpha_1}\epsilon_1} \Bigg[\frac{1}{2\sigma^2_1}\bigg\lVert \frac{\mathbf{x}_1 - \sqrt{1-\alpha_1}\epsilon_1}{\sqrt{\alpha_1}} - \frac{\mathbf{x}_1 - \sqrt{1-\alpha_1}\epsilon^{(1)}_\theta(\mathbf{x}_1)}{\sqrt{\alpha_1}} \bigg\rVert^2_2  \Bigg] \\
  &= \mathbb{E}_{\mathbf{x}_{0}\ \sim\ q_{\sigma}(\mathbf{x}_{0}),\ \epsilon\ \sim \mathcal{N}(\mathrm{0},\ \mathrm{I}),\ \mathbf{x}_1=\sqrt{\alpha_1}\mathbf{x}_0 + \sqrt{1-\alpha_1}\epsilon_1} \Bigg[\frac{1-\alpha_1}{2\sigma^2_1\alpha_1}\bigg\lVert  \epsilon_1\ -\ \epsilon^{(1)}_\theta(\mathbf{x}_1)\bigg\rVert^2_2  \Bigg]
  \end{align}
  $$

- 따라서 최종 objective function은 다음과 같습니다.

  $$
  \begin{align}
  J_\sigma &\equiv \sum^T_{t=2}D_{KL}(q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t,\mathbf{x}_0)\ ||\ p^{(t)}_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)) - \mathbb{E}_{\mathbf{x}_{0:T}\sim q_{\sigma}(\mathbf{x}_{0:T})}\big[\log p^{(1)}_\theta(\mathbf{x}_0|\mathbf{x}_1)\big] \\
  &= \sum^T_{t=2}\mathbb{E}_{\mathbf{x}_{0}\ \sim\ q_{\sigma}(\mathbf{x}_{0}),\ \epsilon\ \sim \mathcal{N}(\mathrm{0},\ \mathrm{I}),\ \mathbf{x}_t=\sqrt{\alpha_t}\mathbf{x}_0 + \sqrt{1-\alpha_t}\epsilon_t}\Bigg[\frac{1-\alpha_t}{2\sigma^2_t\alpha_t}\bigg\lVert  \epsilon_t\ -\ \epsilon^{(t)}_\theta(\mathbf{x}_t)\bigg\rVert^2_2  \Bigg] + \mathbb{E}_{\mathbf{x}_{0}\ \sim\ q_{\sigma}(\mathbf{x}_{0}),\ \epsilon\ \sim \mathcal{N}(\mathrm{0},\ \mathrm{I}),\ \mathbf{x}_1=\sqrt{\alpha_1}\mathbf{x}_0 + \sqrt{1-\alpha_1}\epsilon_1} \Bigg[\frac{1-\alpha_1}{2\sigma^2_1\alpha_1}\bigg\lVert  \epsilon_1\ -\ \epsilon^{(1)}_\theta(\mathbf{x}_1)\bigg\rVert^2_2  \Bigg] \\
  &= \sum^T_{t=1}\mathbb{E}_{\mathbf{x}_{0}\ \sim\ q_{\sigma}(\mathbf{x}_{0}),\ \epsilon\ \sim \mathcal{N}(\mathrm{0},\ \mathrm{I}),\ \mathbf{x}_t=\sqrt{\alpha_t}\mathbf{x}_0 + \sqrt{1-\alpha_t}\epsilon_t}\Bigg[\frac{1-\alpha_t}{2\sigma^2_t\alpha_t}\bigg\lVert  \epsilon_t\ -\ \epsilon^{(t)}_\theta(\mathbf{x}_t)\bigg\rVert^2_2  \Bigg] \\
  &= \sum^T_{t=1}\frac{1-\alpha_t}{2\sigma^2_t\alpha_t}\mathbb{E}\bigg[\big\lVert  \epsilon_t\ -\ \epsilon^{(t)}_\theta(\mathbf{x}_t)\big\rVert^2_2  \bigg] \\
  &= L_\gamma \qquad \text{when }\gamma_t = \frac{1-\alpha_t}{2\sigma^2_t\alpha_t}
  \end{align}
  $$

- 최종 objective function을 생략 없이 나타낸다면 아래와 같이 나타낼 수 있습니다.

  $$J_\sigma(\epsilon_\theta) = L_\gamma(\epsilon_\theta) + C$$

## 4. Sampling From Generalized Generative Processes

진행 중...
