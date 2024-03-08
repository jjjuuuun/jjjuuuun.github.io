---
layout: post
title: "[Generative Model] DDIM : Denoising Diffusion Implicit Models"
author: kjy
date: 2024-03-08 15:00:00 +09:00
categories: [Deep Learning, Paper Review]
tags: [Deep Learning, Paper Review, Computer Vision, Generative Model]
comments: true
toc: true
math: true
---

해당 포스트에서는 [DDIM : Denoising Diffusion Implicit Models](https://arxiv.org/abs/2010.02502) 논문에서 핵심적인 부분을 알아보겠습니다.

논문을 전체적으로 읽고 싶으신 분은 [Paper Reading](https://jjjuuuun.github.io/posts/Paper-Reading-DDIM/)으로 이동해주세요!

## 1. DDIM 등장 배경

최근에 iterative generative models(DDPM, NCSN)에 대한 연구가 진행 중입니다. Iterative generative models은 GANs과 비교했을 때 학습이 쉽지만 높은 품질의 sample을 생성할 수 있습니다. 그렇지만 sample을 생성하는데 걸리는 시간이 GANs보다 매우 오래 걸린다는 단점이 있습니다. 이러한 단점을 보완하고자 하는 것이 DDIM입니다.

DDIM은 DDPM에서 사용되는 Markov chain과정을 수정하여 sample 생성 속도를 높이고자 했습니다.

## 2. DDIM의 특징

※ [DDIM 정리](#3-ddim-정리)와 이어집니다.

1. Forward process에서 더 이상 Markov chain을 사용하지 않습니다. ([바로 보기](#31-non-markov-process))
2. DDPM과 같은 objective function을 사용해서 학습을 진행합니다. ([바로 보기](#33-objective-function-정의))
3. Sampling 속도가 DDPM보다 많이 빨라졌습니다. ([바로 보기](#35-sampling-속도-향상))
4. Training image의 유일한 latent가 존재합니다. ([바로 보기](#32-generative-process-정의))
   - $\sigma\_t$를 0으로 설정함으로써 sample은 고정된 latent에서 생성이 됩니다.
   - 하나의 latent로 고정되면서 timesteps과 상관없이 high-level image feature를 유지합니다.
   - Latent를 가지고 GANs처럼 semantically meaningful interpolation이 가능합니다.

## 3. DDIM 정리

### 3.1 Non-Markov process

DDIM에서는 sampling 속도를 높이기 위해 DDPM의 forward process를 Non-Markov process로 재정의하여 사용합니다.

$$q_\sigma(\mathbf{x}_{1:T}|\mathbf{x}_0) := q_\sigma(\mathbf{x}_T|\mathbf{x}_0)\prod^T_{t=2}q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0)\tag{1}$$

위의 수식처럼 forward process를 정의하기 위해서는 아래와 같은 사전 정의가 필요합니다. (구체적인 수식 증명은 [Paper Reading](https://jjjuuuun.github.io/posts/Paper-Reading-DDIM/)을 참고해주세요.)

$$
\begin{align}
&q_\sigma(\mathbf{x}_t|\mathbf{x}_0) = \mathcal{N}(\sqrt{\alpha_t}\mathbf{x}_0, (1-\alpha_t)\mathrm{I})\tag{2}\\
&q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) = \mathcal{N}\bigg(\sqrt{\alpha_{t-1}}\mathbf{x}_0 + \sqrt{1-\alpha_{t-1} - \sigma^2_t}\cdot \frac{\mathbf{x}_t - \sqrt{\alpha_t}\mathbf{x}_0}{\sqrt{1-\alpha_t}}, \sigma^2_t\mathrm{I}\bigg)\qquad (t > 1)\tag{3}\\
&q_\sigma(\mathbf{x}_{t}|\mathbf{x}_{t-1}, \mathbf{x}_0) = q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_{t}, \mathbf{x}_0) \times \frac{q_{\sigma}(\mathbf{x}_t|\mathbf{x}_0)}{q_\sigma(\mathbf{x}_{t-1}|\mathbf{x}_0)}\tag{4}
\end{align}
$$

### 3.2 Generative process 정의

- 식 $2$를 변경해서 학습된 모델로부터 $\mathbf{x}\_0$를 예측

  $$
  \begin{align}
  f_{\theta}^{(t)}(\mathbf{x}_{t})
  &=\ \hat{\mathbf{x}}_0 \\
  &=\ (\mathbf{x}_t - \sqrt{1-\alpha_{t}}\cdot \epsilon_{\theta}^{t}(\mathbf{x}_{t})) / \sqrt{\alpha_{t}} \tag{5}
  \end{align}
  $$

- 식 $3$의 $\mathbf{x}\_0$부분에 위에서 예측한 $\mathbf{x}\_0$을 대입

  $$
  \begin{align}
  q_{\sigma}(\mathbf{x}_{t-1}|\mathbf{x}_{t}, \hat{\mathbf{x}}_{0})
  &= q_{\sigma}(\mathbf{x}_{t-1}|\mathbf{x}_{t}, f_\theta^{(t)}(\mathbf{x}_t)) \\
  &= \mathcal{N}\big(\sqrt{\alpha_{t-1}}f_{\theta}^{(t)}(\mathbf{x}_{t}) + \sqrt{1-\alpha_{t-1} - \sigma^2_t}\cdot \frac{\mathbf{x}_t - \sqrt{\alpha_t}f_{\theta}^{(t)}(\mathbf{x}_{t})}{\sqrt{1-\alpha_t}}, \sigma^2_t\mathrm{I}\big) \tag{6}
  \end{align}
  $$

- 최종 generative process 정의

  - 식 $3$에서 $t$의 범위가 $1$보다 큰 경우이므로 $t=1$인 경우 따로 정의를 해주어야 합니다.
  - Generative process가 모든 $t$에서 작동하게 하기위해서 $t=1$일 때 gaussian noise를 추가해줍니다.
  - 그러면 아래와 같이 최종 generative process를 정의할 수 있습니다.

  $$
  p_{\theta}^{(t)}(\mathbf{x}_{t-1}|\mathbf{x}_{t})=
  \begin{cases}
  \mathcal{N}(f_\theta^{(1)}(\mathbf{x}_1),\ \sigma_1^2\mathrm{I}), & \text{if } t = 1 \\
  q_{\sigma}(\mathbf{x}_{t-1}|\mathbf{x}_{t}, f_\theta^{(t)}(\mathbf{x}_t)), & \text{otherwise}
  \end{cases}
  \tag{7}
  $$

### 3.3 Objective function 정의

Forward process의 정의가 달라짐에 따라서 generative process도 바뀐 것 처럼 objective function 또한 바뀌었습니다. 그러나 $\mathbb{E}\left[-\log\ p\_\theta(\mathbf{x}\_0)\right]$을 최소화 하고자 한 것은 DDPM과 다르지 않습니다.

최종 objective function 식을 DDPM과 비교해보겠습니다.

$$
\begin{align}
&L = \sum^T_{t=1}\frac{\beta_t^2}{2\sigma_t^2\cdot\alpha_t\cdot(1-\bar{\alpha}_t)}\mathbb{E}\bigg[\big\lVert  \epsilon_t\ -\ \epsilon^{(t)}_\theta(\mathbf{x}_t)\big\rVert^2_2  \bigg]& \cdots\quad \text{DDPM} \tag{8}\\
&J_\sigma = \sum^T_{t=1}\frac{1-\alpha_t}{2\sigma^2_t\alpha_t}\mathbb{E}\bigg[\big\lVert  \epsilon_t\ -\ \epsilon^{(t)}_\theta(\mathbf{x}_t)\big\rVert^2_2  \bigg]& \cdots\quad \text{DDIM}\tag{9}
\end{align}
$$

DDPM과 DDIM에서 사용하는 notation이 다르기는 하지만 그 차이는 상수배의 차이만 있을 것입니다. 따라서 DDIM의 objective를 아래와 같이 정리할 수 있습니다.

$$J_\sigma(\epsilon_\theta) = L(\epsilon_\theta) + C\tag{10}$$

처음 시작점은 같고 중간과정은 달랐을지 몰라도 DDPM의 objective function를 DDIM에서 사용해도 무방합니다.

### 3.4 $\sigma$가 무엇인가?

Objective function과 sampling을 위한 식 $6$에 $\sigma$가 포함되어 있습니다. 여기서 objective function에서는 DDPM과 같은 objective function을 사용하기 때문에 무시하고 식 $6$만 살펴보겠습니다.

식 $6$을 $\mathbf{x}_{t-1}$에 관한 식으로 정리하면 아래와 같습니다.

$$
\begin{align}
q_{\sigma}(\mathbf{x}_{t-1}|\mathbf{x}_{t}, \hat{\mathbf{x}}_{0}) &= q_{\sigma}(\mathbf{x}_{t-1}|\mathbf{x}_{t}, f_\theta^{(t)}(\mathbf{x}_t)) & \qquad \\

&= \mathcal{N}\big(\sqrt{\alpha_{t-1}}f_{\theta}^{(t)}(\mathbf{x}_{t}) + \sqrt{1-\alpha_{t-1} - \sigma^2_t}\cdot \frac{\mathbf{x}_t - \sqrt{\alpha_t}f_{\theta}^{(t)}(\mathbf{x}_{t})}{\sqrt{1-\alpha_t}}, \sigma^2_t\mathrm{I}\big) & \qquad \\ &= \sqrt{\alpha_{t-1}}f_{\theta}^{(t)}(\mathbf{x}_{t}) + \sqrt{1-\alpha_{t-1} - \sigma^2_t}\cdot \frac{\mathbf{x}_t - \sqrt{\alpha_t}f_{\theta}^{(t)}(\mathbf{x}_{t})}{\sqrt{1-\alpha_t}} + \sigma_t \epsilon_t & \qquad \\

&= \sqrt{\alpha_{t-1}}\Bigg(\frac{\mathbf{x}_t - \sqrt{1-\alpha_{t}}\cdot \epsilon_{\theta}^{t}(\mathbf{x}_{t})}{\sqrt{\alpha_{t}}} \Bigg) + \sqrt{1-\alpha_{t-1} - \sigma^2_t}\cdot \frac{\mathbf{x}_t - \sqrt{\alpha_t}\big(\frac{\mathbf{x}_t - \sqrt{1-\alpha_{t}}\cdot \epsilon_{\theta}^{t}(\mathbf{x}_{t})}{\sqrt{\alpha_{t}}} \big)}{\sqrt{1-\alpha_t}} + \sigma_t \epsilon_t & \qquad \\

&= \sqrt{\alpha_{t-1}}\underset{\text{"predicted }\mathbf{x}_{0}\text{"}}{\underbrace{\bigg(\frac{\mathbf{x}_{t} - \sqrt{1-\alpha_{t}}\epsilon^{(t)}_{\theta}(\mathbf{x}_{t})}{\sqrt{\alpha_{t}}}\bigg)}}\ +\ \underset{\text{"direction pointing to }\mathbf{x}_{t}\text{"}}{\underbrace{\sqrt{1 - \alpha_{t-1} - \sigma^{2}_{t}}\ \cdot\ \epsilon^{(t)}_{\theta}(\mathbf{x}_{t})}}\ +\ \underset{\text{random noise}}{\underbrace{\sigma_{t}\epsilon_{t}}} & \qquad \tag{11}\\

\end{align}
$$

만약 $\sigma$가 $0$이면 random noise에 해당하는 부분이 $0$이 되므로 random하지 않게 sampling이 가능하게 됩니다. 즉, $\sigma$는 stochastic한 정도를 조절하는 parameter라고 할 수 있습니다. $\sigma$가 $0$일 때 DDIM이라고 부르며 random 하지 않으므로 sample은 고정된 latent로부터 생성되게 됩니다.

그러나 random noise에 해당하는 부분에서 $\sigma$가 DDPM에서의 $\tilde{\beta}$와 같다면 DDPM이 됩니다. DDPM에서의 $\tilde{\beta}$를 DDIM notation으로 바꾸면 $\sigma_{t} = \sqrt{(1-\alpha_{t-1})/(1-\alpha_{t})}\sqrt{1-\alpha_{t}/\alpha_{t-1}}$이고 이 때 DDPM의 sampling과 같다고 할 수 있습니다.

### 3.5 Sampling 속도 향상

Forward process가 더 이상 Markov process를 따르지 않기 때문에 $q\_\sigma(\mathbf{x}\_t\|\mathbf{x}\_0)$만 고정되어 있다면 $1 \sim T$ 모두 sampling 할 필요가 없습니다.

왜냐하면 $q\_\sigma(\mathbf{x}\_t\|\mathbf{x}\_0)$만 고정되어 있다면 식 $11$을 $t-1$이 아닌 다른 $t-2$ 또는 $t-3$ 등으로 변경할 수 있기 때문입니다.

여기서 중요한 것은 forward process의 역이 generative process이기 때문에 학습을 timestep $[1,\dots,T]$이 아닌 subset $[\tau_1,\dots,\tau_S]$에 대해 해야 합니다. 그렇다고 해서 앞서서 말한 objective function이 변하지 않습니다.
