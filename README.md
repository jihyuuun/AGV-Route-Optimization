# AGV-Route-Optimization
Capacitated VRP with Time Windows 기반 AGV 경로 최적화: 하이브리드 메타휴리스틱 알고리즘 개발

🏆️ **수상**: 스마트 해운물류 x AI 미션 챌린지 : 스마트 항만 AGV 경로 최적화 경진대회 예선 4위 (본선 진출)


📅 **기간**: Sep 2025 – Oct 2025

## 📖 프로젝트 개요  

본 프로젝트는 스마트 항만 환경에서 **AGV(Automated Guided Vehicle)의 경로를 최적화**해야 하는 **CVRPTW(Capacitated VRP with Time Windows)** 문제를 해결하기 위한 효율적인 알고리즘을 설계합니다. 먼저, **Batch greedy append heuristic**을 통해 초기해를 구성하고, **Variable Neighborhood Search with Simulated Annealing (VNS/SA)** 로 해를 개선하여 총 비용을 최소화합니다.

## 📂 프로젝트 구조

```
├── modeling/                      
│   ├── Khuopt_final_제출용.ipynb          # 알고리즘 코드
├── visualization/                      
│   ├── vivid.ipynb                       # 경로 시각화 및 간트 차트
├── README.md                             # 프로젝트 설명
```

## 🔍 문제 상황

**목적함수**

min 총 이동 시간 + 총 작업 시간 + 지각 계수(λ) x 총 지각 시간

**제약 조건**
- 한 번의 투어(DEPOT → 작업들 → DEPOT) 동안 적재 용량 초과 불가
- 한 번의 투어(DEPOT → 작업들 → DEPOT) 동안 최대 이동거리 초과 불가

**이동 규칙**

이동 시간 = 이동 거리 ÷ AGV 속도
- 모든 AGV는 동시에 출발하며, 서로 충돌하지 않는다고 가정
- AGV는 격자 모양 센서를 따라 이동하여 두 지점 간 거리는 맨해튼 거리로 계산

**작업 규칙**

지각 시간 = 지각 계수(λ) x max(0, 작업 완료 시각 – 수행 기한)
- 작업 지점에 도착하면 해당 작업의 작업 시간만큼 작업을 수행
- 하나의 작업은 한 번에 처리되어야 하고, 나누어서 처리 불가

## 🎯 알고리즘 설계

### **1️⃣ Construction**
: 초기해 구성

- **Batch greedy append heuristic**

  1단계
  - 작업: 수행 기한이 짧은 k1개 작업을 DEPOT에서의 맨해튼 거리가 작은 순서로 정렬
  - AGV: 속도가 느리고, 용량이 작고, 최대 주행 가능 거리가 작은 순서로 정렬
  - k1개의 정렬된 작업을 정렬된 AGV에 일대일 배정
  
  → 인사이트 1, 3 적용
  
  2단계 
  - 작업: 수행 기한이 짧은 k2개 작업을 DEPOT에서의 맨해튼 거리가 작은 순서로 정렬
  - AGV
    
    1) 속도가 2이고 해당 작업의 좌하단에 있는 AGV 중 예상 작업 완료 시각이 가장 빠른 AGV 선택
    2) 위 조건을 만족하는 AGV가 없다면, 속도가 1이고 해당 작업의 우상단에 있는 AGV 중 예상 작업 완료  시각이 가장 빠른 AGV 선택
    3) 위 조건을 만족하는 AGV가 없다면, 모든 AGV 중 예상 작업 완료 시각이 가장 빠른 AGV 선택
  - 남은 작업이 없을 때까지 k2개의 정렬된 작업을 위 조건을 만족하는 AGV에 배정
  
  → 인사이트 2 적용

### **2️⃣ Improvement**
: 해 개선

- **Variable Neighborhood Search (VNS)**
  - Shaking phase
  
    a) 투어 내 한 작업 위치 이동 (같은 AGV 선택 시)
    
    b) 투어 간 두 작업 위치 교환 (다른 AGV 선택 시)
    
    k번의 무작위 작업 이동/교환 수행 (k값은 adaptive하게 정체 시 증가)
  
  - Local Search phase
    
    a) Intra Or-Opt: 투어 내 연속된 작업 블록을 다른 위치로 이동
    
    b) Intra 2-Opt: 투어 내 연속된 작업 블록을 역순으로 뒤집기
    
    c) Inter Cross-Exchange: 두 AGV 간 연속된 작업 블록 교환
    
    d) Intra 3-opt Partial: 투어 내 세 지점 분할 후 네 작업 블록 재배열

- **Simulated Annealing (SA)**
  - 수용 방법
    - 후보해가 현재해보다 작거나 같으면 수용
    - 후보해가 현재해보다 크더라도 다음과 같은 확률로 수용

      $$Prob{acceptance} = e^x where x = −(현재해 − 후보해)/T$$

- **VNS/SA**
  
  <img width="414" height="119" alt="image" src="https://github.com/user-attachments/assets/16e4b2d4-b727-4f3e-b78b-f9ad97d4f923" />

## 📊 결과

### **개발 및 실험 환경**

- Python in Microsoft® Visual Studio 2022
- Intel(R) Core(TM) Ultra 5 125H @ 3600Mhz on Window 11

### **알고리즘 비교**

- Construction
  - Greedy insertion heuristic (목적함수 값=19946)
    : 수행 기한이 짧은 작업부터 해당 작업을 가장 빨리 처리를 할 수 있는 AGV에 하나씩 배정하는 대회에서 제공한 베이스라인 알고리즘

    <img width="843" height="242" alt="image" src="https://github.com/user-attachments/assets/4bf0bba4-a8c5-4f5d-9c54-65af6bdc9511" />

  - Batch greedy append heuristic (목적함수 값=18241)

    <img width="843" height="242" alt="image" src="https://github.com/user-attachments/assets/3025a870-48e1-4dee-bb78-f4ca725ed899" />

- Improvement
  - VNS/SA (최종 목적함수 값=12457)
 
    <img width="843" height="242" alt="image" src="https://github.com/user-attachments/assets/28a8322d-7682-477e-a964-ad41c6dfd5aa" />

    - 1차 목적함수 값=12509
    - 2차 목적함수 값=12509
    - 3차 목적함수 값=12457

