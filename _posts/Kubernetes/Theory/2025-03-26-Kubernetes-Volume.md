---
layout: post
title: "☸️ [Kubernetes] Volume ☸️"
author: kjy
date: 2025-03-27 08:00:02 +09:00
categories: [Kubernetes, Theory]
tags: [Kubernetes]
comments: true
toc: true
math: true
---

## ☸️ Volume

Volume은 Container File System을 구성하는 수단 중 하나입니다.

Volume을 정의하는 방법에 따라서 Volume에 저장되는 데이터들의 생애주기가 다음과 같이 달라집니다.

| Type | 특징 |
| :-: | :- |
| • [EmptyDir](#️-emptydir) | Pod와 생애주기를 함께 하는 Volume |
| • [HostPath](#️-hostpath) | Node와 생애주기는 함께 하는 Volume |
| • [PVC / PV](#️-pvc--pv) | Cluster에 있는 모든 Component에서 접근이 가능한 Volume |

## ☸️ EmptyDir

EmptyDir은 Pod 생성시 만들어지고 삭제시 없어지는 Volume입니다.

## ☸️ HostPath

HostPath는 Worker Node의 Storage(HDD, SSD 등)를 가리키는 Volume입니다. 따라서 HostPath는 Worker Node의 생애주기와 같습니다.

HostPath의 단점으로는 만약 HostPath를 Mount하여 사용하는 Pod가 삭제되고 생성될 때 이전 Worker Node와 다른 Worker Node에 재생성되는 경우 HostPath의 수정없이 사용할 수 없다는 문제가 있습니다.

또한 Worker Node의 Storage를 직접 가리키는 Volume이기 때문에 보안적으로 취약할 수 밖에 없습니다.

## ☸️ PVC / PV

PV(Persistent Volume)는 Cluster 내부 또는 외부 Storage(Volume)와 연결하여 Cluster 내부에 있는 모든 Component에서 접근이 가능하도록 하는 Volume입니다.

그러나 Cluster 내부의 모든 Component에서 직접 PV에 접근하지는 못하고 PVC(Persistent Volume Claim)을 통해 접근합니다.

PV를 생성하는 방법에는 다음 두 가지 방법이 있습니다.

| PV 생성 방법 | 특징 |
| :- | :- |
| • Static Provisioning | • PV를 직접 생성하여 PVC와 연결하는 방식 <br/> • PVC의 요구 사항을 만족하는 PV가 생성될 때까지 대기 상태 |
| • Dynamic Provisioning | • PVC만 생성하면 StorageClass라는 Component를 사용해 Cluster 내부에 PV를 생성해 주는 방식 <br/> • K8s Platform에 따라 동작 방식이 다름 |

그리고 PV의 Status에는 다음과 같은 것들이 있습니다.

| Status | 설명 |
| :- | :- |
| • Available | 최초 PV가 만들어졌을 때의 상태 |
| • Bound | PVC와 연결이된 상태 |
| • Released | PVC가 삭제되어 PV와 연결이 끊어진 상태 |
| • Failed | PV에 연결된 Storage에 문제가 생겨 Storage와 연결이 끊어진 상태 |

PV의 Status가 Released일 때 ReclaimPolicy에 따라 PV의 상태가 다음과 같이 달라집니다.

| ReclaimPolicy | 바뀐 Status | 특징 |
| :- | :-: | :- |
| • Retain | Released | • PV를 만들 때 ReclaimPolicy를 설정하지 않은 경우 default <br/> • PV를 다른 PVC에 연결하여 재사용 불가 <br/> • Storage의 데이터는 보존 |
| • Delete |  | • StorageClass를 사용해 PV가 만들어진 경우 default <br/> • PV도 함께 삭제 <br/> • Storage에 따라 데이터가 삭제 여부가 달라짐 |
| • Recyle | Available | • PV를 PVC에 연결하여 재사용 가능 <br/> • Storage의 데이터 삭제 |