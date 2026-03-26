---
title: "Parallel SCARA"
excerpt: "2-DoF Parallel 5-Bar SCARA Robot — Gröbner Basis IK, Analytical Velocity Profiling, Layered Motor Control"
header:
  image: /assets/images/projects/parallel-scara.png
  teaser: /assets/images/projects/parallel-scara.png
tags:
  - Robotics
  - Control
  - Kinematics
  - Embedded
toc: true
toc_sticky: true
---

## 연구 배경

산업 현장에서 Pick & Place 작업에 널리 사용되는 SCARA 로봇은, 일반적으로 직렬(Serial) 구조를 채택합니다. 반면 **병렬(Parallel) 5-Bar 구조**는 두 모터가 모두 베이스에 고정되어 관성이 작고 고속 동작에 유리하지만, 역기구학이 복잡하여 실시간 제어가 까다롭다는 한계가 있습니다.

기존의 수치 해석 기반 역기구학(Newton-Raphson 등)은 초기값 의존성과 특이점 근방에서의 발산 문제가 존재합니다. 또한 수치 미분으로 산출된 속도 프로파일은 정밀도가 떨어져, 이를 보상하기 위해 PID 등 복잡한 제어기가 요구되는 구조가 일반적입니다.

## 연구 목적

**[Gröbner Basis](/study/mathematics/grobner-basis/)** 기반의 해석적 역기구학을 통해 관절 궤적과 속도 프로파일을 오프라인에서 정밀하게 사전 계산하고, 이를 기반으로 상위 제어기의 복잡도를 최소화하여 **피드포워드 + P 보정만으로 충분한 추종 정밀도를 달성**하는 것을 목표로 합니다.

궁극적으로, Trajectory Planning부터 Motor Control까지의 전체 파이프라인을 통합 구현하여 실제 로봇에서 동작을 검증합니다.

## 연구 방법

### System Hierarchy

본 시스템은 5개의 계층으로 구성되며, 각 계층의 역할이 명확히 분리되어 있습니다.

```
Layer 4 ─ Trajectory Planning (오프라인, Python)
  Gröbner IK + Analytical Jacobian + 5th-order Smoothing
  → 관절 위치/속도 프로파일 사전 계산

Layer 3 ─ Supervisory Controller (100Hz, Python)
  v_cmd = v_feedforward + Kp × position_error
  → 모터에 전달할 속도 명령 생성

Layer 2 ─ CAN Communication (Python → SocketCAN)
  rad/s → ERPM 변환 → int32 Big-Endian → CAN 프레임 전송

Layer 1 ─ Motor Firmware (VESC 기반, 모터 내부)
  속도 PI 제어 → FOC 전류 제어 → PWM 출력

Layer 0 ─ Electrical / Mechanical
  3상 인버터 → BLDC 코일 → 9:1 감속기 → 관절 회전
```

**설계 의도**: Layer 4에서 궤적을 해석적으로 정밀하게 계산함으로써, Layer 3의 상위 제어기는 피드포워드 + P 보정만으로 충분한 추종 정밀도를 달성합니다. 모터 내부 VESC 펌웨어(Layer 1)의 속도 PI + FOC 전류 제어가 댐핑과 정상상태 오차 제거를 담당하므로, 상위 제어기에서 별도의 I, D항이 불필요한 구조입니다.

### Motion Pipeline: P1 → P2

엔드이펙터가 점 P1에서 P2로 이동할 때의 전체 데이터 흐름입니다.

#### Step 1. Cartesian Path Generation

P1과 P2 사이를 선형 보간하되, 시간 매개변수에 **5차 다항식 스무딩**을 적용합니다.

```
s(t) = 10t³ - 15t⁴ + 6t⁵
```

이 함수는 `s(0)=0, s(1)=1`이면서 `s'(0)=s'(1)=0`, `s''(0)=s''(1)=0`을 만족하여, 경로의 시작과 끝에서 속도와 가속도가 자연스럽게 0이 됩니다.

#### Step 2. Inverse Kinematics — Gröbner Basis

각 카테시안 웨이포인트 (x, y)에 대해 해석적 역기구학을 수행합니다.

```
θ₁ = 2·atan(S₁(x, y))
θ₂ = 2·atan(S₂(x, y))
```

S₁, S₂는 5-Bar Parallel Mechanism의 기구학 방정식을 **[Gröbner Basis](/study/mathematics/grobner-basis/)**로 풀어 유도된 닫힌 형태(closed-form) 해입니다. 수치 해석 기반 IK 대비 다음과 같은 이점이 있습니다:

- 반복 수렴 과정이 없어 계산이 빠르고 결정론적
- 특이점(singularity) 근방에서도 수치 발산 없이 해를 산출
- 해석적 미분이 가능하여 야코비안을 심볼릭으로 유도 가능

#### Step 3. Analytical Jacobian → Joint Velocity

Gröbner Basis 해(S₁, S₂)를 심볼릭 미분하여 **해석적 야코비안 행렬** J(2×2)를 유도합니다.

```
J = [∂θ₁/∂x  ∂θ₁/∂y]    (체인 룰: ∂θ/∂x = 2/(1+S²) · ∂S/∂x)
    [∂θ₂/∂x  ∂θ₂/∂y]
```

카테시안 속도에 야코비안을 적용하여 관절 속도를 산출합니다:

```
[ω₁, ω₂]ᵀ = J · [ẋ, ẏ]ᵀ
```

야코비안 내부는 CSE(Common Subexpression Elimination)로 최적화되어 32개의 중간 변수로 효율적으로 계산됩니다.

#### Step 4. Interpolation Functions

이산 데이터를 연속 시간 함수로 변환합니다:

- `posFun(t)` → 시각 t에서의 목표 관절 위치 (선형 보간)
- `velFun(t)` → 시각 t에서의 피드포워드 관절 속도 (선형 보간)

#### Step 5. Real-time Control Loop (100Hz)

```python
# JointController.solveSpeed — 매 10ms마다 실행
targetPos = posFun(t)                          # 목표 위치
targetVel = velFun(t)                          # 피드포워드 속도
err = (targetPos - actualPos + π) % 2π - π     # 각도 래핑된 위치 오차
cmdVel = targetVel + Kp × err                  # 제어 법칙 (Kp = 2.0)
```

#### Step 6. CAN Transmission → TMotor

```
cmdVel [rad/s]
  → ERPM 변환 (÷ 5.82×10⁻⁴)
    → int32 Big-Endian 4바이트 패킹
      → CAN 프레임 (arbitration_id = motor_ID | 0x0300)
        → TMotor AK10-9 속도 모드 수신
```

### Angle Continuity

역기구학의 `atan` 특성상 π/-π 경계에서 불연속이 발생합니다. 이를 두 단계로 처리합니다:

1. **세그먼트 내**: `np.unwrap` + 커스텀 클리핑으로 연속성 보장
2. **세그먼트 간**: 이전 세그먼트의 종료 각도를 기준으로 2π 오프셋 보정

이를 통해 다중 세그먼트 궤적(Pick & Place 등)에서도 모터에 급격한 명령이 전달되지 않습니다.

### CAN Communication Architecture

Python 환경에서의 모터 제어는 다음 스택으로 구현됩니다:

```
Python (servo_can.py)
  → python-can 라이브러리
    → Linux SocketCAN 커널 드라이버
      → USB-CAN 어댑터 (하드웨어)
        → CAN Bus (1Mbps)
          → TMotor AK10-9
```

**CAN 프로토콜**: Arbitration ID에 모터 ID(하위 8bit)와 명령 타입(bit 8-15)을 인코딩합니다. 데이터는 Big-Endian으로 패킹됩니다.

**비동기 수신**: `python-can`의 `Notifier` + `Listener` 패턴으로 모터 피드백(위치, 속도, 전류, 온도, 에러)을 비동기 수신하며, **더블 버퍼링**으로 비동기 버퍼와 동기 버퍼를 분리하여 읽기/쓰기 경합을 방지합니다.

### Motor Internal Control (VESC Firmware)

TMotor AK10-9는 VESC 기반 펌웨어를 내장하고 있으며, 속도 모드에서 다음과 같은 이중 루프 제어를 수행합니다:

- **외측 루프**: 속도 PI 제어 → 목표 전류(Iq) 산출
- **내측 루프**: FOC(Field-Oriented Control) 전류 제어 → PWM 듀티 산출

이 내부 제어기가 댐핑(D적 역할)과 정상상태 오차 제거(I적 역할)를 담당하므로, 상위 제어기에서 PID가 불필요합니다.

### Trajectory Types

| 유형 | 설명 |
|:---|:---|
| **기하학적 경로** | Circle, Triangle, Square, Star, Spiral, Sinewave, Heart, Rose |
| **Pick & Place** | 다중 세그먼트 정렬 경로 (Vertical / Horizontal Sorting) |
| **Custom Segment** | 임의의 두 점 사이 단일 세그먼트 이동 |

모든 경로는 5차 스무딩이 적용되며, Pick & Place는 거리 비례 시간 배분과 세그먼트 간 연속성 보정이 추가됩니다.

## 연구 결과

### Simulation

MATLAB 기반 시뮬레이션으로 설계 단계에서 기구학 검증 및 작업 영역 분석을 수행했습니다.

- **grobnerFiveBar.m** — 5-Bar Mechanism 역기구학 검증
- **guiManager.m** — Workspace 및 Manipulability Map 시각화 GUI

### Tech Stack

| Category | Details |
|:---|:---|
| Language | Python 3.9+ |
| Platform | Raspberry Pi 4B / 5 |
| GUI | CustomTkinter |
| Motor | CubeMars AK10-9 (CAN Bus, VESC Firmware) |
| Communication | python-can → Linux SocketCAN → USB-CAN Adapter |
| Math | NumPy, Gröbner Basis IK, Analytical Jacobian (CSE Optimized) |
| Simulation | MATLAB (Workspace Analysis, Manipulability Map) |
| PCB | Custom Carrier Board (KiCad, 3 iterations) |

## 로봇 제작 및 검증

### Hardware

Custom PCB 설계를 통해 Raspberry Pi와 CAN 트랜시버, 센서 인터페이스를 통합했습니다 (V1.0 → V1.2, 3차 반복 설계).

### Safety & Peripherals

- **EMS (Emergency Stop)**: GPIO 기반 비상 정지 모니터링, 제어 루프 매 주기 체크
- **Auto Homing**: 초기 관절 위치로 자동 복귀
- **Solenoid Control**: 직렬 통신 기반 진공 흡착 Pick/Place 제어
- **Temperature Protection**: 모터 MOSFET 온도 50°C 초과 시 자동 정지
- **Context Manager**: `with` 구문으로 모터 전원 on/off를 안전하게 관리

## 결론

본 프로젝트는 Gröbner Basis 기반의 해석적 역기구학과 야코비안을 활용하여, 5-Bar Parallel SCARA 로봇의 궤적 계획 정밀도를 극대화하고 상위 제어기의 복잡도를 최소화하는 접근을 제시했습니다.

핵심 기여는 다음과 같습니다:

- **해석적 IK/야코비안**: Gröbner Basis로 유도된 닫힌 형태 해를 통해, 수치 해석 대비 결정론적이고 특이점에 강건한 역기구학을 구현
- **계층적 제어 구조**: 모터 내부 VESC 펌웨어(속도 PI + FOC)와 상위 제어기(피드포워드 + P)의 역할을 명확히 분리하여, 각 계층의 책임을 최적화
- **정밀한 오프라인 계획이 온라인 제어를 단순화**: 해석적 속도 프로파일의 품질이 피드포워드 정확도를 보장함으로써, 복잡한 PID 튜닝 없이도 충분한 추종 성능을 달성

## Links

- [GitHub Repository](https://github.com/junyoung-five/Parallel_SCARA_Controller)
