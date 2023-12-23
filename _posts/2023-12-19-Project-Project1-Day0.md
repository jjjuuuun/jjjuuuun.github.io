---
layout: post
title: "[웹툰을 원작으로 제작된 드라마 및 영화의 주연 배우 추천] Day 0 - Project를 시작하면서"
author: kjy
date: 2023-12-19 16:46:00 +9:00
categories: [Project, 웹툰을 원작으로 제작된 드라마 및 영화의 주연 배우 추천]
tags: [Deep Learning, Project]
comments: true
toc: true
math: true
---

딥러닝을 시작한 이후 가장 관심이 있었던 분야는 이미지 생성 모델입니다. 그 중에서도 style transfer와 관련된 분야에 관심이 있었습니다. 하지만 수학적인 한계 그리고 gpu와 같은 리소스의 한계와 같은 문제들 때문에 깊게 공부해보지 못한 것이 계속해서 후회로 남아있었습니다. 후회를 계속 남기기 싫어 제대로된 공부를 진행하고자 합니다. 무작정 논문을 읽는 것보다는 프로젝트를 진행하며 공부를 해가는 것이 좋다고 생각해 해당 프로젝트를 진행하게 되었습니다. 여전히 gpu 문제가 해결되지는 않았지만 우선적으로 해당 분야의 트렌드를 쭉 공부한 후 고민하고자 합니다.

해당 프로젝트에서는 졸업작품으로 진행했었던 ["웹툰을 원작으로 제작된 드라마 및 영화의 주연 배우 추천"](https://drive.google.com/file/d/1jHqIhJNpUh4vigUMUpX0TQXUwvr1itRV/view?usp=sharing)을 조금 더 develop 해보고자 합니다.

추가적으로 해당 포스트를 통해 진행상황을 계속해서 올리도록 하겠습니다.

<details>
    <summary><a href="https://jjjuuuun.github.io/posts/Project-Project1-Day0/">Day0 (2023.12.19)</a></summary>
    1. Project 주제 선정
</details>

<details>
<summary><a href="https://jjjuuuun.github.io/posts/Project-Project1-Day1/">Day1 (2023.12.21)</a></summary>

1. <a href="https://drive.google.com/file/d/1jHqIhJNpUh4vigUMUpX0TQXUwvr1itRV/view?usp=sharing">졸업논문</a> 다시 한번 살펴보기<br>
2. <a href="https://deview.kr/2023">NAVER Deview 2023</a><br>
   &nbsp; &nbsp; ➡️ <a href="https://deview.kr/2023/sessions/565">WebtoonMe 개발기</a><br>
   &nbsp; &nbsp; ➡️ <a href="https://deview.kr/2023/sessions/556">SNOW AI FILTER : 나인 듯 나 같지 않은 나보다 예쁜 나</a><br>
3. 공부해야할 논문 선정<br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/1912.04958">StyleGAN2</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/2010.05334">Toonify</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/2207.02426">DCT-Net: Domain-Calibrated Translation for Portrait Stylization</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://studios.disneyresearch.com/app/uploads/2022/10/Production-Ready-Face-Re-Aging-for-Visual-Effects.pdf">Production-Ready Face Re-Aging for Visual Effects</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/2210.10335">WebtoonMe</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/2205.12450">Cross-Domain Style Mixing for Face Cartoonization</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/2212.09555">Interactive Cartoonization with Controllable Perceptual Factors</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/2112.11641">JoJoGAN: One Shot Face Stylization</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/2203.13248">DualStyleGAN</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/2112.10752">Stable Diffusion</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/1907.10830">U-GAT-IT</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://openaccess.thecvf.com/content_CVPR_2020/papers/Wang_Learning_to_Cartoonize_Using_White-Box_Cartoon_Representations_CVPR_2020_paper.pdf">White-box cartoonization</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/1703.10593">CycleGAN</a> <br>
   &nbsp; &nbsp; ➡️ <a href="https://arxiv.org/abs/1611.07004">Pix2Pix based approach</a> <br>
4. 앞으로의 계획 짜기<br>
   &nbsp; &nbsp; ➡️ 빠른 프로토타입 <br>
   &nbsp; &nbsp; ➡️ 논문 공부하면서 고품질의 데이터 생성 <br>

</details>

<details>
    <summary><a href="https://jjjuuuun.github.io/posts/Project-Project1-Day2/">Day2 (2023.12.23)</a></summary>
    1. About fast.ai
    2. Single-label Classification using fast.ai
</details>
