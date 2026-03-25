---
title: "Groove Joint Modeling"
layout: single
author_profile: true
sidebar:
  nav: "study"
use_math: true
toc: true
toc_sticky: true
---

> Groove Joint의 모델링 방법 개발 및 Kinematics 검증에 대한 연구 정리

## 연구 배경

- Groove Joint가 실제 기구 설계에 있어 쓰임이 많다.
  - Groove Joint란 기본적으로 RP(or PR) joint에서 그 궤적이 곡선으로 변형된 joint를 의미한다.
- 현재 메커니즘 데이터 생성기에선 R, P Joint만 고려되고 있다.

## 연구 목표

Groove Joint의 모델링 방법 개발 및 그에 따른 Kinematics 검증을 목표로 한다.

---

## Groove Joint Modeling

### Constraint Equation (Position)

<details open markdown="1">
<summary>상세 내용</summary>

$$\Phi_G(\mathbf{q}_i(t)) = \hat{\mathbf{n}}(s^*(t))^\top \cdot \left( \mathbf{p}(t) - \mathbf{C}(s^*(t)) \right) = 0$$

- $\mathbf{q}_i(t) = \begin{bmatrix} x_i(t) \\ y_i(t) \\ \phi_i(t) \end{bmatrix}$ : 평면에서 강체의 상태 정의를 위한 일반화 좌표

- 링크 $i$ 위에 있는 groove joint의 현재 위치:

$$\mathbf{p}(t) = \mathbf{r}_i(t) + \mathbf{R}(\phi_i(t)) \, \mathbf{s}^{(i,j)}_{\text{local}}$$

  - $\mathbf{r}_i = \begin{bmatrix} x_i(t) \\ y_i(t) \end{bmatrix}$ : 링크 $i$의 무게중심(COM) 위치
  - $\phi_i$ : 링크 $i$의 회전 각
  - $\mathbf{R}(\phi_i(t)) = \begin{bmatrix} \cos\phi_i(t) & -\sin\phi_i(t) \\ \sin\phi_i(t) & \cos\phi_i(t) \end{bmatrix}$ (회전 행렬)
  - $\mathbf{s}^{(i,j)}_{\text{local}}$ : COM에서 조인트 $j$까지의 body-fixed 벡터 (상수)

- 파라미터 $s$에 대응되는 곡선 레일 위의 점:

$$\mathbf{C}(s) = \begin{bmatrix} C_x(s) \\ C_y(s) \end{bmatrix}$$

  - CubicSpline 또는 B-Spline으로 정의
  - $s$ : arc-length 파라미터, $[0, 1]$ 범위로 정규화

- $s^*$ : 레일 위에서 $\mathbf{p}$에 가장 가까운 점

$$\left(\mathbf{p}(t) - \mathbf{C}(s^*(t))\right) \cdot \mathbf{C}'(s^*(t)) = 0$$

- 단위 법선 벡터:

$$\hat{\mathbf{n}}(s^*(t)) = \frac{1}{\|\mathbf{C}'(s^*(t))\|} \begin{bmatrix} -C'_y(s^*(t)) \\ C'_x(s^*(t)) \end{bmatrix}$$

**물리적 의미**: 조인트 점 $\mathbf{p}$가 레일 위 최근접점 $\mathbf{C}(s^*)$로부터 법선 방향으로 떨어져 있지 않다 $\Leftrightarrow$ 점이 곡선 위에 존재한다.

</details>

---

### Jacobian (Velocity Equation)

<details open markdown="1">
<summary>상세 내용</summary>

$$\mathbf{J}_G = \frac{\partial \Phi_G}{\partial \mathbf{q}} = \begin{bmatrix} \dfrac{\partial \Phi_G}{\partial \mathbf{r}_i} & \dfrac{\partial \Phi_G}{\partial \phi_i} \end{bmatrix}$$

#### $s^*$를 상수로 취급하는 근거

엄밀하게는 $s$도 $\mathbf{q}$의 함수이므로 $ds/d\mathbf{q}$ 보정항이 존재한다:

$$\frac{\partial \Phi_G}{\partial \mathbf{q}} = \left. \frac{\partial \Phi_G}{\partial \mathbf{q}} \right|_{s^{*}=\text{const}} + \underbrace{\frac{\partial \Phi_G}{\partial s^*}}_{\text{보정항}} \frac{d s^*}{d \mathbf{q}}$$

$$\frac{\partial \Phi_G}{\partial s^{*}} = \underbrace{\left. \frac{d \hat{\mathbf{n}}}{d s^{*}} \right|^{\!\top} \cdot \left( \mathbf{p} - \mathbf{C}(s^{*}) \right)}_{\text{A}} + \underbrace{\hat{\mathbf{n}}(s^{*})^{\top} \cdot \left( -\mathbf{C}'(s^{*}) \right)}_{\text{B}}$$

- **항 B = 0** (항상 성립): 법선 벡터와 접선 벡터는 항상 직교

$$\hat{\mathbf{n}}(s^{*})^{\top} \cdot \mathbf{C}'(s^{*}) = 0 \quad (\hat{\mathbf{n}} \perp \mathbf{C}')$$

- **항 A = 0** (구속 만족 상태 $\Phi_G = 0$에서 성립):
  - 직교 조건에 의해 $(\mathbf{p} - \mathbf{C}(s^*))$는 접선에 수직 → 법선 방향 성분만 존재
  - $\Phi_G = 0$에 의해 법선 방향 성분도 0

$$A = 0, \quad B = 0 \quad \Rightarrow \quad \frac{\partial \Phi_G}{\partial s^*} = 0 \quad (\text{on constraint surface})$$

**따라서 구속면 위에서 보정항이 소멸 → $s^*$를 상수로 취급해도 정확하다.**

#### Jacobian 유도

- 위치에 대한 편미분:

$$\frac{\partial \Phi_G}{\partial \mathbf{r}_i} = \hat{\mathbf{n}}(s^*)^\top \quad \left(\because \ \frac{\partial \mathbf{p}}{\partial \mathbf{r}_i} = \mathbf{I}_{2 \times 2}\right)$$

- $\frac{\partial \mathbf{R}(\phi_i)}{\partial \phi_i} = \mathbf{E} \, \mathbf{R}(\phi_i)$, where $\mathbf{E} = \begin{bmatrix} 0 & -1 \\ 1 & 0 \end{bmatrix}$ (90도 회전 행렬)

- 회전된 body-fixed 벡터:

$$\bar{\mathbf{s}}_i = \mathbf{R}(\phi_i) \, \mathbf{s}^{(i,j)}_{\text{local}}$$

- $\frac{\partial \Phi_G}{\partial \phi_i} = \hat{\mathbf{n}}(s^*)^\top \, \mathbf{E} \, \bar{\mathbf{s}}_i$

**최종 Jacobian:**

$$\mathbf{J}_G(t) = \begin{bmatrix} \hat{n}_x(t) & \hat{n}_y(t) & \hat{\mathbf{n}}(s^*)^\top \mathbf{E} \bar{\mathbf{s}}_i(t) \end{bmatrix}$$

#### 속도 구속 방정식

$$\frac{d \Phi_G}{d t} = \mathbf{J}_G \, \dot{\mathbf{q}} = \hat{\mathbf{n}}(s^*)^\top \dot{\mathbf{p}} = 0$$

</details>

---

### Acceleration Equation

<details open markdown="1">
<summary>상세 내용</summary>

$$\ddot{\mathbf{p}} = \ddot{\mathbf{r}}_i + \alpha_i \, \mathbf{E} \, \bar{\mathbf{s}}_i - \omega_i^2 \, \bar{\mathbf{s}}_i$$

where $\alpha_i(t) = \ddot{\phi}_i(t)$

가속도 구속:

$$\mathbf{J}_G \, \ddot{\mathbf{q}} = -\gamma_G$$

$$\gamma_G = \underbrace{\hat{\mathbf{n}}(s^*)^\top \left( -\omega_i^2 \, \bar{\mathbf{s}}_i \right)}_{\gamma_{\text{centripetal}}} + \underbrace{\kappa(s^*) \, \dot{s}^{2} \, \|\mathbf{C}'(s^*)\|^2}_{\gamma_{\text{curvature}}}$$

where 곡률:

$$\kappa(s^*) = \frac{C'_x \, C''_y - C'_y \, C''_x}{\left( C'^2_x + C'^2_y \right)^{3/2}}$$

</details>

---

## Kinematics 검증

솔리드웍스에서 72 step 점에 대하여 J1, J2 조인트의 Pos 값을 추출하여 비교.

<!-- 이미지 추가 예정: groove_solver_vs_solidworks.gif -->

- J1, J2 위치 오차: 소수점 두 자리 수준
- 속도: 노이즈 존재하지만 경향성 일치 확인
- 가속도: 유효 숫자 정밀도에 따른 노이즈 증폭으로 비교 불가
  - 현재 groove joint 수학적 모델링이 Kinematics 해석에 있어 정합성을 보인다고 결론

---

## 기구 메커니즘 최적 설계

### 곡선 정의: B-Spline 매개변수화

<details open markdown="1">
<summary>상세 내용</summary>

CubicSpline 기반은 과도하게 많은 설계 변수가 필요 → B-Spline으로 교체.

$$\mathbf{C}(s) = \sum_{i=0}^{n} N_{i,p}(s) \cdot P_i$$

- 제어점 $P_i$ : 곡선의 형상을 결정하는 점 ($n+1$ 개)
- 차수 $p$ : 곡선의 다항식 차수 (cubic = 3)
- Knot vector $T$ : 파라미터 구간을 나눠주는 비감소 수열
  - knot 개수 = 제어점 수 + 차수 + 1
  - Clamped: 양 끝 $p+1$번 반복 → 곡선이 첫/끝 제어점을 정확히 통과

</details>

### 최적 설계 정식화

<details open markdown="1">
<summary>상세 내용</summary>

**목적함수:**

$$\min_{\mathbf{x}} \quad \left( f_1(\mathbf{x}),\; f_2(\mathbf{x}) \right)$$

1. **E.E 궤적 추종 오차 최소화**

$$f_1(\mathbf{x}) = \sqrt{ \frac{1}{S} \sum_{t=1}^{S} \left\| \mathbf{p}_{ee}(t;\,\mathbf{x}) - \mathbf{p}_{target}(t) \right\|^2 }$$

2. **최대 Groove 압력각 최소화**

$$f_2(\mathbf{x}) = \max_{t=1,\ldots,S} \left[90° - \arccos \left( \frac{ \left| \mathbf{d}(t) \cdot \boldsymbol{\tau}(t) \right| }{\left\| \mathbf{d}(t) \right\| \cdot \left\| \boldsymbol{\tau}(t) \right\| } \right) \right]$$

**설계변수:**

$$\mathbf{x} = \Big[\underbrace{L_1,\; \theta_0,\; L_2,\; L_3,\; \beta,\; \varphi}_{\text{기구 형상 (6)}},\;\; \underbrace{c_{x,1},\, c_{y,1},\, \ldots,\, c_{x,9},\, c_{y,9}}_{\text{groove 제어점 (18)}}\Big]$$

**제약조건:**

| 제약 | 내용 |
|:---|:---|
| 전 구간 조립/구동 가능 | $\|\boldsymbol{\Phi_G}(\mathbf{q}(t),\, t)\| < \epsilon_{NR} \quad \forall\, t$ |
| 링크 길이 | $L_{min} \leq \|p_{Ji} - p_{Jk}\| \leq L_{max}$ |
| Rail 자기 교차 금지 | 비인접 선분 쌍 교차 검사 (CCW 판별법) |
| Rail 공간 범위 | J0 중심 500x500mm 경계 내 |
| J0 위치 고정 | $\mathbf{p}_{J0} = \mathbf{p}_{J0}^{\,ref}$ |

</details>

### 최적화 전략 (3-Stage)

<details open markdown="1">
<summary>상세 내용</summary>

| Stage | Method | 대상 | 목적 |
|:---|:---|:---|:---|
| 1 | Nelder-Mead | J1, J2 only | 저차원에서 대략적인 조인트 위치 결정 |
| 2 | CMA-ES | 전체 24개 | 고차원 전역 탐색 |
| 3 | Nelder-Mead | 전체 24개 | CMA-ES 결과 기반 로컬 미세 조정 |

</details>

### 후처리 (Refinement)

<details open markdown="1">
<summary>상세 내용</summary>

**Pass-1: E.E + Mechanism Smoothing**

$$f(\mathbf{x}) = w_{ee} \cdot \text{std}(\kappa_{EE}) + w_{rail} \cdot \text{std}(\kappa_{rail}) + w_{rmse} \cdot \max(\text{RMSE} - \text{RMSE}_{orig} - \text{tol},\, 0) + \text{oob}_{penalty}$$

| 항목 | Weight | 역할 |
|:---|:---|:---|
| EE 곡률 std | 1.0 | EE 궤적의 곡률 균일화 |
| Rail 곡률 std | 0.5 | Groove rail 곡률 균일화 |
| RMSE 초과 페널티 | 50.0 | 원본 대비 RMSE 악화를 0.3mm 이내로 제한 |
| Out-of-bounds 페널티 | 10.0 | 변수가 bounds 밖으로 나가는 것 억제 |

**Pass-2: Groove Rail Smoothing**

$$f(\mathbf{c}) = \text{mean}(\text{deviation}^2) + w_{fair} \cdot \text{std}(\kappa_{rail})$$

기구 파라미터(조인트 위치)는 고정, rail 곡선만 독립적으로 smoothing.

</details>

---

## 결론

### Groove Joint 정의
RP, PR 조인트와 같이 두 조인트 간 연결 관계에 있어 링크의 길이가 0이며, 곡선 형태의 레일을 추종하며 움직이는 새로운 조인트 타입.

### Groove Joint 구속 방정식

$$\Phi_G(\mathbf{q}_i(t)) = \hat{\mathbf{n}}(s^*(t))^\top \cdot \left( \mathbf{p}(t) - \mathbf{B}(s^*(t)) \right) = 0$$

### Kinematics Solver 구속 방정식 체계

| Type | Equation | # of eq |
|:---|:---|:---|
| Revolute Joint | $\Phi_R = \mathbf{p}_b(\mathbf{q}) - \mathbf{p}_a(\mathbf{q}) = 0$ | 2 per joint |
| Groove Joint | $\Phi_G = \hat{\mathbf{n}}(s^*)^\top \cdot (\mathbf{p}(\mathbf{q}) - \mathbf{C}(s^*)) = 0$ | 1 per joint |
| Driving | $\Phi_D = \varphi_{input} - (\varphi_0 + \theta(t)) = 0$ | 1 |

<!--
TODO: 이미지 추가
- groove_mechanism_kinematics.gif
- groove_solver_vs_solidworks.gif
- optimization_result 관련 이미지들
-->
