# 프로젝트 최종보고 | Building Energy Efficiency Analysis: London EPC

최은혜(데이터 전처리 및 모델 구축), 김준서(주제 분석 및 데이터 수집)

## 0. 연구 배경

건물 부문은 에너지 소비와 탄소 배출의 주요 원인 중 하나로, 각국 정부는 건물의 에너지 효율 향상을 위한 다양한 정책을 시행하고 있다. 영국에서는 EPC(Energy Performance Certificate)를 통해 건물의 에너지 성능을 A~G 등급으로 평가하며, 해당 결과는 부동산 거래, 임대, 에너지 정책 수립 등 다양한 분야에서 활용된다.

EPC 등급은 건물의 에너지 효율을 나타내는 대표적인 지표로 사용되고 있지만, 동일 건물에 대한 반복 평가 결과가 항상 일관적인지에 대한 검증은 상대적으로 부족하다. 만약 동일한 건물에 대해 시기에 따라 상이한 평가 결과가 나타난다면, 이는 실제 건물 성능 변화뿐 아니라 평가 과정의 변동성이나 데이터 품질 문제와도 관련될 수 있다.

기존 연구들은 EPC 등급 예측이나 에너지 소비량 추정과 같은 예측 성능 향상에 주로 집중해 왔다. 반면, EPC 평가 결과 자체의 신뢰성과 일관성을 분석하고 검증하는 연구는 상대적으로 제한적이다.

## 1. 프로젝트 개요

본 프로젝트는 London EPC (Energy Performance Certificate) 데이터를 활용하여 건물 에너지 효율 등급(EPC Rating)의 신뢰성과 일관성을 분석하는 것을 목표로 한다.

기존 연구들이 에너지 효율 등급 예측 정확도 향상에 집중한 것과 달리, 본 프로젝트는 EPC 평가 결과 자체를 분석 대상으로 설정하였다. 동일 건물에 대한 반복 평가 결과와 머신러닝 기반 예측 결과를 비교함으로써 평가 시스템의 일관성(consistency)과 신뢰성(reliability)을 검증하고, 구조적으로 불일치하는 사례(Hidden Inefficiency)를 탐지하였다.

---

# 2. 데이터셋

## Dataset

* Source: UK Government Open Data
* Data: Domestic Energy Performance Certificates (EPC)
* Region: London Boroughs
* Format: Borough별 폴더 내 CSV 파일

```text
all-domestic-certificates/
 ├── domestic-E09xxxx/
      └── certificates.csv
```

---

## 분석 변수

### 건물 식별

* UPRN
* LODGEMENT_DATE

### 에너지 평가

* CURRENT_ENERGY_RATING
* CURRENT_ENERGY_EFFICIENCY
* CO2_EMISSIONS_CURRENT

### 건물 특성

* TOTAL_FLOOR_AREA
* PROPERTY_TYPE
* BUILT_FORM
* CONSTRUCTION_AGE_BAND
* TENURE
* MAINHEAT_DESCRIPTION

---

# 3. 데이터 전처리

초기 데이터에서 분석 목적과 관련된 핵심 변수만 선택하였다.

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

* UPRN 결측 제거
* 분석 변수 결측 제거
* 동일 건물 반복 평가 분석을 위해 UPRN 기준 2회 이상 등장 건물 추출

결과:

```text
전체 데이터: 8228 rows
반복 평가 데이터: 4786 rows
실제 건물 수: 약 2200개
```
---

# 4. 동일 건물 기준 평가 일관성 분석

동일 건물에 대해 시간에 따른 에너지 등급 변화를 분석하였다.

각 건물별로 최대 등급과 최소 등급의 차이를 계산하여 Rating Gap을 정의하였다.

```python
rating_gap = max(rating) - min(rating)
```

---

## 결과

<img width="549" height="393" alt="image" src="https://github.com/user-attachments/assets/660343ba-edc0-460a-8582-35f3108fb106" />

```text
rating_gap 분포

0 → 1008
1 → 856
2 → 256
3 → 64
4 → 9
5 → 2
```

### 주요 발견

<img width="549" height="393" alt="image" src="https://github.com/user-attachments/assets/b3edce8d-ed85-4b97-b775-ec1a3c6909b9" />
 
약 15%의 건물에서 2단계 이상의 에너지 등급 변화가 발생하였다.

이는 동일 건물에 대해서도 EPC 평가 결과가 항상 일관적이지 않을 수 있음을 시사한다.

---

# 5. Energy Efficiency 점수 변동 분석

동일 건물 내 Energy Efficiency 점수 변화량을 분석하였다.

```python
eff_gap = max(efficiency) - min(efficiency)
```

---

## 결과

```text
평균: 30.9
중앙값: 29
최소: 13
최대: 78
```

### 주요 발견

Energy Efficiency 점수 변화가 클수록 Rating Gap도 증가하는 경향을 확인하였다. 

즉, EPC 등급 변화는 랜덤하게 발생하는 것이 아니라 Efficiency 점수 변화와 직접적으로 연결되어 있었다.

---

# 6. 물리적 특성 안정성 검증

건물 자체가 실제로 변한 것인지 확인하기 위해 면적 변화를 분석하였다.

## 결과

```text
면적 변화

Median: 2.5㎡
75%: 약 7㎡
```

### 주요 발견

대부분의 건물에서 면적 변화는 매우 작았다.

즉, 건물의 물리적 특성은 비교적 안정적인 반면 Energy Efficiency 점수는 크게 변동하는 현상이 관찰되었다.

---

# 7. 머신러닝 기반 검증

## Model as a Validator

본 프로젝트에서는 머신러닝 모델을 단순 예측기가 아닌 평가 시스템 검증 도구(Validator)로 활용하였다.

---

## 모델 구축

Leakage를 방지하기 위해 CURRENT_ENERGY_EFFICIENCY를 제외하였다.

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

---

# Model Architectiure

<img width="579" height="538" alt="image" src="https://github.com/user-attachments/assets/d3d61364-bad7-4831-9365-3cdfb7f861dd" />

---

## 모델

* CatBoostClassifier (범주형 칼럼 처리에 용이)
* Multi-class Classification (A~G)

---

## 성능

<img width="583" height="547" alt="image" src="https://github.com/user-attachments/assets/e7b0632a-52fb-4b8b-be4b-68fc97d00b98" />

대부분의 오차는 인접 등급(B↔C, C↔D) 사이에서 발생하였으며, 극단적인 오분류는 거의 관찰되지 않았다.

```text
Accuracy ≈ 0.72
```

### 해석

건물 특성만으로도 EPC 등급을 상당 수준 예측할 수 있었으며, EPC 평가 체계가 일정한 규칙성을 가진다는 사실을 확인하였다.

---

# 8. Feature Importance 분석

CatBoost Feature Importance 결과:

```text
1. CO2_EMISSIONS_CURRENT
2. MAINHEAT_DESCRIPTION
3. TOTAL_FLOOR_AREA
4. CONSTRUCTION_AGE_BAND
```

### 주요 발견

* CO2 배출량이 가장 중요한 변수
* 난방 방식이 EPC 등급 결정에 큰 영향
* 건물 연식 또한 중요한 요인

이는 EPC 평가 체계가 에너지 소비 및 난방 구조에 강하게 의존함을 보여준다.

---

# 9. Hidden Inefficiency 탐지

모델 예측 결과와 실제 EPC 등급 간 차이를 이용하여 Hidden Inefficiency를 탐지하였다.

정의:

```text
모델 예측 등급 > 실제 EPC 등급
```

---

## 결과

<img width="686" height="644" alt="image" src="https://github.com/user-attachments/assets/967e8059-eb81-4247-95ce-1d5b7cfc25bd" />

총 18개의 Hidden Inefficiency 후보 건물을 발견하였다.

대표적인 특징:

* Electric underfloor heating
* Electric storage heaters
* Community scheme heating
* 오래된 건물
* Flat / Terrace 구조

---

## 해석

모델은 정상 범주로 판단하였지만 실제 EPC에서는 낮은 등급을 받은 건물들이 존재하였다.

이는 특정 난방 방식 또는 평가 입력 정보에 의해 구조적인 평가 불일치가 발생할 가능성을 시사한다.

---

# 10. Strong Anomaly 사례 분석

Strong Anomaly는 다음 두 조건을 동시에 만족하는 건물로 정의하였다.

* 반복 평가 inconsistency 존재
* Hidden Inefficiency 존재

---

## 결과

총 8개의 Strong Anomaly 건물을 발견하였다.

대표 사례:

<img width="624" height="393" alt="image" src="https://github.com/user-attachments/assets/c7671d1b-0501-4be2-a976-acb0f42ea18d" />

```text
UPRN: 95509767

2013
Rating: D
Efficiency: 61

2016
Rating: G
Efficiency: 15

Area: 동일
Heating: 동일
```

### 해석

건물의 물리적 특성은 유지되었으나 Energy Efficiency 점수와 EPC 등급은 큰 폭으로 변화하였다.

이는 평가 입력 정보, 평가 방식 또는 데이터 관리 과정의 일관성 문제 가능성을 보여준다.

---

# 11. 최종 결론

본 연구에서는 London EPC 데이터를 활용하여 동일 건물에 대한 반복 평가 결과와 머신러닝 기반 예측 결과를 비교 분석하였다. 분석 결과 EPC 등급은 건물 특성과 전반적으로 일관된 관계를 보였으며, CatBoost 모델 또한 약 72%의 예측 성능을 달성하였다.

그러나 동일 건물 기준 약 15%의 사례에서 2단계 이상의 등급 변화가 관찰되었고, 건물의 물리적 특성은 안정적인 반면 Energy Efficiency 점수는 큰 폭으로 변동하였다. 또한 Hidden Inefficiency 분석을 통해 특정 Electric Heating 기반 건물에서 반복적인 평가 불일치가 발견되었으며, Strong Anomaly 사례에서는 동일 건물 내에서도 상이한 평가 결과가 기록되는 현상을 확인하였다. 따라서 EPC는 전반적으로 유효한 평가 체계이지만, 특정 건물 유형 및 평가 조건에서는 일관성과 신뢰성에 대한 추가 검증이 필요함을 확인하였다.

---

# 12. 프로젝트 의의

본 프로젝트는 단순한 에너지 등급 예측을 넘어 공공 평가 시스템 자체의 신뢰성을 데이터 기반으로 검증하였다. 특히 머신러닝 모델을 Validator로 활용하여 평가 결과와 실제 건물 특성 간의 불일치를 탐지하였으며, Hidden Inefficiency 및 Strong Anomaly 사례를 통해 EPC 평가 체계의 구조적 한계를 정량적으로 분석하였다. 이는 향후 에너지 정책 수립, 건물 평가 기준 개선 및 데이터 품질 관리 측면에서 활용될 수 있는 분석 프레임워크를 제시한다.

---

# 13. 정책 개선 방안

## 13.1 EPC 평가 자동 검증 시스템 도입
본 연구에서는 동일 건물에 대해서도 상이한 EPC 평가 결과가 반복적으로 나타나는 사례를 확인하였다. 특히 일부 사례에서는 건물의 면적 및 난방 방식이 거의 동일함에도 불구하고 Energy Efficiency 점수와 EPC 등급이 크게 변화하는 현상이 관찰되었다. 이는 평가 과정에서 입력 정보의 변동성 또는 평가 기준 적용 방식의 차이가 결과에 영향을 줄 가능성을 시사한다. 따라서 EPC 평가 결과 제출 이전에 자동 검증 시스템(Auto Validation System)을 도입할 필요가 있다.

## 13.2 평가자 의존성 감소

현재 EPC 평가는 다양한 건물 특성 정보를 평가자가 직접 수집·입력하는 구조를 가진다. 이 경우 평가자 간 해석 차이 또는 입력 방식 차이로 인해 결과가 달라질 가능성이 존재한다.

따라서 향후에는:

- 디지털 건축 도면
- IoT 센서
- 스마트 미터 데이터
- 자동 면적 산출 시스템

등을 활용하여 평가자의 주관적 판단이 개입되는 영역을 최소화할 필요가 있다.

즉, "사람이 평가를 수행하되, 측정과 계산은 시스템이 담당하는 구조"로 전환함으로써 평가 결과의 일관성을 향상시킬 수 있다. 이는 평가자의 역할을 제거하기 위한 것이 아니라, 측정 및 계산 과정의 표준화를 통해 평가 결과의 재현성과 신뢰성을 향상시키기 위한 접근이다.

# 14. 참고 프로젝트와의 차별성

기존 연구들은 주로 EPC 등급 또는 에너지 소비량을 얼마나 정확하게 예측할 수 있는지에 초점을 맞추었다. 따라서 모델의 예측 성능은 확인할 수 있었지만, 실제 EPC 평가 결과가 얼마나 신뢰할 수 있는지에 대한 검증은 제한적이었다.

본 프로젝트는 머신러닝 모델을 단순 예측기가 아닌 검증 도구(Validator) 로 활용하였다. 특히 동일 건물에 대한 반복 평가 결과를 분석하여 평가 일관성을 정량적으로 측정하였으며, 모델 예측 결과와 실제 EPC 등급 간의 차이를 이용해 Hidden Inefficiency와 Strong Anomaly 사례를 탐지하였다. 이를 통해 기존 예측 중심 연구에서 다루지 않았던 EPC 평가 체계의 신뢰성과 구조적 한계를 분석하였다는 점에서 차별성을 가진다.

# 15. 추후 발전 방향

본 연구는 London EPC 데이터를 활용하여 평가 체계의 신뢰성과 일관성을 분석하였으나, 데이터 범위와 활용 가능한 정보의 제약으로 인해 일부 한계가 존재한다. 향후에는 보다 다양한 데이터와 분석 기법을 활용하여 연구를 확장할 수 있다.

## 15.1 분석 지역 확대

본 연구는 London 지역 EPC 데이터만을 대상으로 수행되었다. 향후에는 영국 전역의 EPC 데이터를 활용하여 지역별 평가 일관성 차이를 비교하고, 특정 지역에서 발생하는 구조적 평가 편향 여부를 분석할 수 있다.

## 15.2 실제 건물 변화 정보 연계

현재 연구에서는 면적, 난방 방식 등의 정보를 활용하여 건물 특성 변화를 확인하였다. 향후에는 리모델링 이력, 설비 교체 기록, 건축 허가 데이터 등 실제 건물 변화 정보를 추가적으로 연계하여 EPC 등급 변화의 원인을 보다 정확하게 분석할 수 있다.

## 15.3 EPC 자동 검증 시스템 구축

본 연구에서 제안한 머신러닝 기반 Validator를 발전시켜 실제 EPC 평가 과정에 적용 가능한 자동 검증 시스템을 구축할 수 있다. 평가 결과가 기존 건물 특성과 크게 다를 경우 자동으로 재검토를 요청함으로써 평가 오류 및 입력 오류를 사전에 탐지할 수 있을 것으로 기대된다.

