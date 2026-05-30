# 프로젝트 중간보고 | Building Energy Efficiency Analysis: London EPC

## 1. 프로젝트 개요

본 프로젝트는 London EPC (Energy Performance Certificate) 데이터를 활용하여 건물의 물리적/구조적 특성과 에너지 효율 간의 관계를 분석하고, 동일 건물에 대한 반복 평가 결과의 일관성을 검증하는 것을 목표로 함.

# 2. 중간 결과

## 2.1 데이터 수집 및 구조 이해

* London EPC 데이터셋 (domestic certificates) 로드

* 데이터가 borough 단위 폴더 구조로 분산되어 있어 병합 수행

```text
all-domestic-certificates/
 ├── domestic-E09xxxx/
      └── certificates.csv
```

## 2.2 데이터 정제 및 분석 대상 구성

초기 데이터에서 분석 목적에 맞게 핵심 변수만 선택:

```python
cols = [
    "UPRN",
    "CURRENT_ENERGY_RATING",
    "CURRENT_ENERGY_EFFICIENCY",
    "TOTAL_FLOOR_AREA",
    "CO2_EMISSIONS_CURRENT",
    "PROPERTY_TYPE",
    "BUILT_FORM",
    "CONSTRUCTION_AGE_BAND",
    "TENURE",
    "MAINHEAT_DESCRIPTION",
    "LODGEMENT_DATE"
]
```

### 전처리 과정

* UPRN 결측 제거 → 동일 건물 식별 가능 데이터만 유지

* 동일 건물 반복 평가 분석을 위해 **UPRN 기준 2회 이상 등장 데이터만 추출**

```text
전체 데이터: 8228 rows
중복 평가 데이터: 4786 rows
실제 건물 수: 약 2200개
```

## 2.3 동일 건물 기준 평가 일관성 분석

각 건물(UPRN)별로 에너지 등급 변화량 계산:

```python
rating_gap = max(rating) - min(rating)
```

### 결과

```text
rating_gap 분포:

0 → 1008
1 → 856
2 → 256
3 → 64
4 → 9
5 → 2
```
약 **15%의 건물에서 2단계 이상의 등급 변화 발생**

## 등급 변화 분포

<img width="580" height="448" alt="image" src="https://github.com/user-attachments/assets/5013ac68-a0a2-46ed-a61e-77e701803a79" />

### Insight

* 대부분의 건물은 안정적인 등급 유지

* 그러나 일부 건물에서는 큰 폭의 등급 변화 발생

* EPC 평가 결과가 동일 건물에 대해서도 완전히 일관적이지 않을 가능성 확인

## 2.4 이상 건물 분석 (rating_gap ≥ 2)

이상 건물: **331개**

# 2.5 효율 점수 변동 분석

각 건물의 efficiency 변화량 계산:

```python
eff_gap = max(efficiency) - min(efficiency)
```

### 결과

```text
평균: 30.9
중앙값: 29
최소: 13
최대: 78
```

## Rating vs Efficiency 변화

<img width="563" height="453" alt="image" src="https://github.com/user-attachments/assets/0f392e6a-8729-48fa-be1c-1f69f8bb959c" />

### Insight

* efficiency 점수 변화가 클수록 등급 변화도 증가

* 등급 변화는 랜덤하게 발생하는 것이 아니라, efficiency 점수 변동과 강하게 연결됨

* EPC 시스템은 사실상 efficiency 점수를 기반으로 등급을 결정하는 구조임을 확인

# 2.6 물리적 특성 안정성 검증

면적 변화 분석:

```text
median: 2.5
75%: ~7
```

### Insight

* 대부분의 건물에서 면적 변화는 매우 작음

* 동일 건물로 간주 가능한 수준의 안정성 확인

* 즉, 건물 자체보다 평가 결과 및 efficiency 점수의 변동성이 더 크게 나타남

# 2.7 모델 기반 검증 (CatBoost)

## 모델 구축

Leakage 방지를 위해
`CURRENT_ENERGY_EFFICIENCY` 제거 후 모델 학습 수행

사용 변수:

```python
[
    "TOTAL_FLOOR_AREA",
    "CO2_EMISSIONS_CURRENT",
    "PROPERTY_TYPE",
    "BUILT_FORM",
    "CONSTRUCTION_AGE_BAND",
    "TENURE",
    "MAINHEAT_DESCRIPTION"
]
```

## 모델 성능

```text
Accuracy ≈ 0.72
```

### 해석

* 건물 특성만으로도 EPC 등급 예측 가능

* EPC 시스템이 일정 수준의 구조적 규칙성을 가짐을 확인

## Confusion Matrix

<img width="583" height="547" alt="image" src="https://github.com/user-attachments/assets/205e42a7-1cc9-46a8-993a-c86669aeef2e" />

### Insight

* 대부분의 예측은 실제 등급과 일치

* 오차는 주로 인접한 등급(B↔C, C↔D) 사이에서 발생

* EPC 등급 체계 자체는 전반적으로 일관된 패턴을 보임

# 2.8 Feature Importance 분석

<img width="844" height="470" alt="image" src="https://github.com/user-attachments/assets/5a3571c0-593c-46f3-9499-39978275a29d" />

### 주요 변수

```text
1. CO2_EMISSIONS_CURRENT
2. MAINHEAT_DESCRIPTION
3. TOTAL_FLOOR_AREA
4. CONSTRUCTION_AGE_BAND
```

### Insight

* CO2 배출량이 EPC 등급 결정에 가장 큰 영향

* 난방 방식(Main Heating)이 중요한 변수로 작용

* 오래된 건물일수록 낮은 등급 경향 확인

# 2.9 Hidden Inefficiency 탐지

모델 예측 등급과 실제 EPC 등급 차이를 기반으로 anomaly 탐지 수행.

```text
모델 예측 > 실제 EPC 등급
```

인 경우를 hidden inefficiency 후보로 정의.

## 발견된 패턴

### 반복적으로 등장한 특성

* Electric underfloor heating

* Electric storage heaters

* Community scheme heating

* 오래된 건물(before 1900, 1930–1949)

* Flat / Terrace 계열 구조

## 핵심 결과

* 모델 기준으로는 정상 범주에 가까운 건물들이 실제 EPC에서는 낮은 등급(E/F/G)으로 평가되는 사례 확인

* 특정 electric heating 기반 건물에서 anomaly가 반복적으로 발생

## Strong Anomaly 사례

다음 조건을 동시에 만족하는 건물 발견:

* 동일 UPRN 내 반복 평가 inconsistency 존재

* 모델 기반 hidden inefficiency 존재

### 대표 사례

* 동일 UPRN 내에서 서로 다른 Built Form 및 Property Type 기록

* 동일 시점에서도 상이한 efficiency 및 rating 기록 존재

예시:

```text
Rating: E → C
Efficiency: 51 → 73
```

### 해석

* EPC 평가 입력 정보의 일관성 문제 가능성

* 특정 난방 방식에 대한 구조적 평가 편향 가능성

* UPRN 매핑 및 평가 데이터 관리 문제 가능성 확인

# Final Conclusion

## 동일 건물에서도 등급 변화 존재

→ 약 **15%에서 2단계 이상 변화**

## 건물 자체는 상대적으로 안정적

→ 면적 변화는 작음

## efficiency 점수는 크게 변동

→ 평균 약 30 수준 변화

## 모델 기반 anomaly 존재

→ 건물 특성 대비 실제 EPC 등급이 비정상적으로 낮은 사례 발견

## electric heating 기반 anomaly 반복 확인

→ 특정 heating 구조에서 EPC 평가 불일치 가능성 존재

## 최종 결론

* 동일 건물에 대해 물리적 특성은 상대적으로 안정적이었으나,

* efficiency 점수와 EPC 등급은 큰 폭으로 변동하는 사례가 존재하였음.

* 또한 모델 기반 분석 결과, 특정 electric heating 기반 건물에서 실제 EPC 등급이 구조적으로 낮게 평가되는 경향이 반복적으로 관찰되었음.

* 이는 EPC 평가 시스템이 일정 수준의 규칙성을 가지면서도, 일부 건물 유형 및 입력 정보에 대해 구조적 inconsistency를 포함할 가능성을 시사함.

# 3. 현재 프로젝트 방향 변화

기존:

```text
효율 점수 예측 → anomaly 탐지
```

현재:

```text
평가 시스템 자체의 일관성 검증
+
모델 기반 hidden inefficiency 탐지
```

→ Model = Validator 역할로 재정의

# 4. 추후 일정

## 4.1 결과 시각화 강화

* Borough 단위 anomaly 분포 시각화

* 지도 기반 EPC inconsistency 분석

## 4.2 최종 결과 정리

* EPC 평가 체계의 구조적 한계 정리

* 정책 및 평가 개선 방향 제안

# 5. 현재 진행 상황 평가

✓ 데이터 구조 이해 완료

✓ 동일 건물 기반 분석 구조 구축 완료

✓ EPC 평가 inconsistency 발견

✓ efficiency 기반 등급 변화 원인 분석 완료

✓ CatBoost 기반 검증 모델 구축 완료

✓ Hidden inefficiency 사례 탐지 완료

✓ Strong anomaly 사례 도출 완료

✓ 핵심 인사이트 도출 완료
