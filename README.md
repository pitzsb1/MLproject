# Building Energy Efficiency Analysis: London EPC

# 0. Research Background

The building sector is one of the largest contributors to global energy consumption and carbon emissions. To improve building energy efficiency, the UK government operates the **Energy Performance Certificate (EPC)** system, which evaluates buildings on a scale from **A (most efficient) to G (least efficient)**. EPC ratings are widely used in property transactions, rental markets, and energy policy development.

Although EPC ratings serve as an important indicator of building energy performance, relatively few studies have examined whether repeated assessments of the **same building** produce consistent results. If the same building receives substantially different ratings over time, such discrepancies may reflect not only actual changes in building performance but also inconsistencies in the assessment process or data quality.

Previous studies have primarily focused on improving prediction accuracy for EPC ratings or estimating building energy consumption. In contrast, relatively little attention has been given to evaluating the **reliability and consistency of the EPC assessment system itself**.

---

# 1. Project Overview

This project investigates the **reliability and consistency of EPC ratings** using the London Energy Performance Certificate dataset.

Unlike conventional studies that focus on improving prediction accuracy, this work treats the EPC assessment itself as the subject of analysis. By comparing repeated EPC assessments of identical buildings with machine-learning-based predictions, we evaluate the **consistency and reliability** of the EPC system and identify structurally inconsistent cases, referred to as **Hidden Inefficiencies**.

---

# 2. Dataset

## Dataset

- **Source:** UK Government Open Data
- **Dataset:** Domestic Energy Performance Certificates (EPC)
- **Region:** London Boroughs
- **Format:** Borough-level CSV files

```text
all-domestic-certificates/
 ├── domestic-E09xxxx/
      └── certificates.csv
```

## Variables

### Building Identification

- UPRN
- LODGEMENT_DATE

### Energy Assessment

- CURRENT_ENERGY_RATING
- CURRENT_ENERGY_EFFICIENCY
- CO2_EMISSIONS_CURRENT

### Building Characteristics

- TOTAL_FLOOR_AREA
- PROPERTY_TYPE
- BUILT_FORM
- CONSTRUCTION_AGE_BAND
- TENURE
- MAINHEAT_DESCRIPTION

---

# 3. Data Preprocessing

Only variables directly related to the research objective were retained.

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

### Preprocessing Steps

- Removed records with missing UPRN values
- Removed missing values from key analysis variables
- Extracted buildings assessed at least twice using UPRN

**Result**

```text
Total records: 8,228
Repeated assessment records: 4,786
Unique buildings: ~2,200
```

---

# 4. Consistency Analysis of Repeated EPC Assessments

The variation in EPC ratings over time was analyzed for each building.

The **Rating Gap** was defined as the difference between the highest and lowest EPC ratings assigned to the same building.

```python
rating_gap = max(rating) - min(rating)
```

## Results

<img width="549" height="393" alt="image" src="https://github.com/user-attachments/assets/a1f4a89c-5a33-4325-a6c6-48df37f77bc2" />

```text
Rating Gap Distribution

0 → 1008
1 → 856
2 → 256
3 → 64
4 → 9
5 → 2
```

### Key Finding

Approximately **15% of buildings experienced EPC rating changes of two or more levels**, suggesting that repeated EPC assessments are not always fully consistent.

---

# 5. Energy Efficiency Score Analysis

Changes in Energy Efficiency scores were analyzed for each building.

```python
eff_gap = max(efficiency) - min(efficiency)
```

## Results

```text
Average : 30.9
Median  : 29
Minimum : 13
Maximum : 78
```

### Key Finding

Buildings with larger Energy Efficiency score changes tended to exhibit larger Rating Gaps, indicating that EPC rating changes are closely associated with changes in Energy Efficiency scores rather than random variation.

---

# 6. Verification of Physical Building Stability

To determine whether the observed rating changes reflected actual structural changes, variations in floor area were examined.

## Results

```text
Median Area Change : 2.5㎡
75th Percentile    : ~7㎡
```

### Key Finding

Most buildings showed minimal changes in floor area, indicating that the buildings themselves remained largely unchanged while Energy Efficiency scores varied substantially.

---

# 7. Machine Learning-Based Validation

## Model as a Validator

Instead of using machine learning solely for prediction, the model was employed as a **validation tool** for evaluating the EPC assessment system.

## Features

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

## Model Architecture

<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/b14cdd08-e268-4191-b9c9-a4348454eb81" />

## Model

- CatBoostClassifier
- Multi-class Classification (A–G)

## Performance

<img width="583" height="547" alt="image" src="https://github.com/user-attachments/assets/633439bd-d98d-4835-b174-14287ce5929c" />

```text
Accuracy ≈ 0.72
```

### Interpretation

Using only building characteristics, the model achieved approximately **72% classification accuracy**, indicating that EPC ratings generally follow consistent structural patterns.

Most prediction errors occurred between neighboring grades (e.g., B↔C and C↔D), while extreme misclassifications were rare.

---

# 8. Feature Importance

Top features identified by CatBoost:

```text
1. CO2_EMISSIONS_CURRENT
2. MAINHEAT_DESCRIPTION
3. TOTAL_FLOOR_AREA
4. CONSTRUCTION_AGE_BAND
```

### Key Findings

- CO₂ emissions were the most influential factor.
- Heating systems significantly affected EPC ratings.
- Building age also played an important role.

These findings indicate that EPC ratings are strongly influenced by energy consumption characteristics and heating systems.

---

# 9. Hidden Inefficiency Detection

Hidden Inefficiencies were identified by comparing model predictions with actual EPC ratings.

Definition:

```text
Predicted Rating > Actual EPC Rating
```

## Results

<img width="686" height="644" alt="image" src="https://github.com/user-attachments/assets/ea869c14-10e9-4079-898e-da4438fd244a" />

A total of **18 Hidden Inefficiency candidates** were identified.

Common characteristics included:

- Electric underfloor heating
- Electric storage heaters
- Community heating systems
- Older buildings
- Flat and terrace housing

### Interpretation

Although the model predicted these buildings to belong to higher EPC categories, they received relatively low official ratings, suggesting potential structural inconsistencies in the assessment process.

---

# 10. Strong Anomaly Case Study

Strong Anomalies were defined as buildings satisfying both of the following conditions:

- Repeated EPC assessment inconsistency
- Hidden Inefficiency

## Results

A total of **8 Strong Anomaly buildings** were identified.

Representative Case:

<img width="624" height="393" alt="image" src="https://github.com/user-attachments/assets/4c60dfd9-a435-4ec8-ace9-40317d2df700" />

```text
UPRN: 95509767

2013
Rating: D
Efficiency: 61

2016
Rating: G
Efficiency: 15

Floor Area: Unchanged
Heating System: Unchanged
```

### Interpretation

Despite no significant changes in physical building characteristics, both the Energy Efficiency score and EPC rating changed dramatically, suggesting possible inconsistencies in assessment inputs or evaluation procedures.

---

# 11. Conclusion

This study evaluated the reliability of the London EPC assessment system by combining repeated building assessments with machine-learning-based validation.

Although the CatBoost model achieved approximately **72% accuracy**, around **15% of buildings exhibited EPC rating changes of two or more levels** across repeated assessments.

Furthermore, Hidden Inefficiency analysis revealed recurring inconsistencies among buildings using electric heating systems, while Strong Anomaly analysis identified buildings whose ratings changed substantially despite stable physical characteristics.

Overall, the EPC system appears to be generally reliable; however, certain building types and assessment conditions require additional validation to ensure consistency.

---

# 12. Project Contribution

Rather than focusing solely on prediction, this project evaluates the **reliability of a public assessment system** using data-driven methods.

By employing machine learning as a **validator**, we identified Hidden Inefficiencies and Strong Anomalies, providing quantitative evidence of structural limitations within the EPC assessment framework.

This analytical framework may support future improvements in energy policy, building assessment standards, and data quality management.

---

# 13. Policy Recommendations

## 13.1 Automated EPC Validation System

Repeated inconsistencies were observed even for identical buildings. An automated validation system could detect suspicious assessment results before official registration, improving the consistency of EPC evaluations.

## 13.2 Reducing Assessor Dependency

Current EPC assessments rely heavily on manually collected information, making results susceptible to differences in assessor interpretation.

Future EPC assessments could integrate:

- Digital building drawings
- IoT sensors
- Smart meter data
- Automated floor-area measurement systems

This approach would standardize measurement and calculation processes while allowing assessors to focus on professional evaluation, ultimately improving the reproducibility and reliability of EPC ratings.

---

# 14. Comparison with Previous Studies

Previous studies primarily aimed to improve the prediction accuracy of EPC ratings or building energy consumption.

In contrast, this project employs machine learning as a **validation framework** rather than a prediction tool.

By combining repeated assessment analysis with machine-learning validation, we quantitatively evaluated EPC consistency, identified Hidden Inefficiencies, and discovered Strong Anomalies, offering insights beyond conventional prediction-focused research.

---

# 15. Future Work

## 15.1 Nationwide Analysis

Future studies may extend the analysis beyond London to compare EPC consistency across the entire UK.

## 15.2 Integration with Building Renovation Data

Incorporating renovation records, equipment replacement histories, and building permit information would provide deeper insights into the causes of EPC rating changes.

## 15.3 Deployment of an Automated Validation System

The proposed validator could be integrated into future EPC assessment workflows, automatically flagging assessment results that deviate significantly from expected ratings based on building characteristics.
