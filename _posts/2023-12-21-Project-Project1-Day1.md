---
layout: post
title: "[웹툰을 원작으로 제작된 드라마 및 영화의 주연 배우 추천] Day 1 - 졸업논문 살펴보기 & 관련 논문 리서치 & 앞으로의 방향성"
author: kjy
date: 2023-12-21 14:00:00 +9:00
categories: [Project, 웹툰을 원작으로 제작된 드라마 및 영화의 주연 배우 추천]
tags: [Deep Learning, Project]
comments: true
toc: true
math: true
---

## 1. 졸업 논문 살펴보기

### 1.1 해당 프로젝트의 필요성

1. 최근 웹툰 시장이 국내외로 빠르게 성장
2. 이와 함께 다양한 작품이 연재되고 있으며 다양한 작품들이 드라마나 영화로 제작
3. 드라마나 영화로 제작되는 경우 주요 관심사는 주인공 역할에 어떤 배우가 캐스팅 될 지

### 1.2 해당 프로젝트의 목표

1. 배우를 캐스팅할 때 후보군을 추려내기 위해 배우 이미지와 웹툰 주인공 이미지의 비슷함 정도를 측정
2. 웹툰 이미지는 데포르메가 존재하고 색감(style) 또한 현실과는 차이가 있기 때문에 그 차이를 줄여서 비슷함 정도를 측정
3. Style 차이를 줄이기 위해 style transfer를 수행

### 1.3 사용한 모델

1. [StyleGAN2](https://arxiv.org/abs/1912.04958)

2. [Toonify](https://arxiv.org/abs/2010.05334)

## 2. 관련 논문 리서치

### 2.1 WebtoonMe

[Deview 2023에서 발표한 WebtoonMe 개발기](https://deview.kr/2023/sessions/565)를 참고하여 트렌드와 관련 논문들을 조사했습니다.

1. Trend

   - 기존에는 성능이 좋은 모델을 만드는 것이 중요했다면 최근에는 이 모델을 활용하여 task에 적합한 고품질의 데이터를 생성한 후 가볍고 학습이 쉬운 별도의 network를 학습시키는 것이 트렌드라고 합니다.
   - 사실 data-centric이 굉장히 중요한 것은 알고 있었지만 생성모델을 통해 고품질의 데이터를 생성한 후 별도의 network를 학습시키는 것은 처음 알게 된 사실입니다. 이 부분에 대해서도 공부가 필요해보입니다.
   - 고품질의 데이터 생성
     - [StyleGAN](https://arxiv.org/abs/1812.04948)
   - 가볍고 학습이 쉬운 별도의 network
     - [DCT-Net: Domain-Calibrated Translation for Portrait Stylization](https://arxiv.org/abs/2207.02426)
     - [Production-Ready Face Re-Aging for Visual Effects](https://studios.disneyresearch.com/app/uploads/2022/10/Production-Ready-Face-Re-Aging-for-Visual-Effects.pdf)

2. WebtoonMe를 개발하면서 발표한 논문

   - [WebtoonMe](https://arxiv.org/abs/2210.10335)
   - [Cross-Domain Style Mixing for Face Cartoonization](https://arxiv.org/abs/2205.12450)
   - [Interactive Cartoonization with Controllable Perceptual Factors](https://arxiv.org/abs/2212.09555)

3. WebtoonMe에서 활용한 SoTA 모델

   - [JoJoGAN: One Shot Face Stylization](https://arxiv.org/abs/2112.11641)
   - [DualStyleGAN](https://arxiv.org/abs/2203.13248)
   - [Stable Diffusion](https://arxiv.org/abs/2112.10752)

4. WebtoonMe에서 활용한 최신 데이터셋

   - [FFHQ](https://github.com/NVlabs/ffhq-dataset)
   - [LSUN](https://github.com/fyu/lsun?tab=readme-ov-file)
   - [DeepFashion-MultiModal](https://github.com/yumingj/DeepFashion-MultiModal)
   - [SHHQ](https://stylegan-human.github.io/data.html)

### 2.2 SNOW AI FILTER

[Deview 2023에서 발표한 SNOW AI FILTER : 나인 듯 나 같지 않은 나보다 예쁜 나](https://deview.kr/2023/sessions/556)를 참고하여 관련 논문들을 조사했습니다.

1. Dataset에 따른 분류

   - Unpair Dataset
     - [U-GAT-IT](https://arxiv.org/abs/1907.10830)
     - [White-box cartoonization](https://openaccess.thecvf.com/content_CVPR_2020/papers/Wang_Learning_to_Cartoonize_Using_White-Box_Cartoon_Representations_CVPR_2020_paper.pdf)
     - [CycleGAN](https://arxiv.org/abs/1703.10593)
   - Pair Dataset
     - [Pix2Pix based approach](https://arxiv.org/abs/1611.07004)

2. Dataset 생성

   - 당연하게도 Unpair dataset에 비해서 pair dataset이 더 좋은 성능을 보이지만 pair dataset을 생성해야 한다는 단점이 존재합니다.
   - Pair dataset을 효율적으로 생성하기 위해 Toonify를 사용했는데 제가 졸업논문에서 사용한 방식과는 차이가 있어 보입니다. Toonify에 대해 공부를 다시 한 번 해볼 필요가 있을 것 같습니다.
   - 최근에는 diffusion을 사용해 더 퀄리티 있는 dataset을 생성하고 있으므로 이 부분도 한 번 살펴볼 필요가 있을 것 같습니다.

## 3. 앞으로의 방향성

해당 프로젝트의 완성도를 높이는 것도 중요하지만 서빙을 직접 해보고자 합니다. 네이버 부스트캠프를 통해 간접적으로 서빙을 경험하기는 했지만 직접 해보지 못한 것이 아쉬움으로 남아 이번 프로젝트를 진행하면서 함께 진행하고자 합니다.

가장 좋은 예시가 WebtoonMe인 것 같아 WebtoonMe의 개발 과정을 따라해 보고자 합니다.

### 3.1 WebtoonMe

혼자 진행하는 프로젝트이다 보니 우선은 최대한 간단하게 만드는 것이 먼저라고 생각합니다. 그래서 WebtoonMe와 마찬가지로 빠르게 한 바퀴를 돌리고자 합니다.

WebtoonMe의 개발과정을 한 번 정리해 보겠습니다.

|           | Step 1              | Step 2         | Step 2                 |
| :-------- | :------------------ | :------------- | :--------------------- |
| Training  | fast.ai             | PyTorch        | PyTorch & Hugging Face |
| Tracking  | METAFLOW            | CLEAR ML       | Weights & Bias         |
| Serving   | Hugging Face Spaces | RAY & TensorRT | RAY & TensorRT         |
| Front-end | gradio              | Streamlit      | 자체 APP 개발          |

위에서 처음들어보는 것들도 있고 알고 있는 것도 있는데 하나씩 알아가보면서 프로젝트를 진행하겠습니다.

## 4. 다음 계획

➡️ 가장 먼저 프로토타입을 완성하려고 합니다.  
➡️ 프로토타입을 완성하면 위에 나온 논문들을 우선적으로 읽어보고자 합니다. ([Overview](https://jjjuuuun.github.io/posts/Project-Project1-Day0/)에 논문들을 정리했습니다.)  
➡️ 공부하고 있는 논문이 데이터 생성이 가능한 논문이라면 데이터 생성도 해보고자 합니다. 나중에 데이터들의 비교를 위해 기록을 잘 해야 할 것 같습니다.

## 5. 알게된 점

🤔 고품질의 데이터 생성을 생성모델을 통해 한다는 점  
🤔 Generative Model에서도 data-centric이 중요시 되고 있다는 점  
🤔 Stable diffusion의 파급력이 생각보다 더 크다는 점

## ※ Reference

- NAVER Deview 2023
  - [WebtoonMe 개발기](https://deview.kr/2023/sessions/565)
  - [SNOW AI FILTER : 나인 듯 나 같지 않은 나보다 예쁜 나](https://deview.kr/2023/sessions/556)
