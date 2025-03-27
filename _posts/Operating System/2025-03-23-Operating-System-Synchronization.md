---
layout: post
title: "💾 [Operating System] Synchronization 💾"
author: kjy
date: 2025-03-23 08:00:01 +09:00
categories: [Computer Science, Operating System]
tags: [Computer Science, Operating System]
comments: true
toc: true
math: true
---

## 💾 Synchronization

동기화(Synchronization)은 특정 자원에 접근할 때 한 개의 Process(Thread)만 접근하게 하거나 Process(Thread)를 올바른 순서대로 실행하는 것을 의미합니다.

Process(Thread)의 동기화를 통해 실행 순서와 자원의 일관성을 보장할 수 있습니다.

|                 실행 순서 제어를 위한 동기화                  |                                             상호 배제를 위한 동기화                                              |
| :-----------------------------------------------------------: | :--------------------------------------------------------------------------------------------------------------: |
| 동시에 실행되는 Process(Thread)를 올바른 순서대로 실행하는 것 |                                    공유자원에 동시에 접근하지 못하게 하는 것                                     |
|                                                               | Race Condition이 발생하지 않도록 두 개 이상의 Process(Thread)가 임계 구역에 동시에 접근하지 못하도록 관리하는 것 |

> - **공유자원(Shared Resource)**: 공동으로 이용하는 변수, 파일, 장치 등
> - **Race Condition**: 여러 Process(Thread)가 동시 다발적으로 임계 구역의 코드를 실행하여 생기는 문제
> - **임계 구역(Critical Section)**: 공유자원에 접근하는 코드 중 동시에 실행하면 문제가 발생하는 코드 영역
{: .prompt-tip }

## 💾 Synchronization Techniques

![](../../assets/img/operating system/synchronization_1.png)
_Monitor_

- **Mutex Lock(MUTual Exclusion Lock)**: 하나의 공유자원에 동시에 접근하지 못하도록 만드는 도구
- **Counting Semaphore**: 하나의 공유자원이 아닌 여러 개의 공유자원에 대해 접근을 제어하는 조금 더 일반화된 방식의 동기화 도구
- **Monitor**: 공유자원을 다루는 Interface에 접근하기 위한 Queue를 만들고 Monitor 안에 항상 하나의 Process(Thread)만 들어오도록 하여 상호 배제를 위한 동기화와 조건 변수를 통해 실행 순서 제어를 위한 동기화를 제공하는 동기화
