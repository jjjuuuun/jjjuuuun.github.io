---
layout: post
title: "[Generative Model] DDPM : Denoising Diffusion Probabilistic Models"
author: kjy
date: 2024-01-23 17:00:00 +09:00
categories: [Deep Learning, Paper Review]
tags: [Deep Learning, Paper Review, Computer Vision, Generative Model]
comments: true
toc: true
math: true
---

해당 포스트에서는 [DDPM : Denoising Diffusion Probabilistic Models](https://arxiv.org/abs/2006.11239) 논문에서 핵심적인 부분들을 코드와 함께 알아보겠습니다.

논문을 전체적으로 읽고 싶으신 분은 [Paper Reading](https://jjjuuuun.github.io/posts/Paper-Reading-DDPM/)으로 이동해주세요!

## 1. DDPM의 Training & Sampling

### 1.1 Training

DDPM의 Objective function을 구하는 과정을 간단히 한 번 살펴보겠습니다.

1. NLL(Negative Log Likelihood)

   $$\mathbb{E}\left[-\log\ p_\theta(\mathbf{x}_0)\right]$$

2. NLL을 variational inference를 통해 Negative ELBO term을 최소화하는 문제로 변경

   $$\mathbb{E}_\left[-\log\ p_\theta(\mathbf{x}_0)\right] \le \mathbb{E}_q\left[-\log\ \frac{p(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}\right] = \mathbb{E}_q\left[-\log\ p(\mathbf{x}_T) -\sum_{t=1}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_t|\mathbf{x}_{t-1})}\right] =: L$$

3. $\mathbf{x}\_0$를 condition으로 추가

   $$
   \begin{align}
   L

   &= \mathbb{E}_q\left[-\log\ p(\mathbf{x}_T) -\sum_{t=1}^T\log \frac{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}{q(\mathbf{x}_t|\mathbf{x}_{t-1})}\right] \\

   &= D_{KL}(q(\mathbf{x}_T|\mathbf{x}_0)||p(\mathbf{x}_T)) + \sum_{t=2}^T D_{KL}(q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_{0}) || p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)) + \mathbb{E}_{\mathbf{x}_T\ \sim\ q(\mathbf{x}_T|\mathbf{x}_0)}\left[- \log\ p_\theta(\mathbf{x}_{0}|\mathbf{x}_1)\right] \\

   &= L_T + L_{1:T-1} + L_0

   \end{align}
   $$

4. $\beta\_t$를 constant로 고정하면서 학습해야하는 term이 줄어듬

   $$
   \begin{align}
   L

   &= L_T + L_{1:T-1} + L_0 \\

   &\approx L_{1:T-1} + L_0

   \end{align}
   $$

5. $L\_{1:T-1}$의 $p\_\theta(\mathbf{x}\_{t-1}\|\mathbf{x}\_t)$의 분산($\sigma\_t^2$)을 timestep에 의존적이며 학습이 필요없는 $\beta\_t$ 또는 $\tilde\beta\_t$로 설정하면서 더 간단히 표현이 가능

   $$
   \begin{align}
   L

   &= L_{1:T-1} + L_0 \\

   &= \sum_{t=2}^T \mathbb{E}_q\left[\frac{1}{2\sigma_t^2}\lVert\tilde{\mu}_t(\mathbf{x}_t, \mathbf{x}_0) - \mu_\theta(\mathbf{x}_t, t)\rVert^2\right] + L_0

   \end{align}
   $$

6. Reparameterization trick과 function approximator $\epsilon\_\theta$를 통해 더 간단히 표현이 가능

   $$
   \begin{align}
   L

   &= \sum_{t=2}^T \mathbb{E}_q\left[\frac{1}{2\sigma_t^2}\lVert\tilde{\mu}_t(\mathbf{x}_t, \mathbf{x}_0) - \mu_\theta(\mathbf{x}_t, t)\rVert^2\right] + L_0 \\

   &= \mathbb{E}_{\mathbf{x}_0,\ \epsilon}\left[\frac{\beta_t^2}{2\sigma_t^2\cdot\alpha_t\cdot(1-\bar{\alpha}_t)}\lVert\epsilon - \epsilon_\theta(\mathbf{x}_t, t)\rVert^2\right] + L_0

   \end{align}
   $$

7. $L\_0$를 discrete decoder로 정의하고 $L\_{1:T-1}$에서 상수를 제거하면 $L$를 한 번에 표현이 가능

   $$
   \begin{align}
   L

   &= \mathbb{E}_{\mathbf{x}_0,\ \epsilon}\left[\frac{\beta_t^2}{2\sigma_t^2\cdot\alpha_t\cdot(1-\bar{\alpha}_t)}\lVert\epsilon - \epsilon_\theta(\mathbf{x}_t, t)\rVert^2\right] + L_0 &\\

   &\approx \mathbb{E}_{\mathbf{x}_0,\ \epsilon}\left[\lVert\epsilon - \epsilon_\theta(\mathbf{x}_t, t)\rVert^2\right] &\\

   &= \mathbb{E}_{\mathbf{x}_0,\ \epsilon}\left[\lVert\epsilon - \epsilon_\theta(\sqrt{\bar{\alpha}_t}\mathbf{x}_0 + \sqrt{1-\bar{\alpha}_t}\cdot\epsilon, t)\rVert^2\right] &\because \mathbf{x}_t \sim q(\mathbf{x}_t|\mathbf{x}_0) = \mathcal{N}(\mathbf{x}_t; \sqrt{\bar{\alpha}_t}\mathbf{x}_0, (1-\bar{\alpha}_t)\mathrm{I})

   \end{align}
   $$

### 1.2 Sampling

1. [Training](#11-training)의 5번을 최소화 하려면 아래와 같이 $p\_\theta(\mathbf{x}\_{t-1}\|\mathbf{x}\_t)$의 평균($\mu\_\theta(\mathbf{x}\_t, t)$)이 아래와 같아야 함을 알 수 있습니다.

   $$
   \mu_\theta(\mathbf{x}_t, t) = \tilde{\mu}_t(\mathbf{x}_t, \mathbf{x}_0)
   $$

2. $\mathbf{x}\_t \sim q(\mathbf{x}\_t\|\mathbf{x}\_0) = \mathcal{N}(\mathbf{x}\_t; \sqrt{\bar{\alpha}\_t}\mathbf{x}\_0, (1-\bar{\alpha}\_t)\mathrm{I})$이므로 reparameterization tirck을 통해 $\mathbf{x}\_t$를 $\sqrt{\bar{\alpha}\_t}\mathbf{x}\_0 + \sqrt{1-\bar{\alpha}\_t}\cdot\epsilon$로 표현할 수 있고 해당 식을 $\mathbf{x}\_0$로 정리하여 $\tilde{\mu}\_t(\mathbf{x}\_t, \mathbf{x}\_0)$를 표현하면 아래와 같습니다.

   $$
   \begin{align}
   \mu_\theta(\mathbf{x}_t, t)

   &= \tilde{\mu}_t(\mathbf{x}_t, \mathbf{x}_0) \\

   &= \tilde{\mu}_t\left(\mathbf{x}_t, \frac{1}{\sqrt{\alpha_t}}(\mathbf{x}_t - \sqrt{1-\bar{\alpha}_t}\epsilon_\theta(\mathbf{x}_t))\right)

   \end{align}
   $$

3. 논문에서의 공식 7번($\tilde{\mu}\_t(\mathbf{x}\_{t}, \mathbf{x}\_{0}) := \frac{\sqrt{\bar{\alpha}\_{t-1}}\beta\_t}{1-\bar{\alpha}\_t}\mathbf{x}\_0 + \frac{\sqrt{\alpha\_t}(1-\bar{\alpha}\_{t-1})}{1-\bar{\alpha}\_t}\mathbf{x}_t$)을 통해 위의 식을 정리하면 아래와 같아집니다.

   $$
   \begin{align}
   \mu_\theta(\mathbf{x}_t, t)

   &= \tilde{\mu}_t(\mathbf{x}_t, \mathbf{x}_0) \\

   &= \tilde{\mu}_t\left(\mathbf{x}_t, \frac{1}{\sqrt{\alpha_t}}(\mathbf{x}_t - \sqrt{1-\bar{\alpha}_t}\epsilon_\theta(\mathbf{x}_t))\right) \\

   &= \frac{1}{\sqrt{\alpha_t}}\Bigg(\mathbf{x}_t - \frac{\beta_t}{\sqrt{1 - \bar{\alpha}_t}}\cdot\epsilon_\theta(\mathbf{x}_t, t)\Bigg)

   \end{align}
   $$

4. 위에서 찾은 평균과 논문에서 설정한 분산을 통해 Sampling 할 수 있습니다. 이를 총 정리하면 아래와 같습니다.

   $$
   \begin{align}

   p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)

   &= \mathcal{N}(\mathbf{x}_{t-1}; \mu_\theta(\mathbf{x}_t, t), \Sigma_\theta(\mathbf{x}_t, t)) \qquad & \\

   &= \mathcal{N}(\mathbf{x}_{t-1}; \mu_\theta(\mathbf{x}_t, t), \sigma^2_t) \qquad & \sigma^2_t = \beta_t\ \text{or}\ \sigma^2_t = \tilde{\beta}_t \\

   &= \mathcal{N}\Bigg(\mathbf{x}_{t-1}; \frac{1}{\sqrt{\alpha_t}}\bigg(\mathbf{x}_t - \frac{\beta_t}{\sqrt{1 - \bar{\alpha}_t}}\cdot\epsilon_\theta(\mathbf{x}_t, t)\bigg), \sigma^2_t\Bigg) \qquad & \\

   \mathbf{x}_{t-1} &= \frac{1}{\sqrt{\alpha_t}}\bigg(\mathbf{x}_t - \frac{\beta_t}{\sqrt{1 - \bar{\alpha}_t}}\cdot\epsilon_\theta(\mathbf{x}_t, t)\bigg) + \sigma_t\cdot \mathbf{z} \quad \text{where}\ \mathbf{z} \sim \mathcal{N}(\mathrm{0}, \mathrm{I}) \qquad & \because \text{Reparameterization trick}\\

   \end{align}
   $$

### 1.3 Training을 위한 objective function과 Sampling을 코드로 나타내면 아래와 같습니다.

```python
class ddpm:
    def __init__(self, ddpm: nn.Module, T: int, device:torch.device):
        super().__init__()

        # Paper - 4. Experiments : U-Net backbone
        self.ddpm = ddpm

        # Paper - 4. Expeirments : Set T = 1000
        self.T = T

        # Paper - 2. Background : \beta_t -> reparameterization or constant
        # Paper - 4. Expeirments : Forward process variances to constants increasing linearly
        self.beta_t = torch.linspace(1e-4, 0.02, T).to(device)

        # Paper - 2. Background : Closed form
        self.alpha_t = 1. - self.beta_t
        self.alpha_bar_t = torch.cumprod(self.alpha_t, dim = 0)

        # 3.2 Reverse process :
        # set \Sigma_{\theta}(x_t, t) = \sigma^2_tI to untrained time dependent constants
        # and \sigma^2_t = \beta_t or \sigma^2_t = \tilde{beta}_t had similar results
        self.sigma2t = self.beta_t

    def q_xt_x0(self, x0: torch.Tensor, t: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
        """
        q(x_t|x_0) = N(x_t; \sqrt{\bar{\alpha}_t}x_0, (1 - \bar{\alpha}_t))
        """

        # 주어진 t에 대한 모든 \bar{\alpha}_t
        alpha_bar_t = self.alpha_bar_t.gather(-1, t).reshape(-1, 1, 1, 1)

        # q(x_t|x_0)의 평균 = \sqrt{\bar{\alpha}_t} * x0
        mean = alpha_bar_t ** 0.5 *x0

        # q(x_t|x_0)의 분산 = 1 - \bar{\alpha}_t
        var = 1 - alpha_bar_t

        return mean, var

    def q_sample(self, x0: torch.Tensor, t: torch.Tensor, epsilon: Optional[torch.Tensor] = None):
        """
        Forward process (Diffusion Process)

        q(x_t|x_0) = N(x_t; \sqrt{\bar{\alpha}_t}x_0, (1 - \bar{\alpha}_t))
        x_t = mean + \sqrt{var}*\epsilon (\epsilon ~ N(0, I))
            = \sqrt{\bar{\alpha}_t}x_0 + (1 - \bar{\alpha}_t) ** 0.5 * \epsilon
        """

        if epsilon is None:
            epsilon = torch.randn_like(x0)

        mean, var = self.q_xt_x0(x0, t)

        xt = mean + (var ** 0.5) * epsilon

        return xt

    def p_sample(self, xt: torch.Tensor, t: torch.Tensor):
        """
        Reverse process (Denoising process)

        x_{t-1} ~ p_\theta(x_{t-1})|x_t)
        p_\theta(x_{t-1}|x_t)   := N(x_{t-1}; \mu_\theta, \Sigma_\theta)
                                := N(x_{t-1}; \mu_\theta, \beta_t)
                                := N(x_{t-1}; 1/\sqrt{\alpha_t}(x_t - (\beta_t/\sqrt{1 - \bar{\alpha}_t})\epsilon_\theta(x_t, t)), \beta_t)
        x_{t-1} = \mu_\theta + \sqrt{\Sigma_\theta} * z (z ~ N(0, I))
                = 1/\sqrt{\alpha_t}(x_t - (\beta_t/\sqrt{1 - \bar{\alpha}_t})\epsilon_\theta(x_t, t)) + \sqrt{\beta_t} * z
                = denoise_xt
        """
        epsilon_theta = self.ddpm(xt, t)

        # 주어진 t에 대한 모든 \bar{\alpha}_t
        alpha_bar_t = self.alpha_bar_t.gather(-1, t.reshape(-1, 1, 1, 1))

        # 주어진 t에 대한 모든 \beta_t
        beta_t = self.beta_t.gather(-1, t).reshape(-1, 1, 1, 1)

        # 주어진 t에 대한 모든 \alpha_t
        alpha_t = 1 - beta_t

        # \mu_\theta
        mean = 1/(alpha_t**0.5) * (xt - (beta_t/(1-alpha_bar_t)**0.5)*epsilon_theta)

        # \Sigma_\theta = \sigma^2_t = \beta_t
        var = beta_t

        # z ~ N(0, I)
        z = torch.randn(xt.shape, device=xt.device)

        # x_{t-1} = \mu_\theta + \sqrt{\Sigma_\theta} * z
        denoise_xt = mean + (var ** 0.5) * z


    def loss_simple(self, x0: torch.Tensor, epsilon: Optional[torch.Tensor]=None):
        """
        L_simple    = L2(epsilon, epsilon_theta)
                    = L2(epsilon, xt ~ q(xt|x0))

        Function approximator \epsilon_theta와 Gaussian noise(\epsilon)를 통해 training
        """
        batch_size = x0.shape[0]

        t = torch.randint(0, self.T, (batch_size,), device=x0.device, dtype=torch.long)

        if epsilon is None:
            epsilon = torch.randn_like(x0)

        xt = self.q_sample(x0, t, epsilon=epsilon)

        epsilon_theta = self.ddpm(xt, t)

        return F.mse_loss(epsilon, epsilon_theta)
```

## 2. DDPM의 backbone (U-Net)

## 3. Interpolation

## ※ Reference

- [https://nn.labml.ai/diffusion/ddpm/index.html](https://nn.labml.ai/diffusion/ddpm/index.html)
