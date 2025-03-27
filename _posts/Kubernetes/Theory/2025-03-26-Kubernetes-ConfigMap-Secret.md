---
layout: post
title: "☸️ [Kubernetes] ConfigMap & Secret ☸️"
author: kjy
date: 2025-03-27 08:00:03 +09:00
categories: [Kubernetes, Theory]
tags: [Kubernetes]
comments: true
toc: true
math: true
---

## ☸️ ConfigMap & Secret

Container에서 Application을 실행할 때 대표적인 장점 중 하나는 다양한 환경 간 차이를 원천적으로 없앨 수 있다는 점입니다. 그리고 이 때 필요한 Environment Variable과 같은 Configuration Value는 외부에서 Container에 주입해야 합니다.

이처럼 Container에 Configuration Value를 주입하는 데 사용되는 Component가 ConfigMap과 Secret입니다.

ConfigMap과 Secret은 스스로 어떤 기능을 하지 않으며 단지 작은 양의 데이터를 저장하는 것이 목적입니다. 또한 ConfigMap과 Secret은 Pod로 전달되어 사용됩니다.

추가적으로 ConfigMap과 다르게 Secret은 Storage에 저장되지 않고 Memory에 1 Mbyte까지만 저장되며 모든 과정에서 암호화가 적용됩니다.

## ☸️ ConfigMap & Secret 주입

Pod에 ConfigMap과 Secret을 주입하는 방법에는 세 가지가 있습니다.

| 방법 | File에 변경이 있는 경우 |
| :- | :-: |
| • Container의 Environment Variable에 ConfigMap과 Secret을 직접 주입하는 방법 |  |
| • Container의 Environment Variable에 ConfigMap과 Secret을 File로 주입하는 방법 | 적용이 안됨 |
| • ConfigMap과 Secret을 Volume 형태로 정의하고 Container에 Mount하는 방법 | 적용이 됨 |