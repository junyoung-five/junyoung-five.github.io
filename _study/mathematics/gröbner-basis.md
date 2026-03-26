---
title: Gröbner-Basis
layout: single
author_profile: true
sidebar:
  nav: study
use_math: true
toc: true
toc_sticky: true
---

> 그뢰브너 기저(Gröbner Basis)의 개념과 로봇 역기구학에의 적용을 정리합니다.

## 다항식 연립방정식을 푸는 문제

로봇의 역기구학(Inverse Kinematics)은 결국 **다항식 연립방정식**을 푸는 문제로 귀결되는 경우가 많습니다. 예를 들어, 2-DoF 5-Bar Parallel Mechanism에서 엔드이펙터 좌표 $(x, y)$가 주어졌을 때 관절 각도 $\theta_1, \theta_2$를 구하려면 다음과 같은 구속 조건을 풀어야 합니다:

$$
f_1(\theta_1, \theta_2, x, y) = 0
$$

$$
f_2(\theta_1, \theta_2, x, y) = 0
$$

이 방정식들이 삼각함수를 포함하더라도, **반각 치환** $s = \tan(\theta/2)$를 적용하면 순수한 다항식으로 변환됩니다:

$$
\cos\theta = \frac{1 - s^2}{1 + s^2}, \quad \sin\theta = \frac{2s}{1 + s^2}
$$

이제 문제는 **다변수 다항식 연립방정식**의 해를 구하는 것이 됩니다.

## 1변수에서의 직관: 유클리드 호제법

Gröbner Basis를 이해하려면, 먼저 1변수 다항식에서의 유사한 문제를 생각하면 좋습니다.

두 다항식 $f(x) = 0$, $g(x) = 0$의 **공통근**을 찾고 싶다면, **유클리드 호제법(GCD)**을 사용합니다:

$$
\gcd(f, g) = h(x) \quad \Rightarrow \quad h(x) = 0 \text{ 이 공통근}
$$

이 과정에서 나눗셈을 반복하여 차수를 줄여 나갑니다. Gröbner Basis는 이 아이디어를 **다변수**로 확장한 것입니다.

## Gröbner Basis란?

### 이상(Ideal)과 생성원

다항식 $f_1, f_2, \ldots, f_m$이 주어졌을 때, 이들의 **이상(Ideal)**은:

$$
I = \langle f_1, f_2, \ldots, f_m \rangle = \left\{ \sum_{i=1}^{m} h_i \cdot f_i \ \middle|\ h_i \in k[x_1, \ldots, x_n] \right\}
$$

즉, $f_1, \ldots, f_m$의 모든 다항식 결합(polynomial combination)으로 만들 수 있는 다항식의 집합입니다. 핵심은 **같은 이상에 속하는 다른 생성원 집합을 찾을 수 있다**는 것입니다:

$$
\langle f_1, f_2, \ldots, f_m \rangle = \langle g_1, g_2, \ldots, g_k \rangle
$$

양쪽의 연립방정식은 **정확히 같은 해 집합**을 가지지만, 오른쪽이 훨씬 풀기 쉬울 수 있습니다.

### 단항식 순서(Monomial Ordering)

다변수 다항식에서는 "어떤 항이 더 높은 차수인가?"가 자명하지 않습니다. 예를 들어 $x^2 y$와 $x y^2$ 중 어느 것이 먼저인지를 정해야 합니다. 대표적인 순서:

- **사전식 순서(Lexicographic, lex)**: $x > y$일 때, $x$의 차수를 먼저 비교. $x^2 y > x y^2 > y^3$
- **전차수 역사전식(grevlex)**: 총차수를 먼저 비교, 같으면 역사전식

**사전식 순서(lex)**로 Gröbner Basis를 계산하면, 결과가 **삼각형 꼴(triangular form)**로 나옵니다:

$$
g_1(x_n) = 0
$$

$$
g_2(x_{n-1}, x_n) = 0
$$

$$
\vdots
$$

이것은 마치 가우스 소거법으로 행렬을 상삼각 형태로 만드는 것과 같은 효과입니다.

### Buchberger 알고리즘

Gröbner Basis를 계산하는 기본 알고리즘은 **Buchberger 알고리즘**입니다:

1. 생성원 쌍 $(f_i, f_j)$에 대해 **S-다항식(S-polynomial)**을 계산:

   $$
   S(f_i, f_j) = \frac{\text{lcm}(\text{LT}(f_i), \text{LT}(f_j))}{\text{LT}(f_i)} \cdot f_i - \frac{\text{lcm}(\text{LT}(f_i), \text{LT}(f_j))}{\text{LT}(f_j)} \cdot f_j
   $$

   여기서 $\text{LT}$는 선택한 단항식 순서에서의 선두항(Leading Term)입니다.

2. S-다항식을 현재 기저로 **나눗셈하여 나머지**를 구함
3. 나머지가 0이 아니면 기저에 추가
4. 모든 쌍의 나머지가 0이 될 때까지 반복

직관적으로, S-다항식은 두 다항식의 선두항을 소거하여 **새로운 관계식**을 발견하는 과정입니다. 1변수에서의 유클리드 호제법과 본질적으로 같은 역할을 합니다.

## 가우스 소거법과의 비유

Gröbner Basis가 하는 일을 선형대수와 비교하면 이해가 쉽습니다:

| 선형 연립방정식 | 다항식 연립방정식 |
|:---|:---|
| 계수 행렬 $A$ | 다항식 집합 $\{f_1, \ldots, f_m\}$ |
| 가우스 소거법 | Buchberger 알고리즘 |
| 행 사다리꼴(REF) | Gröbner Basis |
| 기약 행 사다리꼴(RREF) | 기약 Gröbner Basis(Reduced GB) |
| 후진 대입(Back Substitution) | 삼각형 꼴에서 순차적으로 해 대입 |

**핵심 차이**: 가우스 소거법은 1차식만 다루지만, Gröbner Basis는 **임의 차수의 다항식**을 다룹니다.

## 5-Bar Parallel Mechanism에의 적용

### 문제 설정

5-Bar Parallel SCARA의 기구학 구속 조건을 반각 치환으로 다항식화하면:

$$
f_1(s_1, s_2, x, y) = 0
$$

$$
f_2(s_1, s_2, x, y) = 0
$$

여기서 $s_1 = \tan(\theta_1/2)$, $s_2 = \tan(\theta_2/2)$이고, $(x, y)$는 주어진 엔드이펙터 좌표입니다.

### Gröbner Basis 적용 결과

사전식 순서 $s_1 > s_2$로 Gröbner Basis를 계산하면, **변수 소거**가 일어나 다음과 같은 삼각형 꼴이 됩니다:

$$
g_1(s_1, x, y) = 0 \quad \Rightarrow \quad s_1 = \frac{381y + \sqrt{-(9x^2 + 9y^2 - 1)(5625x^2 + 5625y^2 - 23104)}}{381x + 225x^2 + 225y^2 - 152}
$$

$$
g_2(s_2, x, y) = 0 \quad \Rightarrow \quad s_2 = \frac{-(381y - \sqrt{(9x^2 - 18x + 9y^2 + 8)(11250x - 5625x^2 - 5625y^2 + 17479)})}{69x - 225x^2 - 225y^2 + 308}
$$

$s_1$과 $s_2$가 **완전히 분리**되어, 각각 $(x, y)$만의 함수로 표현됩니다. 최종 관절 각도는:

$$
\theta_1 = 2\arctan(s_1(x, y)), \quad \theta_2 = 2\arctan(s_2(x, y))
$$

### 수치 해석 대비 이점

| | Newton-Raphson (수치 IK) | Gröbner Basis (해석 IK) |
|:---|:---|:---|
| **수렴성** | 초기값 의존, 발산 가능 | 항상 해를 산출 (닫힌 형태) |
| **특이점** | 야코비안 비정칙 → 발산 | 근호 내부 부호로 판별 가능 |
| **계산 비용** | 반복당 O(n³), 반복 횟수 불확정 | 단일 함수 평가 (O(1)) |
| **미분 가능성** | 수치 미분만 가능 | **심볼릭 미분** → 해석적 야코비안 |
| **실시간성** | 반복 횟수에 따라 가변 | 결정론적 실행 시간 |

마지막 항목이 특히 중요합니다. Gröbner Basis로 유도된 해석해를 심볼릭 미분하면 **해석적 야코비안**을 얻을 수 있고, 이를 통해 정밀한 관절 속도 프로파일을 생성할 수 있습니다. 이것이 [Parallel SCARA 프로젝트](/projects/parallel-scara/)에서 단순한 피드포워드 + P 제어만으로 충분한 추종 정밀도를 달성할 수 있었던 기반입니다.

## 한계와 고려사항

- **계산 복잡도**: Buchberger 알고리즘은 최악의 경우 이중 지수 시간(doubly exponential)이 소요될 수 있음. 단, 이는 **오프라인에서 한 번만** 수행하면 되므로 실시간 성능에는 영향 없음
- **변수 수 제한**: 변수가 많아지면 기저 계산이 급격히 어려워짐. 5-Bar(2변수)에서는 문제없지만, 6-DoF 이상에서는 다른 접근 필요
- **작업 영역**: 근호 내부가 음수가 되면 실수 해가 존재하지 않음 → 해당 $(x, y)$는 작업 영역 밖

## References

- Buchberger, B. (2006). *Bruno Buchberger's PhD thesis 1965: An algorithm for finding the basis elements of the residue class ring of a zero dimensional polynomial ideal*
- Cox, D., Little, J., & O'Shea, D. (2015). *Ideals, Varieties, and Algorithms* (4th ed.). Springer
