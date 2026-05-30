# Evaluating the Reliability of Energy Ratings and Detecting Hidden Inefficiencies in Buildings

## Overview

This project investigates the **reliability and consistency of Energy Performance Certificate (EPC) ratings** using the London EPC dataset.

Unlike conventional studies that focus solely on predicting energy efficiency scores, this project treats the EPC rating system itself as the object of analysis.

By combining:

* Repeated assessments of the same buildings (UPRN-based analysis)
* Machine learning validation (CatBoost)
* Hidden inefficiency detection

the project evaluates whether EPC ratings remain consistent across time and whether assigned ratings align with underlying building characteristics.

---

## Key Findings

### Building Rating Consistency

* Approximately **15% of buildings** showed EPC rating changes of **two or more levels**
* Some buildings exhibited rating shifts as large as **5 levels**

```text
Rating Gap Distribution

0 → 1008
1 → 856
2 → 256
3 → 64
4 → 9
5 → 2
```

📌 **Figure 1. Rating Gap Distribution**

> <img width="549" height="393" alt="image" src="https://github.com/user-attachments/assets/63bfd303-539f-4a8f-83fc-9bbfce6f2b4b" />

---

### Energy Efficiency Score Variability

Although building characteristics remained relatively stable, Energy Efficiency scores showed substantial variation.

```text
Average Efficiency Gap: 30.9
Median Efficiency Gap: 29
Maximum Gap: 78
```

📌 **Figure 2. Efficiency Gap vs Rating Gap**

> <img width="562" height="455" alt="image" src="https://github.com/user-attachments/assets/340d6025-8882-4cc7-a940-9563b88a2ed9" />

**Key Insight**

Buildings themselves changed little, but their Energy Efficiency scores often changed dramatically.

---

## Machine Learning as a Validator

Rather than using machine learning solely for prediction, this project uses it as a **validation tool**.

### Model

* CatBoostClassifier
* Multi-class classification (A–G)

### Features

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

### Performance

```text
Accuracy ≈ 0.72
```

Most classification errors occurred between neighboring grades (e.g., B↔C, C↔D), suggesting that the EPC system follows a generally consistent structure.

📌 **Figure 3. Confusion Matrix**

> <img width="583" height="547" alt="image" src="https://github.com/user-attachments/assets/9702e4a0-541b-4811-a5bd-111747068d24" />

---

## Feature Importance

Top predictors of EPC ratings:

1. CO2 Emissions
2. Main Heating Type
3. Total Floor Area
4. Construction Age Band

📌 **Figure 4. Feature Importance**

> <img width="844" height="470" alt="image" src="https://github.com/user-attachments/assets/6aeecd12-036a-47df-8b46-4959266d9664" />

---

## Hidden Inefficiency Detection

Buildings were flagged when:

```text
Predicted Rating > Actual Rating
```

In other words, the building characteristics suggested a higher EPC rating than the one actually assigned.

### Results

* 18 Hidden Inefficiency candidates discovered

Common patterns:

* Electric underfloor heating
* Electric storage heaters
* Community heating schemes
* Older buildings
* Flat / Terrace structures

📌 **Figure 5. Heating Types in Hidden Inefficiency Cases**

> <img width="686" height="644" alt="image" src="https://github.com/user-attachments/assets/f2a23c6f-b1ad-4467-b9d8-437880d60309" />

---

## Strong Anomaly Analysis

Strong Anomalies were defined as buildings satisfying both:

* Repeated EPC inconsistency
* Hidden Inefficiency

### Results

* 8 Strong Anomaly buildings identified

### Representative Case

UPRN: **95509767**

| Year | Rating | Efficiency |
| ---- | ------ | ---------- |
| 2013 | D      | 61         |
| 2016 | G      | 15         |

Building area remained unchanged:

```text
93㎡ → 93㎡
```

Heating system remained unchanged:

```text
Electric underfloor heating
```

Yet the EPC rating dropped from D to G.

📌 **Figure 6. Strong Anomaly Case Study**

> <img width="624" height="393" alt="image" src="https://github.com/user-attachments/assets/20495003-f2ef-4ed7-8aeb-ec4da893f3d9" />

---

## Policy Implications

The findings suggest that EPC ratings are generally meaningful but may contain inconsistencies under specific conditions.

Potential improvements include:

* Automated EPC validation systems
* Machine-learning-assisted quality control
* Reduced dependence on manual assessment inputs
* Integration of digital building records and smart-meter data

Rather than replacing assessors, these systems could improve the reproducibility and consistency of EPC evaluations.

---

## Key Contribution

Most building energy studies focus on prediction.

This project instead asks:

> **Can we trust the ratings themselves?**

By reframing machine learning as a validation tool rather than a prediction tool, the project identifies hidden inefficiencies, strong anomalies, and potential inconsistencies within a real-world public evaluation system.

---

## Repository Structure

```text
.
├── data/
├── docs/
│   ├── mid-report.md
│   └── final-report.md
├── notebooks/
├── results/
└── README.md
```

## Reference Project

This project was initially inspired by:

https://www.kaggle.com/code/michaelfumery/sea-building-energy-and-ghg-prediction

However, unlike traditional prediction-focused studies, this work focuses on evaluating the reliability and consistency of the EPC assessment system itself.
