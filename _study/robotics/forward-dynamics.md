---
title: Forward Dynamics
layout: single
author_profile: true
sidebar:
  nav: study
use_math: true
toc: true
toc_sticky: true
---
### **포워드 다이내믹스 (Forward Dynamics)**

**"힘(Force)을 주면 어떻게 움직일까(Kinematics) 예측하기"**

- **핵심 원리:** 뉴턴 제2법칙 (Newton's Second Law) 및 수치 적분 (Numerical Integration)
- **철학:** "원인(힘)이 주어졌을 때, 결과(움직임)를 도출한다." 현재 기구의 상태(위치 $q$, 속도 $\dot{q}$)와 입력되는 힘(모터 토크 $\tau$ 및 외력)을 알 때, 시스템이 어떻게 가속($\ddot{q}$)할지 알아내는 과정입니다.
- **특징:**
    - 수식으로는 인버스 다이내믹스의 식을 뒤집어 가속도에 대해 풉니다: $\ddot{q} = M(q)^{-1}(\tau - C(q, \dot{q}) - G(q))$
    - **시뮬레이션의 심장:** MuJoCo, Drake 같은 물리 엔진이 시간 흐름에 따라 세계를 시뮬레이션할 때 기본적으로 수행하는 작업입니다. (MuJoCo의 `mj_step`, `mj_forward` 함수가 하는 일)
    - **시간($t$)의 개입:** 구한 가속도를 시간에 대해 적분(Integration)하여 다음 스텝의 속도와 위치를 알아냅니다. 이 적분 과정 때문에 필연적으로 시간 지연(Lag)이나 수치적 오차(Drift)가 발생합니다.
    - **복잡한 구속조건 해결:** 앞서 겪으신 문제처럼, 폐루프(Closed-loop)나 접촉(Contact) 같은 구속조건을 유지하면서 움직임을 계산해야 하므로, 물리 엔진들은 스프링-댐퍼 모델(페널티 기법)이나 복잡한 최적화 알고리즘(LCP 등)을 동원합니다. 이 과정에서 미세한 진동(Oscillation)이 발생합니다.