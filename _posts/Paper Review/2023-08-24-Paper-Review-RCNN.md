---
layout: post
title: "[Object Detection] R-CNN"
author: kjy
date: 2023-08-24 12:20:00 +09:00
categories: [Deep Learning, Paper Review]
tags: [Deep Learning, Paper Review, Computer Vision, Object Detection]
comments: true
toc: true
math: true
---

해당 포스트에서는 R-CNN 논문을 리뷰 하기 보다는 이후 Fast R-CNN, Fater R-CNN의 기초가 되는 R-CNN에 대해 간략하게 알아보도록 하겠습니다.

---

## 1. Pipeline

R-CNN의 pipeline은 다음과 같습니다.

> ① Input image  
> ② Selective search를 사용해 2,000 개의 RoI 추출  
> ③ 추출된 RoI의 사이즈를 CNN input size에 맞도록 조절  
> ④ 사이즈가 조절된 RoI를 CNN에 넣어 feature를 추출  
> ⑤ SVM을 사용해 RoI의 class를 예측  
> ⑥ RoI의 좌표값에 대해 regression 진행

R-CNN의 Pipeline을 간략하게 알아봤습니다. 이후 하나씩 자세히 알아보겠습니다.

---

## 2. Selective Search

RoI를 추출하는 방법에는 sliding window 방식과 selective search가 있습니다. 그러나 sliding window 방법은 모든 경우의 수를 다 고려하기 때문에 시간이 오래 걸린다는 굉장히 큰 단점이 존재합니다. 그렇기 때문에 R-CNN에서는 selective search를 통해 RoI를 추출 했습니다.

그러나 selective search는 딥러닝 방법이 아닌 사람이 설정한 알고리즘을 통해 RoI가 추출될 뿐만 아니라 이 또한 시간이 오래 걸리고 end-to-end 학습을 하는데 있어서 걸림돌이 되었기 때문에 이후 나오는 모델들에서는 이를 대체하는 다른 방법들을 사용했습니다.

그렇기 때문에 해당 포스트에서는 정말 간단하게만 알아보고 넘어가도록 하겠습니다.

Selective search algorithm은 이미지의 색깔, 주변 pixel 값들의 변화량, region들의 size, candidate bounding box와의 size 차이를 통해 유사성을 통해 유사성이 높은 pixel들을 묶어 나가는 hierarchical grouping algorithm 입니다.

---

## 3. Warped RoI

R-CNN에서 사용한 convolutional network는 AlexNet 입니다. AlexNet의 경우 마지막 layer에 fully-connected layer가 존재하기 때문에 input size가 고정되어 있습니다. 따라서 ImageNet으로부터 학습된 AlexNet의 weight를 사용하기 위해서는 image의 input size를 AlexNet의 image input size인 (224, 224, 3)으로 맞춰줘야 합니다.

이러한 이유에서 selective search를 통해 추출된 RoI를 (24, 224, 3)으로 조정했습니다.

---

## 4. Compute CNN Features

다음은 CNN에 RoI를 넣어 RoI의 feature들을 뽑아내는 과정입니다. ⭐ 그 전에 해당 데이터셋에 맞도록 fine tuning을 진행하는 과정이 있습니다. 지금 부터 나오는 내용들이 이후 비슷하게 반복이 되기 때문에 헷갈리시면 안됩니다. ⭐

이전에 selective search를 통해 찾은 RoI와 ground truth를 비교해서 IoU가 0.5보다 크면 positive samples, IoU가 0.5보다 작으면 negative samples로 하고 ground truth를 positive로 합니다. Positve와 negative로 나뉜 dataset을 하나의 batch로 구성할 때 positive samples 32개, negative samples 96개로 구성합니다.

AlexNet을 fine-tuning 하기 위해 사용한 dataset을 정리해보면 다음과 같습니다.

✍️ Positive Samples = {Selective Search ROI의 IoU > 0.5, Ground Truth}  
✍️ Negative Samples = {Selective Search ROI의 IoU < 0.5}

또한 AlexNet의 마지막 layer를 (Class 개수 + 배경)개의 node를 가지도록 바꾼 후 fine tuning을 진행합니다.

---

## 5. Linear SVM Classifier

위에서 AlexNet을 fine tuning 하기 위해 dataset을 구성 했던 것 처럼 Linear SVM Classifier를 학습 시키기 위해 dataset을 구성해야 합니다. Ground Truth만을 positive samples로 하고 이전에 selective search를 통해 찾은 RoI와 ground truth를 비교해서 IoU가 0.3보다 작은 것을 negative samples로 하고 나머지는 무시하여 Linear SVM Classifier를 학습할 dataset을 구성합니다. Postive와 negative로 나뉜 dataset을 각각 32개, 96개를 하나의 batch로 구성하였으며 Hard negative mining을 통해 이후 batch들의 negative sample들을 구성했습니다.

Linear SVM Classifier를 학습하기 위해 사용한 dataset을 정리해보면 다음과 같습니다.

✍️ Positive Samples = {Ground Truth}  
✍️ Negative Samples = {Selective Search ROI의 IoU < 0.3}

Linear SVM Classifier는 이전에 fine tuning한 AlexNet의 마지막 layer인 4096 차원의 feature vector를 사용해 (Class 개수 + 배경)개의 SVM Classifier를 학습하게 됩니다.

---

## 6. Bounding Box Regression

이전에 학습하기 전에 dataset 구성하는 방법이 모두 달랐던 것을 볼 수 있었습니다. 이전과 마찬가지로 Bounding box regression을 학습하기 위한 dataset을 따로 구성합니다. selective search를 통해 찾은 RoI와 ground truth를 비교해서 IoU가 0.6보다 큰 sample들을 positive samples로 하여 해당 samples들만 사용합니다.

Bounding Box Regression를 학습하기 위해 사용한 dataset을 정리해보면 다음과 같습니다.

✍️ Positive Samples = {Selective Search ROI의 IoU > 0.6, Ground Truth}  
✍️ Negative Samples = {}

Bounding Box Regression을 학습할 때는 AlexNet의 $Pool_5$의 feature map을 사용합니다. $Pool_5$와 learnable vector를 행렬곱해서 bounding box의 좌표값을 예측합니다. MSE Loss를 통해 예측된 좌표값을 조정해 갑니다.

---

## 7. 정리

End-to-End 학습이 안되기 때문에 학습 순서부터 학습을 하기 위한 dataset을 구성하는 것까지 학습 과정이 꽤나 복잡합니다. 학습 순서를 정리해보면 CNN Fine-tuning, SVM, Bounding Box Regression 순서가 됩니다. 이후 나온 Object detection 논문들이 복잡한 이 과정들을 어떻게 해결해 나가는지 살펴보아도 좋은 공부가 될 것 같습니다.

---

## ※ Reference

[https://velog.io/@woojinn8/Object-Detection-1.-R-CNN-리뷰](https://velog.io/@woojinn8/Object-Detection-1.-R-CNN-리뷰)
