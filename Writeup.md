---
title: "Predicting Well-being Disparities Across Socio-Economic Groups and in Relation to Climate and Environmental Risk"
author: "Armande Aboudrar-Méda, Henry Baker, Giorgio Coppola, Lino Hans Julian Zurmuehl"
date: "Hertie School - Machine Learning Final Project"
geometry: margin=1in
fontsize: 11pt
linestretch: 1.15
toc: true
toc-depth: 2
numbersections: true
colorlinks: true
linkcolor: NavyBlue
urlcolor: NavyBlue
header-includes:
  - \usepackage{booktabs}
  - \usepackage{longtable}
  - \usepackage{array}
  - \usepackage{float}
  - \usepackage{xcolor}
  - \floatplacement{table}{H}
---

\newpage

# Introduction and Theoretical Mechanism

## Research Question

How do climate and environmental hazards affect self-perceived health and well-being disparities across socio-economic groups in older European adults?

## Theoretical Background

Individuals from lower socio-economic status (SES) backgrounds disproportionately reside and work in areas and sectors with poorer environmental conditions due to historical, economic, and social factors. These areas are characterized by higher pollution levels, inadequate housing, and limited access to clean water and air. Such environmental conditions can lead to increased exposure to pollutants and environmental hazards, contributing to higher rates of health issues, including respiratory conditions, cardiovascular diseases, and stress-related illnesses.

This creates a compounding effect on top of other socio-economic vulnerabilities, including limited access to healthcare, poor nutrition, and stress from economic instability. Environmental exposures thus interact with social determinants of health to produce systematic health disparities.

## Hypothesis

We hypothesize that environmental variables significantly improve the prediction of well-being disparities beyond socio-economic factors alone. By comparing models with and without environmental variables, we can measure the improvement in predictive accuracy conferred by environmental factors, highlighting their crucial role in identifying and addressing health inequalities among older adults.

# Data Description

## Data Source

We utilize data from the **Survey of Health, Ageing and Retirement in Europe (SHARE)** and its environmental modules (SHARE-ENV). SHARE is a multidisciplinary, cross-national panel database of micro data on health, socio-economic status, and social and family networks of approximately 140,000 individuals aged 50 or older.

## Data Modules Integrated

Our analysis integrates multiple SHARE data modules:

- **easySHARE_rel8-0-0.dta**: Core SHARE data containing demographic and socio-economic variables
- **life_module_v01.dta**: Environmental exposure data at the life level
- **job_module_v01.dta**: Occupational environmental exposures
- **illness_before_module_v01.dta** and **illness_during_module_v01.dta**: Health history data
- **individual_year_panel_v01.dta**: Yearly individual-level panel data
- **young_age_module_v01.dta**: Youth environmental exposure data
- **yearly_module_v01.dta**: Yearly environmental data

## Units of Analysis

Each row in the dataset represents an individual respondent: an EU resident aged 50+ with recorded health and well-being information. Our final dataset after merging modules and selecting Wave 7 (the most recent wave) contains **77,202 respondents**.

## Target Variable

The label for our classification task is **self-perceived health status (`sphus`)**, a widely validated proxy for well-being in health research. The original variable contains five categorical levels: Excellent, Very Good, Good, Fair, and Poor.

For our analysis, we converted this to a **binary classification problem**:

- **Class 0 (Good Health)**: Excellent, Very Good, Good
- **Class 1 (Poor Health)**: Fair, Poor

This binary formulation allows us to focus on identifying individuals at risk of poor health outcomes, which is more actionable for public health interventions.

## Class Distribution

Our dataset exhibits significant class imbalance:

| Class | Description | Proportion |
|:------|:------------|:-----------|
| 0 | Good Health | ~87% |
| 1 | Poor Health | ~13% |

This imbalance presents methodological challenges addressed in our modeling approach.

# Feature Categories

Our feature set encompasses multiple domains relevant to environmental health disparities.

## Environmental Exposure Variables

**Temperature Metrics:**

- Heating degree days (HDD) and cooling degree days (CDD)
- Extreme temperature exposure (e.g., `s_tg_neg2p5_0_w` for extreme cold)
- Seasonal temperature means and medians

**Air Pollution:**

- PM2.5 and PM10 concentrations and exposure duration
- NO2 (nitrogen dioxide) levels
- O3 (ozone) concentrations
- Cumulative exposure years for each pollutant type

**Radiation:**

- Solar radiation metrics across seasons (fall, winter, spring, summer)
- Occupational radiation exposure averages

**Precipitation:**

- Rainfall exposure frequency and duration
- Total and rolling years of precipitation exposure

**Climate Hazards:**

- Flood-related variables: deaths, displacement, weighted exposure days
- Extreme weather event frequencies

## Health Variables

- Presence of chronic diseases (heart disease, diabetes, respiratory diseases)
- Mental health indicators (depression scores)
- Environment-related health conditions: asthma, respiratory problems, cardiovascular conditions, migraines, emotional distress, fatigue, allergies, and infectious diseases

## Socio-Economic Variables

- Household net worth and income
- Education level (ISCED classification)
- Housing quality indicators
- Location of dwelling (urban/rural classification)
- Occupational and work condition variables

## Demographic Variables

- Age and gender
- Country of residence
- Birth country and citizenship status

# Data Preprocessing

## Data Integration

We merged data from multiple SHARE modules using individual identifiers, aligning environmental exposure data with respondent characteristics and health outcomes.

## Missing Data Handling

Given the complexity of our integrated dataset, we employed the following strategy:

1. **Column removal**: Features with >25% missing values were removed to ensure data quality
2. **Numeric imputation**: Mean imputation for continuous variables
3. **Categorical imputation**: Most frequent value imputation for categorical features

## Feature Engineering

- **One-hot encoding**: Applied to categorical variables to create binary indicator features
- **Feature scaling**: StandardScaler normalization applied to all numeric features

## Data Splitting

We divided our data into three sets:

| Set | Proportion | Purpose |
|:----|:-----------|:--------|
| Training | 70% | Model fitting |
| Validation | 15% | Hyperparameter tuning |
| Test | 15% | Final evaluation |

## Final Feature Set

| Stage | Feature Count |
|:------|:--------------|
| Initial features | 413 |
| After missing value filtering | 266 |
| After one-hot encoding | 495 |
| Optimal subset | ~100 |

# Methodology

## Dimensionality Reduction

Given our high-dimensional feature space, we explored several dimensionality reduction techniques:

**Principal Component Analysis (PCA):**

- Standard PCA on all features
- Grouped PCA maintaining domain interpretability
- Kernel PCA for capturing non-linear patterns
- Finding: ~10 components captured approximately 90% of variance

**Visualization Techniques:**

- t-SNE and UMAP for exploratory visualization
- Used to examine clustering patterns in health status categories

## Feature Selection

We implemented a **grouped feature selection** approach to maintain domain interpretability while reducing dimensionality. Features were grouped by environmental domain:

- Temperature group
- Radiation group
- Precipitation group
- Pollution groups (PM2.5, PM10, NO2, O3)
- Flood exposure group
- Heating/Cooling demand group

Within each group, we applied Random Forest feature importance rankings to identify the most predictive variables while preserving domain structure for policy interpretation.

## Handling Class Imbalance

Given the significant class imbalance (87% vs. 13%), we implemented multiple strategies:

1. **Class weight balancing**: Assigning higher weights to minority class samples
2. **SMOTE**: Generating synthetic samples for the minority class
3. **ADASYN**: Adaptive oversampling focusing on difficult-to-learn samples

## Classification Models

We evaluated multiple classification approaches:

**Logistic Regression:**

- Baseline interpretable model
- Hyperparameter tuning: regularization strength (C: 0.01 to 1000), penalty type (L2)
- GridSearchCV for optimal parameter selection

**Random Forest:**

- Ensemble tree-based method
- Hyperparameters tuned: n_estimators (10-150), max_depth (5, 10, 20, None)
- Used for both classification and feature importance analysis

**XGBoost:**

- Gradient boosting approach
- Parameters: max_depth=5, n_estimators=100, learning_rate=0.1
- Combined with SMOTE for class balance

**Neural Networks:**

- Multi-layer perceptron architectures
- Configurations tested: 10 hidden nodes, 100 hidden nodes, varying depths
- Class weight balancing incorporated

## Model Evaluation

We employed comprehensive evaluation metrics appropriate for imbalanced classification:

- **Accuracy**: Overall correct classification rate
- **ROC-AUC**: Area under the Receiver Operating Characteristic curve
- **Precision**: True positive rate among predicted positives
- **Recall**: True positive rate among actual positives
- **F1-Score**: Harmonic mean of precision and recall

# Results

## Model Performance Comparison

| Model | Accuracy | ROC-AUC | Precision | Recall | F1 |
|:------|:---------|:--------|:----------|:-------|:---|
| **Logistic Regression** | **90.42%** | **0.9068** | 0.69 | 0.45 | 0.55 |
| Random Forest | 90.00% | 0.9018 | 0.62 | 0.43 | 0.51 |
| XGBoost + SMOTE | 89.81% | 0.9043 | 0.63 | 0.50 | 0.56 |
| Balanced RF + ADASYN | 89.00% | 0.8917 | 0.58 | 0.53 | 0.55 |
| Neural Networks | 83-84% | -- | 0.43 | 0.80 | 0.57 |

*Note: Precision, Recall, and F1 reported for the minority class (Poor Health).*

## Best Performing Model

**Logistic Regression** achieved the best overall performance with:

- **Test Accuracy: 90.42%**
- **ROC-AUC: 0.9068**
- Consistent performance across validation and test sets
- Superior interpretability for policy applications

## Neural Network Trade-off

While neural networks achieved lower overall accuracy (83-84%), they demonstrated notably higher recall for the minority class (0.80 vs. 0.45 for logistic regression). This suggests neural networks may be preferable when the priority is identifying all at-risk individuals, even at the cost of more false positives.

## Feature Importance by Domain

| Feature Domain | Top Features | Domain Accuracy |
|:---------------|:-------------|:----------------|
| Temperature | `s_tg_neg2p5_0_w`, `s_tx_25_27p5_mean` | 87.13% |
| Radiation | `avgjob_radiation_fall_mean` | 87.17% |
| Precipitation | `s_prec20_w`, `tot_years_exposure_prec` | 87.17% |
| PM2.5 Pollution | Concentration and exposure metrics | 87.17% |
| PM10 Pollution | Emission and concentration variables | 87.17% |
| NO2 Pollution | Regional NO2 levels | 87.17% |
| O3 (Ozone) | Seasonal ozone concentrations | 87.17% |
| Floods | Deaths, displacement, exposure days | 87.17% |
| Heating/Cooling | `s_CDD_mean`, `s_HDD_w` | 87.16% |

The consistent predictive power across environmental domains supports our hypothesis that environmental factors are important predictors of health disparities.

# Discussion

## Key Findings

1. **Environmental Factors Matter**: Environmental variables significantly contribute to predicting health disparities in older adults. Models incorporating environmental features consistently outperformed baseline approaches.

2. **Multiple Environmental Pathways**: Temperature extremes, air pollution, and climate hazards all demonstrate independent predictive value, suggesting multiple pathways through which environmental conditions affect health.

3. **Class Imbalance Challenges**: The 87/13 class split presents significant modeling challenges. While overall accuracy metrics appear high, detecting individuals at risk of poor health remains difficult.

4. **Model Selection Trade-offs**: Logistic regression provides the best balance of accuracy and interpretability, while neural networks offer better minority class detection at the cost of more false positives.

5. **Interpretability for Policy**: The grouped feature selection approach maintains domain interpretability, enabling clearer translation of findings to policy recommendations.

## Limitations

1. **Cross-sectional Analysis**: Using Wave 7 data provides a snapshot rather than longitudinal trajectories.

2. **Self-reported Health**: While `sphus` is a validated measure, it is subjective and may be influenced by cultural factors.

3. **Geographic Aggregation**: Environmental exposure data is aggregated at regional levels (NUTS regions).

4. **Missing Data**: Imputation may introduce bias, particularly if missingness is not random.

5. **Class Imbalance**: Even with sampling techniques, minority class prediction remains challenging.

# Policy Implications

## Identifying Vulnerable Populations

Our model can help identify older adults most at risk of poor health outcomes due to combined environmental and socio-economic vulnerabilities:

- Residents of areas with high pollution concentrations
- Individuals exposed to temperature extremes
- Communities affected by flood events
- Low-SES populations in environmentally degraded areas

## Resource Allocation

Predictions can guide public health resource allocation by:

- Prioritizing environmental monitoring in high-risk areas
- Directing healthcare resources to populations with elevated risk scores
- Informing urban planning and housing policy decisions

## Environmental Justice Advocacy

Results support advocacy for environmental justice by:

- Quantifying the health burden of environmental exposures
- Demonstrating the intersection of socio-economic and environmental disadvantage
- Providing evidence for regulatory interventions on pollution and climate adaptation

# Conclusion

This project demonstrates that machine learning models can effectively predict well-being disparities among older European adults, achieving over 90% accuracy. Critically, we find that environmental variables---including temperature extremes, air pollution, and climate hazards---provide significant predictive value beyond socio-economic factors alone.

Our best-performing model (logistic regression) achieves a ROC-AUC of 0.9068, indicating strong discriminative ability between individuals with good and poor health status. The grouped feature selection approach maintains interpretability for policy applications while achieving near-optimal predictive performance.

These findings support the integration of environmental factors into public health surveillance and intervention targeting. As climate change intensifies environmental health risks, predictive models incorporating environmental exposures will become increasingly important for identifying and protecting vulnerable populations.

# References

- SHARE - Survey of Health, Ageing and Retirement in Europe. http://www.share-project.org
- SHARE-ENV Documentation. Environmental exposure modules.
- Borsch-Supan, A., et al. (2013). Data resource profile: the Survey of Health, Ageing and Retirement in Europe (SHARE). *International Journal of Epidemiology*.

\newpage

# Appendix

## Technical Implementation

- **Programming Language**: Python 3.11+
- **Key Libraries**: pandas, numpy, scikit-learn, PyTorch, XGBoost, imbalanced-learn
- **Preprocessing**: StandardScaler, OneHotEncoder
- **Model Selection**: GridSearchCV with 5-fold cross-validation
- **Class Balancing**: SMOTE, ADASYN, class_weight parameter

## Code Repository

Key notebooks:

| Notebook | Purpose |
|:---------|:--------|
| `preprocessing.ipynb` | Data cleaning and integration |
| `Combined_Clean_Version_Final.ipynb` | EDA and model development |
| `Log_Reg_Combined_clean.ipynb` | Final logistic regression |
| `neural_net_classweights.ipynb` | Neural network implementations |
| `EDA_RandomForest_FeatureSelection.ipynb` | Feature selection |
| `Grouped_FS.ipynb` | Domain-grouped feature selection |

## Reproducibility

1. Obtain SHARE data access (requires registration at share-project.org)
2. Run `preprocessing.ipynb` to generate the merged dataset
3. Execute model notebooks for training and evaluation
