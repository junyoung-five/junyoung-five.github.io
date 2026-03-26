---
title: Inverse Dynamics
layout: single
author_profile: true
sidebar:
  nav: study
use_math: true
toc: true
toc_sticky: true
---
### 인버스 다이내믹스 (Inverse Dynamics)

**"이 움직임을 만들기 위해 필요한 모터 토크 구하기"**

- **핵심 원리:** 뉴턴-오일러(Newton-Euler) 또는 라그랑주(Lagrange) 방정식
- **철학:** 전체 시스템의 에너지나 운동 방정식을 세워, 가속도($\ddot{q}$)를 만들기 위한 **일반화된 힘(Generalized Force, 주로 모터 토크 $\tau$)**을 구합니다.
- **특징:**
    - 관심 대상이 주로 **모터(Actuator)**가 내야 하는 힘에 쏠려 있습니다.
    - 전체 시스템의 상태 방정식($M(q)\ddot{q} + C(q, \dot{q}) + G(q) = \tau$)을 푸는 과정입니다.
    - MuJoCo의 `mj_inverse`가 바로 이 계산을 수행하여 `qfrc_inverse`를 뱉어주는 것입니다.