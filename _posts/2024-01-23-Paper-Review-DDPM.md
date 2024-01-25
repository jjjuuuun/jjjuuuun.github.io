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

[Colab](https://colab.research.google.com/drive/1iKDLKzQulE98Ay5BZ2CgT8B60uDDyajL?usp=sharing)에서 코드 한 번에 보기

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
class DDPM:
    def __init__(self, backbone: nn.Module, T: int, device:torch.device):
        super().__init__()

        # Paper - 4. Experiments : U-Net backbone
        self.backbone = backbone

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
        q(x_t|x_0) = N(x_t; \sqrt(\bar{\alpha}_t)x_0, (1 - \bar{\alpha}_t))
        """

        # 주어진 t에 대한 모든 \bar{\alpha}_t
        alpha_bar_t = self.alpha_bar_t.gather(-1, t).reshape(-1, 1, 1, 1)

        # q(x_t|x_0)의 평균 = \sqrt(\bar{\alpha}_t) * x0
        mean = alpha_bar_t ** 0.5 *x0

        # q(x_t|x_0)의 분산 = 1 - \bar{\alpha}_t
        var = 1 - alpha_bar_t

        return mean, var

    def q_sample(self, x0: torch.Tensor, t: torch.Tensor, epsilon: Optional[torch.Tensor] = None):
        """
        Forward process (Diffusion Process)

        q(x_t|x_0) = N(x_t; \sqrt(\bar{\alpha}_t)x_0, (1 - \bar{\alpha}_t))
        x_t = mean + \sqrt(var)*\epsilon (\epsilon ~ N(0, I))
            = \sqrt(\bar{\alpha}_t)x_0 + (1 - \bar{\alpha}_t) ** 0.5 * \epsilon
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
        x_{t-1} = \mu_\theta + \sqrt(\Sigma_\theta) * z (z ~ N(0, I))
                = 1/\sqrt{\alpha_t}(x_t - (\beta_t/\sqrt{1 - \bar{\alpha}_t})\epsilon_\theta(x_t, t)) + \sqrt(\beta_t) * z
                = denoise_xt
        """
        epsilon_theta = self.backbone(xt, t)

        # 주어진 t에 대한 모든 \bar{\alpha}_t
        alpha_bar_t = self.alpha_bar_t.gather(-1, t).reshape(-1, 1, 1, 1)

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

        # x_{t-1} = \mu_\theta + \sqrt(\Sigma_\theta) * z
        denoise_xt = mean + (var ** 0.5) * z

        return denoise_xt

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

        epsilon_theta = self.backbone(xt, t)

        return F.mse_loss(epsilon, epsilon_theta)
```

## 2. DDPM의 backbone (U-Net)

DDPM에서는 Unmasked PixelCNN++과 비슷한 U-Net backbone을 사용했으며 group normalization을 사용했습니다.

또한 transformer에서 사용한 positional encoding을 time embedding을 하는 데 사용했으며 $16 \times 16$ 사이에 있는 convolution network에는 self-attention을 사용했습니다. 즉, $8 \times 8$, $4 \times 4$에 self-attention을 적용했다는 이야기가 됩니다.

### 2.1 U-Net 그림으로 살펴보기

먼저 모델의 구성을 그림으로 그리면 아래와 같습니다. ([코드 먼저 보기](#22-u-net-코드))

※ 아래 코드의 Up, Down은 각각 UpBlock, DonwBlock의 집합을 이야기 하며 하나의 라인을 뜻합니다.

※ 그림에서 색이 칠해진 부분은 U-Net 구조에서 concatenate 되는 부분을 뜻합니다.

![](../../assets/img/Paper_Review/DDPM/ddpm_1.png){: .normal}

### 2.2 U-Net 코드

위의 그림을 통해 살펴본 U-Net 구조를 코드를 나타내면 아래와 같습니다. ([그림 다시 보기](#21-u-net-그림으로-살펴보기))

※ 코드의 Up, Down은 각각 UpBlock, DonwBlock의 집합을 이야기 하며 하나의 라인을 뜻합니다.(그림에는 표시가 되어 있지 않음)

※ 주석을 함께 보시면 논문에 어떻게 기술되어 있는지 비교할 수 있습니다.

#### 2.2.1 Time Embedding

```python
class TimeEmbedding(nn.Module):
    def __init__(self, n_channels: int):
        """
        Appendix B. Experiments details about Residual blocks
            - Diffusion time t is specified by adding the Transformer sinusoidal position embedding into each residual block.
        """
        super().__init__()

        self.n_channels = n_channels
        self.embed1 = nn.Linear(self.n_channels // 4, self.n_channels)
        self.silu = nn.SiLU()
        self.embed2 = nn.Linear(self.n_channels, self.n_channels)

    def forward(self, t: torch.Tensor):
        half_dim = self.n_channels // 8
        emb = math.log(10_000) / (half_dim - 1)
        emb = torch.exp(torch.arange(half_dim) * -emb)
        emb = t[:, None] * emb[None, :]
        emb = torch.cat((emb.sin(), emb.cos()), dim=1)

        t = self.embed1(emb)
        t = self.silu(t)
        t = self.embed2(t)

        return t
```

#### 2.2.2 Residual Block

```python
class ResidualBlock(nn.Module):
    def __init__(self, in_channels: int, out_channels: int, time_channels: int, n_groups: int, dropout_rate: float):
        """
        Appendix B. Experiments details about Residual blocks
            - Wide ResNet
            - We replaced weight normalization with group normalization to make the implementation simpler
            - Diffusion time t is specified by adding the Transformer sinusoidal position embedding into each residual block

            - We set the dropout rate on CIFAR10 to 0.1 by sweeping over the values {0.1, 0.2, 0.3, 0.4}
        """
        super().__init__()

        # 현재 n_feature에 맞춰 time embedding 변경
        self.time_proj = nn.Linear(time_channels, out_channels)
        self.time_silu = nn.SiLU()

        self.norm1 = nn.GroupNorm(n_groups, in_channels)
        self.silu1 = nn.SiLU()
        self.conv1 = nn.Conv2d(in_channels, out_channels, kernel_size=3, stride=1, padding=1)

        self.norm2 = nn.GroupNorm(n_groups, out_channels)
        self.silu2 = nn.SiLU()
        self.conv2 = nn.Conv2d(out_channels, out_channels, kernel_size=3, stride=1, padding=1)

        if in_channels != out_channels:
            self.shortcut = nn.Conv2d(in_channels, out_channels, kernel_size=1, stride=1, padding=0)
        else:
            self.shortcut = nn.Identity()

        self.dropout = nn.Dropout(dropout_rate)

    def forward(self, x: torch.Tensor, t: torch.Tensor):
        residual = x

        x = self.norm1(x)
        x = self.silu1(x)
        x = self.conv1(x)

        t = self.time_silu(t)
        t = self.time_proj(t)
        t = t[:, :, None, None]

        x += t

        x = self.norm2(x)
        x = self.silu2(x)
        x = self.dropout(x)
        x = self.conv2(x)

        residual = self.shortcut(residual)
        x += residual

        return x
```

#### 2.2.3 Self Attention Block

```python
class SelfAttentionBlock(nn.Module):
    def __init__(self, n_channels: int, n_heads: int, n_groups: int):
        """
        Appendix B. Experiments details about Residual blocks
            - All models have self-attention blocks at the 16 × 16 resolution between the convolutional blocks
            - We replaced weight normalization with group normalization to make the implementation simpler.
        """
        super().__init__()

        self.n_heads = n_heads
        self.d_k = n_channels

        self.proj = nn.Linear(n_channels, n_heads * self.d_k * 3) # W_q, W_k, W_v
        self.output = nn.Linear(n_heads * self.d_k, n_channels) # W_o

        self.norm = nn.GroupNorm(n_groups, n_channels)


    def forward(self, x: torch.Tensor, t: torch.Tensor):
        batch_size, n_channels, height, width = x.shape

        # Normalization
        x = self.norm(x)

        # Query, Key, Value
        x = x.view(batch_size, n_channels, -1).permute(0, 2, 1)
        proj_x = self.proj(x)
        qkv = proj_x.view(batch_size, -1, self.n_heads, self.d_k * 3)
        q, k, v = torch.chunk(qkv, 3, dim = -1)

        # Multi-Head Attention
        attention_score = torch.einsum("bihd, bjhd -> bijh", q, k) * (self.d_k ** -0.5) # QK^T/\sqrt(d_k)
        attention_score = attention_score.softmax(dim=2) # Softmax(QK^T/\sqrt(d_k))
        attention = torch.einsum("bijh, bjhd -> bihd", attention_score, v) # Softmax(QK^T/\sqrt(d_k)) * V
        attention = attention.view(batch_size, -1, self.n_heads * self.d_k)
        attention_output  = self.output(attention)

        # Skip Connection
        x += attention_output

        # Recover shape
        x = x.permute(0, 2, 1).view(batch_size, n_channels, height, width)

        return x
```

#### 2.2.4 DownBlock

```python
class DownBlock(nn.Module):
    def __init__(self, in_channels: int, out_channels: int, has_attention: bool,
                 time_channels: int, n_groups: int, dropout_rate: float, n_heads: int):
        """
        Appendix B. Experiments details about Residual blocks
            - All models have self-attention blocks at the 16 × 16 resolution between the convolutional blocks
        """
        super().__init__()

        self.has_attention = has_attention

        self.residual_block = ResidualBlock(in_channels, out_channels, time_channels, n_groups, dropout_rate)
        self.attn = SelfAttentionBlock(out_channels, n_heads, n_groups)

    def forward(self, x: torch.Tensor, t: torch.Tensor):
        x = self.residual_block(x, t)
        if self.has_attention:
            x = self.attn(x, t)

        return x
```

#### 2.2.5 Down

```python
class Down(nn.Module):
    def __init__(self, in_channels: int, out_channels: int, has_attention: bool,
                 time_channels: int, n_groups: int, dropout_rate: float, n_heads: int, is_down: bool):
        """
        Appendix B. Experiments details about Residual blocks
            - All models have two convolutional residual blocks per resolution level
        """
        super().__init__()

        self.is_down = is_down
        self.down_sample = nn.Conv2d(in_channels, in_channels, kernel_size=3, stride=2, padding=1)

        self.down_block1 = DownBlock(in_channels, out_channels, has_attention, time_channels, n_groups, dropout_rate, n_heads)
        self.down_block2 = DownBlock(out_channels, out_channels, has_attention, time_channels, n_groups, dropout_rate, n_heads)

    def forward(self, x, t):
        result = [] # For skip connection

        if self.is_down:
            x = self.down_sample(x)
        result.append(x)

        x = self.down_block1(x, t)
        result.append(x)

        x = self.down_block2(x, t)
        result.append(x)

        return x, result
```

#### 2.2.6 Middle

```python
class Middle(nn.Module):
    def __init__(self, in_channels: int, out_channels: int, time_channels: int,
                 n_groups: int, dropout_rate: float, n_heads: int):
        super().__init__()

        self.residual_block1 = ResidualBlock(in_channels, out_channels, time_channels, n_groups, dropout_rate)
        self.attn = SelfAttentionBlock(out_channels, n_heads, n_groups)
        self.residual_block2 = ResidualBlock(in_channels, out_channels, time_channels, n_groups, dropout_rate)

    def forward(self, x, t):
        x = self.residual_block1(x, t)
        x = self.attn(x, t)
        x = self.residual_block2(x, t)

        return x
```

#### 2.2.7 UpBlock

```python
class UpBlock(nn.Module):
    def __init__(self, in_channels: int, out_channels: int, has_attention: bool, time_channels: int,
                 n_groups: int, dropout_rate: float, n_heads: int):
        """
        Appendix B. Experiments details about Residual blocks
            - All models have self-attention blocks at the 16 × 16 resolution between the convolutional blocks
        """
        super().__init__()

        self.has_attention = has_attention

        self.residual_block = ResidualBlock(in_channels + out_channels, out_channels, time_channels, n_groups, dropout_rate)
        self.attn = SelfAttentionBlock(out_channels, n_heads, n_groups)

    def forward(self, x, t):
        x = self.residual_block(x, t)
        if self.has_attention:
            x = self.attn(x, t)

        return x
```

#### 2.2.8 Up

```python
class Up(nn.Module):
    def __init__(self, in_channels: int, out_channels: int, has_attention: bool,
                 time_channels: int, n_groups: int, dropout_rate: float, n_heads: int, is_up: bool):
        """
        Appendix B. Experiments details about Residual blocks
            - All models have two convolutional residual blocks per resolution level
        """
        super().__init__()

        self.is_up = is_up
        self.up_sample = nn.ConvTranspose2d(out_channels, out_channels, kernel_size=4, stride=2, padding=1)

        self.up_block1 = UpBlock(in_channels, in_channels, has_attention, time_channels, n_groups, dropout_rate, n_heads) # down_{3-i}의 결과와 concatenate
        self.up_block2 = UpBlock(in_channels, in_channels, has_attention, time_channels, n_groups, dropout_rate, n_heads) # down_{3-i}의 결과와 concatenate
        self.up_block3 = UpBlock(in_channels, out_channels, has_attention, time_channels, n_groups, dropout_rate, n_heads) # down_{3-i}의 결과와 concatenate

    def forward(self, x, t, down):
        x = torch.cat((x, down[-1]), dim=1)
        x = self.up_block1(x, t)

        x = torch.cat((x, down[-2]), dim=1)
        x = self.up_block2(x, t)

        x = torch.cat((x, down[-3]), dim=1)
        x = self.up_block3(x, t)

        if self.is_up:
            x = self.up_sample(x)

        return x

```

#### 2.2.9 U-Net

```python
class UNet(nn.Module):
    def __init__(self, img_channels: int = 3, proj_channels: int = 64,
                 n_groups: int = 32, dropout_rate: float = 0.1, n_heads: int = 1,
                 has_down_attention: Union[Tuple[bool, ...], List[bool, ]] = [False, False, True, True],
                 has_up_attention: Union[Tuple[bool, ...], List[bool, ]] = [True, True, False, False],
                 is_down: Union[Tuple[bool, ...], List[bool, ]] = [False, True, True, True],
                 is_up: Union[Tuple[bool, ...], List[bool, ]] = [True, True, True, False]):
        """
        Appendix B. Our neural network architecture follows the backbone of PixelCNN++, which is a U-Net based on a Wide ResNet.
        """
        super().__init__()

        # 이해를 위해 필요한 parameter를 4개 생성. But, U-Net 구조 때문에 대칭이기 때문에 두 개만 사용해도 됨
        assert len(has_down_attention) == len(has_up_attention) == len(is_down) == len(is_up)
        assert has_down_attention == has_up_attention[::-1]
        assert is_down == is_up[::-1]

        # Setting
        time_channels = proj_channels * 4

        # Time Ebedding
        self.time_embedding = TimeEmbedding(time_channels)

        # Image projection 1
        self.proj1 = nn.Conv2d(img_channels, proj_channels, kernel_size=3, stride=1, padding=1) # Channels : 3 -> 64

        # Down
        self.down_0 = Down(64, 64, has_down_attention[0], time_channels, n_groups, dropout_rate, n_heads, is_down[0]) # (-1, 64, 32, 32) -> (-1, 64, 32, 32)
        self.down_1 = Down(64, 128, has_down_attention[1], time_channels, n_groups, dropout_rate, n_heads, is_down[1]) # (-1, 64, 32, 32) -> (-1, 128, 16, 16)
        self.down_2 = Down(128, 256, has_down_attention[2], time_channels, n_groups, dropout_rate, n_heads, is_down[2]) # (-1, 128, 16, 16) -> (-1, 256, 8, 8)
        self.down_3 = Down(256, 1024, has_down_attention[3], time_channels, n_groups, dropout_rate, n_heads, is_down[3]) # (-1, 256, 8, 8) -> (-1, 1024, 4, 4)

        # Middle
        self.middle = Middle(1024, 1024, time_channels, n_groups, dropout_rate, n_heads) # (-1, 1024, 4, 4) -> (-1, 1024, 4, 4)

        # Up
        self.up_0 = Up(1024, 256, has_up_attention[0], time_channels, n_groups, dropout_rate, n_heads, is_up[0]) # (-1, 1024, 4, 4) + down_3  -> (-1, 256, 4, 4)
        self.up_1 = Up(256, 128, has_up_attention[1], time_channels, n_groups, dropout_rate, n_heads, is_up[1]) # (-1, 256, 4, 4) + down_2 -> (-1, 128, 8, 8)
        self.up_2 = Up(128, 64, has_up_attention[2], time_channels, n_groups, dropout_rate, n_heads, is_up[2]) # (-1, 128, 8, 8) + down_1 -> (-1, 64, 16, 16)
        self.up_3 = Up(64, 64, has_up_attention[3], time_channels, n_groups, dropout_rate, n_heads, is_up[3]) # (-1, 64, 16, 16) + down_0 -> (-1, 64, 32, 32)

        # Image projection 2
        self.proj2 = nn.Conv2d(proj_channels, img_channels, kernel_size=3, stride=1, padding=1) # Channels : 64 -> 3

    def forward(self, x: torch.Tensor, t: torch.Tensor):
        # Time embdding
        t = self.time_embedding(t)

        # Image projection 1
        x = self.proj1(x)

        # Down
        x, res0 = self.down_0(x, t)
        x, res1 = self.down_1(x, t)
        x, res2 = self.down_2(x, t)
        x, res3 = self.down_3(x, t)

        # Middle
        x = self.middle(x, t)

        # Up
        x = self.up_0(x, t, res3)
        x = self.up_1(x, t, res2)
        x = self.up_2(x, t, res1)
        x = self.up_3(x, t, res0)

        # Image projection 2
        x = self.proj2(x)

        return x
```

## 3. Inference

`DDPM` 클래스를 정의할 때 `p_sample`을 단계적으로 실행해 생성되는 이미지를 확인할 수 있습니다.

먼저 위에서 정의한 `U-Net`과 `DDPM`을 정의해주고 필요한 파라미터들을 함께 정의해줍니다.

```python
# Time schedule T
T = 1000

# Devcie setting
device = torch.device('cuda:0') if torch.cuda.is_available() else torch.device('cpu')
```

```python
# DDPM의 backbone U-Net
backbone = UNet().to(device)

# DDPM
ddpm = DDPM(backbone=backbone, T=T, device=device)
```

위에서 정의한 모델로 sampling을 진행하는 코드를 아래와 같이 작성하면 생성된 이미지를 볼 수 있습니다.

```python
# Number of sampels
n_samples = 10

# Generate Gaussian noises
xt = torch.randn([n_samples, 3, 32, 32], device=device)

# Generate images
for t in reversed(range(T)):
    xt = ddpm.p_sample(xt, xt.new_full((n_samples, ), t, dtype=torch.long))

# Result
result = xt
```

## 4. Interpolation

생성 모델의 가장 큰 장점이라 하면 dataset에 없는 이미지를 만들어 내는 것이 아닐까 싶습니다. Dataset 분포에 없는 이미지를 만들기 위해서 interpolation 방법을 사용할 수 있습니다.

Dataset에 존재하는 두 이미지를 $x^1\_0$과 $x^2\_0$라 하고 두 이미지의 diffusion space에서의 latent를 $x^1\_t$와 $x^2\_t$라 하면 interpolation은 아래와 같이 할 수 있습니다.

1. 먼저 두 이미지 $x^1\_0$와 $x^2\_0$를 diffusion space로 보내겠습니다.

   $$x^1_t \sim~ q(x^1_t|x^1_0)\qquad \&\qquad x^2_t \sim~ q(x^2_t|x^2_0)$$

2. Diffusion space에서 interpolation을 수행하겠습니다.

   $$\bar{x}_t = (1 - \lambda)x^1_t + \lambda x^2_t$$

3. Diffusion space에서 interpolation된 latent를 다시 image space로 보내어 interpolation을 완성합니다.

   $$\bar{x}_0 \sim~ p_\theta(\bar{x}_0|\bar{x}_t)$$

이 때 $t$에 어떤 값을 주는지에 따라 interpolation 결과가 달라지며 $t$가 커질수록 더 다양한 interpolation이 되며 따라서 새로운 결과를 많이 확인할 수 있습니다.

※ [Inference](#3-inference)에서 사용한 변수들을 그대로 사용했습니다.

```python
# Dataset에 있는 이미지라 가정
x1_0 = torch.randn([n_samples, 3, 32, 32], device=device)
x2_0 = torch.randn([n_samples, 3, 32, 32], device=device)

# Setting interpolation_t (Paper - Fig 8.)
interpolation_t = 500
t = torch.full((n_samples,), interpolation_t, device=device)

# Setting lambda
lambda_ = .5

# Interpolation formula (xt: $\bar{x}_t$)
xt = (1 - lambda_) * ddpm.q_sample(x1_0, t) + lambda_ * ddpm.q_sample(x2_0, t)

# Generate interpolation images
for t in reversed(range(T)):
    xt = ddpm.p_sample(xt, xt.new_full((n_samples, ), t, dtype=torch.long))

# Result
result = xt
```

## ※ Reference

- [https://nn.labml.ai/diffusion/ddpm/index.html](https://nn.labml.ai/diffusion/ddpm/index.html)
