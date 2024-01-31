---
layout: post
title: "[Generative Model] DDIM : Denoising Diffusion Implicit Models"
author: kjy
date: 2024-01-31 13:00:00 +09:00
categories: [Deep Learning, Paper Reading]
tags: [Deep Learning, Paper Reading, Computer Vision, Generative Model]
comments: true
toc: true
math: true
---

해당 포스트에서는 [DDPM : Denoising Diffusion Implicit Models](https://arxiv.org/abs/2010.02502) 논문을 함께 읽어가도록 하겠습니다. 문장마다 해석을 하기 보다는 각 문단에서 중요한 부분을 요약하는 식으로 진행하겠습니다.

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

진행중입니다....
