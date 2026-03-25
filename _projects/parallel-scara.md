---
title: "Parallel SCARA"
excerpt: "2-DoF Parallel 5-Bar SCARA Robot — Trajectory Planning, Inverse Kinematics, Motor Control 통합 시스템"
header:
  image: /assets/images/projects/parallel-scara.png
  teaser: /assets/images/projects/parallel-scara.png
tags:
  - Robotics
  - Control
  - Kinematics
  - Embedded
---

## Overview

2-DoF Parallel 5-Bar SCARA Robot의 **Trajectory Planning → Inverse Kinematics → Motor Control → Data Logging** 전체 파이프라인을 구현한 프로젝트입니다.

## System Architecture

5-Layer 아키텍처를 채택하여 유지보수성과 확장성을 극대화했습니다.

| Layer | Description |
|:---|:---|
| **User Layer** | 사용자 입력 처리 및 시각화 (GUI) |
| **Control Layer** | 시퀀스 제어, 안전 관리 (EMS, Homing) |
| **Motion Layer** | 경로 생성, 역기구학 계산 |
| **Comm Layer** | 하드웨어 통신 인터페이스 (CAN Bus) |
| **HW Layer** | 액추에이터 및 센서 (AK80-9, Photo Sensor, Solenoid) |

## Key Features

### Inverse Kinematics — Gröbner Basis
- **Gröbner Basis**를 활용한 5-Bar Parallel Mechanism의 해석적(Analytical) 역기구학 풀이
- 수치 해석 대비 특이점(Singularity) 문제 없이 빠르고 정확한 관절 각도 계산

### Trajectory Planning
- **기본 도형**: Circle, Triangle, Spiral, Rose 등 다양한 기하학적 경로 지원
- **Pick & Place**: 정렬 알고리즘을 활용한 복합 경로 생성
- Analytical Motion Profiling을 통한 속도 프로파일 설계

### Control & Interface
- **CustomTkinter** 기반 실시간 모니터링/제어 GUI
- 실시간 EMS(비상 정지) 모니터링 및 Auto Homing
- CSV 기반 위치/속도 데이터 자동 로깅

## Tech Stack

| Category | Details |
|:---|:---|
| Language | Python 3.9+ |
| Platform | Raspberry Pi 4B / 5 |
| GUI | CustomTkinter |
| Motor Control | CubeMars AK10-9 & AK80-9 (CAN Bus) |
| Math | NumPy, Gröbner Basis IK |
| Simulation | MATLAB (Workspace Analysis, Singularity Map) |
| PCB | Custom Carrier Board (KiCad, 3 iterations) |

## Simulation

MATLAB 기반 시뮬레이션으로 역기구학 검증 및 작업 영역 분석을 수행했습니다.
- **grobnerFiveBar.m** — 5-Bar Mechanism 기구학 검증
- **guiManager.m** — Workspace 및 Singularity 시각화 GUI

## Hardware

Custom PCB 설계를 통해 신호 무결성 및 전원 관리를 개선했습니다 (V1.0 → V1.2, 3차 반복 설계).

## Links

- [GitHub Repository](https://github.com/junyoung-five/Parallel_SCARA_Controller)
