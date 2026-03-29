---
title: Gaussian Elimination
layout: single
author_profile: true
sidebar:
  nav: study
use_math: true
toc: true
toc_sticky: true
---

> 가우스 소거법(Gaussian Elimination)과 가우스-조던 소거법(Gauss-Jordan Elimination)의 개념 및 절차를 정리합니다.

## 선형 연립방정식과 행렬 표현

다음과 같은 $m$개의 방정식, $n$개의 미지수로 이루어진 선형 연립방정식을 생각합니다:

$$
\begin{cases}
a_{11}x_1 + a_{12}x_2 + \cdots + a_{1n}x_n = b_1 \\
a_{21}x_1 + a_{22}x_2 + \cdots + a_{2n}x_n = b_2 \\
\quad \vdots \\
a_{m1}x_1 + a_{m2}x_2 + \cdots + a_{mn}x_n = b_m
\end{cases}
$$

이를 **첨가 행렬(Augmented Matrix)**로 표현하면:

$$
[A \mid \mathbf{b}] =
\left[\begin{array}{cccc|c}
a_{11} & a_{12} & \cdots & a_{1n} & b_1 \\
a_{21} & a_{22} & \cdots & a_{2n} & b_2 \\
\vdots & \vdots & \ddots & \vdots & \vdots \\
a_{m1} & a_{m2} & \cdots & a_{mn} & b_m
\end{array}\right]
$$

가우스 소거법과 가우스-조던 소거법은 이 첨가 행렬에 **기본 행 연산(Elementary Row Operations)**을 적용하여 해를 구하는 체계적인 방법입니다.

## 기본 행 연산

행렬에 적용할 수 있는 기본 행 연산은 세 가지입니다:

1. **행 교환(Row Interchange)**: 두 행의 위치를 교환 — $R_i \leftrightarrow R_j$
2. **스칼라 곱(Row Scaling)**: 한 행에 0이 아닌 상수를 곱함 — $R_i \leftarrow c \cdot R_i \quad (c \neq 0)$
3. **행 덧셈(Row Addition)**: 한 행에 다른 행의 상수배를 더함 — $R_i \leftarrow R_i + c \cdot R_j$

이 연산들은 연립방정식의 **해 집합을 변경하지 않으면서** 행렬의 형태를 변환합니다. 즉, 원래 연립방정식과 **동치(equivalent)**인 연립방정식을 만들어 냅니다.

## 가우스 소거법 (Gaussian Elimination)

가우스 소거법은 첨가 행렬을 **행 사다리꼴(Row Echelon Form, REF)**로 변환한 뒤 **후진 대입(Back Substitution)**으로 해를 구하는 방법입니다.

### 행 사다리꼴 (REF)

행렬이 행 사다리꼴이 되려면 다음 조건을 만족해야 합니다:

1. 모든 영행(zero row)은 행렬의 맨 아래에 위치
2. 각 행의 **선행 원소(pivot, leading entry)**는 바로 윗행의 선행 원소보다 오른쪽에 위치
3. 선행 원소 아래의 같은 열 원소는 모두 0

$$
\left[\begin{array}{cccc|c}
\boxed{a_{11}} & a_{12} & a_{13} & a_{14} & b_1 \\
0 & \boxed{a_{22}'} & a_{23}' & a_{24}' & b_2' \\
0 & 0 & \boxed{a_{33}''} & a_{34}'' & b_3'' \\
0 & 0 & 0 & 0 & b_4''
\end{array}\right]
$$

### 전진 소거 절차 (Forward Elimination)

$n \times n$ 정방 행렬 기준으로 전진 소거 절차를 설명합니다.

**Step 1** — 1열에서 피벗 $a_{11}$을 선택하고, 1행 아래의 모든 행에서 1열 원소를 소거:

$$
R_i \leftarrow R_i - \frac{a_{i1}}{a_{11}} R_1, \quad i = 2, 3, \ldots, m
$$

**Step 2** — 2열에서 피벗 $a_{22}'$를 선택하고, 2행 아래의 모든 행에서 2열 원소를 소거:

$$
R_i \leftarrow R_i - \frac{a_{i2}'}{a_{22}'} R_2, \quad i = 3, 4, \ldots, m
$$

**Step k (일반)** — $k$열의 피벗 아래 원소를 소거:

$$
R_i \leftarrow R_i - \frac{a_{ik}^{(k)}}{a_{kk}^{(k)}} R_k, \quad i = k+1, \ldots, m
$$

이 과정을 $k = 1, 2, \ldots, n-1$까지 반복하면 REF가 완성됩니다.

### 부분 피벗팅 (Partial Pivoting)

전진 소거 중 피벗 $a_{kk}^{(k)}$가 0이거나 매우 작은 경우, $k$열의 $k$행 이하에서 **절댓값이 가장 큰 원소**를 가진 행과 교환합니다:

$$
|a_{pk}^{(k)}| = \max_{i \geq k} |a_{ik}^{(k)}| \quad \Rightarrow \quad R_k \leftrightarrow R_p
$$

부분 피벗팅은 수치적 안정성을 크게 향상시키며, 실용적인 구현에서는 거의 필수적으로 적용됩니다.

### 후진 대입 (Back Substitution)

REF를 얻은 후, 마지막 행부터 역순으로 미지수를 구합니다. $n \times n$ 시스템에서 REF가 다음과 같을 때:

$$
\left[\begin{array}{ccc|c}
u_{11} & u_{12} & u_{13} & c_1 \\
0 & u_{22} & u_{23} & c_2 \\
0 & 0 & u_{33} & c_3
\end{array}\right]
$$

후진 대입은 아래에서 위로 진행합니다:

$$
x_n = \frac{c_n}{u_{nn}}, \quad x_k = \frac{1}{u_{kk}} \left( c_k - \sum_{j=k+1}^{n} u_{kj} x_j \right), \quad k = n-1, \ldots, 1
$$

### 수치 예제

다음 연립방정식을 가우스 소거법으로 풀어보겠습니다:

$$
\begin{cases}
2x + y - z = 8 \\
-3x - y + 2z = -11 \\
-2x + y + 2z = -3
\end{cases}
$$

**첨가 행렬 구성:**

$$
\left[\begin{array}{ccc|c}
2 & 1 & -1 & 8 \\
-3 & -1 & 2 & -11 \\
-2 & 1 & 2 & -3
\end{array}\right]
$$

**전진 소거:**

$R_2 \leftarrow R_2 + \frac{3}{2}R_1$, $R_3 \leftarrow R_3 + R_1$:

$$
\left[\begin{array}{ccc|c}
2 & 1 & -1 & 8 \\
0 & \frac{1}{2} & \frac{1}{2} & 1 \\
0 & 2 & 1 & 5
\end{array}\right]
$$

$R_3 \leftarrow R_3 - 4R_2$:

$$
\left[\begin{array}{ccc|c}
2 & 1 & -1 & 8 \\
0 & \frac{1}{2} & \frac{1}{2} & 1 \\
0 & 0 & -1 & 1
\end{array}\right]
$$

**후진 대입:**

$$
-z = 1 \Rightarrow z = -1
$$

$$
\frac{1}{2}y + \frac{1}{2}(-1) = 1 \Rightarrow y = 3
$$

$$
2x + 3 - (-1) = 8 \Rightarrow x = 2
$$

$$
\therefore \quad (x, y, z) = (2, 3, -1)
$$

## 가우스-조던 소거법 (Gauss-Jordan Elimination)

가우스-조던 소거법은 행렬을 REF가 아닌 **기약 행 사다리꼴(Reduced Row Echelon Form, RREF)**로 변환하여, 후진 대입 없이 **직접 해를 읽어내는** 방법입니다.

### 기약 행 사다리꼴 (RREF)

RREF는 REF의 조건에 추가로 다음을 만족합니다:

1. 각 행의 선행 원소(피벗)가 **1**
2. 피벗이 포함된 열에서 피벗을 제외한 나머지 원소가 모두 **0** (피벗 위쪽도 포함)

$$
\left[\begin{array}{cccc|c}
\boxed{1} & 0 & 0 & \ast & c_1 \\
0 & \boxed{1} & 0 & \ast & c_2 \\
0 & 0 & \boxed{1} & \ast & c_3 \\
0 & 0 & 0 & 0 & c_4
\end{array}\right]
$$

여기서 $\ast$는 자유 변수(free variable)에 대응하는 열의 원소를 나타냅니다.

### 절차

가우스-조던 소거법은 가우스 소거법의 전진 소거에 **후진 소거(Back Elimination)**를 추가한 것입니다:

1. **전진 소거**: 가우스 소거법과 동일하게 REF를 만듦
2. **피벗 정규화**: 각 행을 피벗으로 나누어 피벗을 1로 만듦 — $R_k \leftarrow \frac{1}{a_{kk}} R_k$
3. **후진 소거**: 아래 행부터 위로 올라가며, 피벗 열의 **위쪽** 원소도 소거 — $R_i \leftarrow R_i - a_{ik} R_k, \quad i < k$

### 수치 예제

앞의 예제를 이어서 가우스-조던 소거법으로 완성합니다. REF에서 이어서 진행:

$$
\left[\begin{array}{ccc|c}
2 & 1 & -1 & 8 \\
0 & \frac{1}{2} & \frac{1}{2} & 1 \\
0 & 0 & -1 & 1
\end{array}\right]
$$

**피벗 정규화:** $R_1 \leftarrow \frac{1}{2}R_1$, $R_2 \leftarrow 2R_2$, $R_3 \leftarrow -R_3$:

$$
\left[\begin{array}{ccc|c}
1 & \frac{1}{2} & -\frac{1}{2} & 4 \\
0 & 1 & 1 & 2 \\
0 & 0 & 1 & -1
\end{array}\right]
$$

**후진 소거:**

$R_2 \leftarrow R_2 - R_3$, $R_1 \leftarrow R_1 + \frac{1}{2}R_3$:

$$
\left[\begin{array}{ccc|c}
1 & \frac{1}{2} & 0 & \frac{7}{2} \\
0 & 1 & 0 & 3 \\
0 & 0 & 1 & -1
\end{array}\right]
$$

$R_1 \leftarrow R_1 - \frac{1}{2}R_2$:

$$
\left[\begin{array}{ccc|c}
1 & 0 & 0 & 2 \\
0 & 1 & 0 & 3 \\
0 & 0 & 1 & -1
\end{array}\right]
$$

RREF로부터 **후진 대입 없이** 해를 직접 읽습니다:

$$
(x, y, z) = (2, 3, -1)
$$

## 가우스 소거법 vs 가우스-조던 소거법

| | 가우스 소거법 | 가우스-조던 소거법 |
|:---|:---|:---|
| **목표 형태** | REF (행 사다리꼴) | RREF (기약 행 사다리꼴) |
| **소거 방향** | 전진(아래 방향)만 | 전진 + 후진(양방향) |
| **해 도출** | 후진 대입 필요 | RREF에서 직접 읽음 |
| **연산량** | $\sim \frac{2}{3}n^3$ (후진 대입 포함) | $\sim n^3$ |
| **주 용도** | 연립방정식 풀기 | 역행렬 계산, 해 구조 분석 |

연산량 관점에서 가우스 소거법이 효율적이므로 **연립방정식 풀이**에는 가우스 소거법 + 후진 대입이 선호됩니다. 반면, 가우스-조던 소거법은 **역행렬 계산**이나 해 공간의 구조 파악에 유리합니다.

## 역행렬 계산에의 적용

$n \times n$ 정방 행렬 $A$의 역행렬을 구하려면, $[A \mid I_n]$에 가우스-조던 소거법을 적용합니다:

$$
[A \mid I_n] \xrightarrow{\text{RREF}} [I_n \mid A^{-1}]
$$

좌측이 단위 행렬이 되면, 우측에 역행렬이 자동으로 나타납니다. 만약 좌측이 단위 행렬이 되지 않으면 $A$는 **비가역(singular)**입니다.

## 해의 존재성과 유일성

RREF를 통해 연립방정식의 해를 분류할 수 있습니다:

- **유일한 해**: 모든 열에 피벗이 존재하고 모순이 없음
- **무수히 많은 해**: 피벗이 없는 열(자유 변수)이 존재하고 모순이 없음
- **해 없음**: $[0 \ 0 \ \cdots \ 0 \mid c]$ $(c \neq 0)$ 형태의 행이 존재 (모순)

이 분류는 **Rank-Nullity Theorem**과 직결됩니다:

$$
\text{rank}(A) + \text{nullity}(A) = n
$$

여기서 $\text{rank}(A)$는 피벗의 개수이고, $\text{nullity}(A)$는 자유 변수의 개수입니다.

## References

- Strang, G. (2016). *Introduction to Linear Algebra* (5th ed.). Wellesley-Cambridge Press
- Lay, D. C., Lay, S. R., & McDonald, J. J. (2016). *Linear Algebra and Its Applications* (5th ed.). Pearson
