# Drug Side-Effect (ADR) Prediction Using Molecular Structure

**A Statistical and Machine Learning Approach Using the SIDER Dataset**

## 1. Project Overview

Adverse Drug Reactions (ADRs) are unwanted or harmful effects associated with the use of medicines. Understanding the relationship between molecular structure and reported side effects is an important research problem in pharmaceutical data analysis.

This project investigates whether molecular fingerprints and physicochemical properties can provide useful information for predicting reported adverse drug reaction categories.

Using the **SIDER (Side Effect Resource) dataset**, the analysis focuses on four selected side-effect categories and compares different molecular representations and machine learning algorithms.

The project combines exploratory data analysis, cheminformatics, feature engineering, statistical inference, cross-validation, model selection and performance evaluation.

## 2. Problem Statement

The objective is to examine whether information extracted from drug molecular structures can help distinguish drugs associated with particular reported adverse drug reaction categories.

Four side-effect categories are selected:

| Abbreviation | Side-effect category |
|---|---|
| HBD | Hepatobiliary Disorders |
| MND | Metabolism and Nutrition Disorders |
| PPPC | Pregnancy, Puerperium and Perinatal Conditions |
| SMP | Surgical and Medical Procedures |

Each category is formulated as a separate binary classification problem:

- **1:** The side-effect category is reported for the drug.
- **0:** The side-effect category is not reported for the drug.

The four prediction tasks are analysed independently rather than using a single joint multi-label prediction model.

## 3. Dataset Description

**Dataset:** SIDER (Side Effect Resource)

**Source:** [DeepChem – SIDER Dataset](https://deepchemdata.s3-us-west-1.amazonaws.com/datasets/sider.csv.gz)

The dataset contains drug molecular structures represented through SMILES (Simplified Molecular Input Line Entry System) and 27 binary side-effect labels.

| Dataset characteristic | Description |
|---|---|
| Observations | 1,427 drugs |
| Variables | 28 columns |
| Molecular representation | SMILES |
| Original side-effect labels | 27 binary variables |
| Selected target variables | 4 |
| Prediction approach | Four separate binary classification tasks |

**Unit of Observation:** Each row represents one drug along with its molecular structure and associated side-effect labels.

### Target Class Distribution

The proportion of positive observations differs across the four selected targets.

| Target | Positive prevalence |
|---|---:|
| HBD | 52.1% |
| MND | 69.8% |
| PPPC | 8.8% |
| SMP | 14.9% |

The relatively low positive prevalence for PPPC and SMP indicates class imbalance.

Therefore, model evaluation considers metrics beyond accuracy, particularly Precision, Recall, F1-score, ROC-AUC, PR-AUC and Matthews Correlation Coefficient (MCC).

## 4. Data Quality Assessment

Initial data checks were performed before feature engineering.

The following were examined:

- Missing values
- Duplicate records
- Duplicate SMILES representations
- Validity of molecular structures

**Data validation results:**

| Check | Result |
|---|---:|
| Missing values | 0 |
| Duplicate rows | 0 |
| Duplicate SMILES | 0 |
| Invalid SMILES | 0 |

All 1,427 molecular structures were successfully parsed using RDKit.

## 5. Feature Engineering

Since machine learning algorithms require numerical inputs, molecular structures were transformed into numerical representations using RDKit.

Two types of molecular features were investigated.

### A. Molecular Fingerprints

Four fingerprint representations were generated, each with 2,048 bits:

1. **Morgan Fingerprints:** Represent local molecular environments around atoms.
2. **AtomPair Fingerprints:** Capture relationships between pairs of atoms and their topological distances.
3. **Topological Torsion Fingerprints:** Represent sequences of connected atoms in molecular structures.
4. **Pattern Fingerprints:** Encode structural patterns identified by RDKit.

### B. Physicochemical Descriptors

Two molecular descriptors were calculated:

- Molecular Weight (MolWt)
- Number of Valence Electrons

Exploratory analysis indicated right-skewed distributions. Log2 transformations were therefore applied to both descriptors.

For valence electrons, a constant of 1 was added before transformation to accommodate zero values.

### C. Hybrid Feature Representation

A hybrid feature set was constructed by combining:

- Morgan fingerprints (2,048 features)
- Two transformed and standardized molecular descriptors

**Total: 2,050 features.**

### Final Feature Sets

Six representations were compared:

- Morgan
- AtomPair
- Torsion
- Pattern
- Descriptors
- Hybrid

Standardization was applied to the numerical descriptors. The scaler was fitted using training data only and then applied to the corresponding test data.

## 6. Train-Test Split

The dataset was divided into training and testing subsets.

| Subset | Observations | Proportion |
|---|---:|---:|
| Training | 1,141 | 80% |
| Testing | 286 | 20% |

A fixed random state of 42 was used for reproducibility.

Stratification was performed using the joint combinations of PPPC and SMP labels to help preserve the distribution of these relatively rare target combinations.

The test set was kept separate during baseline comparison, model selection and hyperparameter tuning.

## 7. Exploratory Model Comparison

Three machine learning algorithms were evaluated:

1. Logistic Regression
2. Support Vector Machine (SVM)
3. AdaBoost

The models were compared across all six feature representations and four target categories.

This resulted in:

**6 Feature Sets × 3 Algorithms × 4 Targets = 72 Model Combinations.**

Five-fold Stratified Cross-Validation was performed using the training data.

### Evaluation Metrics

The following metrics were examined:

- Accuracy
- Precision
- Recall
- F1-score
- ROC-AUC
- PR-AUC (Average Precision)

PR-AUC received particular attention because of the varying class prevalence across the target variables.

A prevalence-based reference was also used to interpret PR-AUC. This is important because a higher PR-AUC is not automatically evidence of useful predictive performance.

## 8. Class Imbalance Analysis

Class imbalance was investigated using balanced sample weights.

The objective was to assign greater importance to observations belonging to the minority class during model training.

Weighted and unweighted versions of the same models were compared.

The analysis focused particularly on:

- Minority-class Recall
- Precision
- Matthews Correlation Coefficient
- PR-AUC

The results showed that class weighting did not improve every target equally. Its effect depended on the particular classification problem and evaluation metric.

## 9. Statistical Analysis

In addition to predictive modelling, a separate statistical analysis was conducted using Logistic Regression with Statsmodels.

The purpose was to investigate the association between molecular descriptors and the four selected side-effect categories.

The analysis considered:

- Molecular Weight
- Number of Valence Electrons
- Odds Ratios
- 95% Confidence Intervals
- Likelihood Ratio Tests
- Variance Inflation Factor (VIF)
- Bonferroni Multiple Testing Correction

### Important Statistical Findings

The two transformed molecular descriptors showed a high correlation:

- Correlation coefficient: 0.936
- VIF: Approximately 8.1 for each descriptor

This indicated substantial multicollinearity.

After Bonferroni correction for 12 tests, HBD was the only category showing statistically significant associations in the examined analyses.

The estimated effect was small, with an odds ratio of approximately 1.16 per doubling of valence electrons in the individual descriptor analysis.

These results indicate statistical association, not causation or predictive superiority.

This inferential analysis was conducted separately on all 1,427 observations and was not used to select the machine learning models.

## 10. Model Selection and Hyperparameter Tuning

### Model Selection

The 72 baseline combinations were initially ranked using five-fold cross-validation on the training set.

The top three combinations for each target were subsequently compared using five repeated cross-validation runs with different fold assignments.

Paired bootstrap comparisons were also performed to examine differences between leading candidates.

The comparisons did not establish clear statistical superiority of the first-ranked combination over the other leading candidates.

The final models were selected using the highest mean PR-AUC in repeated cross-validation as the predefined selection criterion.

### Selected Models

| Target | Feature representation | Algorithm |
|---|---|---|
| HBD | Torsion | SVM |
| MND | Torsion | AdaBoost |
| PPPC | Hybrid | SVM |
| SMP | Torsion | SVM |

### Hyperparameter Tuning

RandomizedSearchCV was used to investigate potential hyperparameter improvements for the four selected models.

The tuning procedure was restricted to training data.

Tuned configurations were checked using new cross-validation fold assignments before being retained.

The final specifications were:

- HBD: Default SVM configuration
- MND: Default AdaBoost configuration
- PPPC: Tuned SVM configuration
- SMP: Default SVM configuration

Only the PPPC tuned configuration was retained after the confirmation procedure.

## 11. Final Model Evaluation

The final models were trained using the complete training subset and evaluated once on the held-out test set.

PR-AUC was used as the primary ranking metric.

### Held-Out Test Results

| Target | Final Model | PR-AUC | ROC-AUC |
|---|---|---:|---:|
| HBD | Torsion + SVM | 0.676 | 0.689 |
| MND | Torsion + AdaBoost | 0.741 | 0.572 |
| PPPC | Hybrid + SVM | 0.134 | 0.572 |
| SMP | Torsion + SVM | 0.253 | 0.577 |

### Interpretation

- **HBD:** The model showed the clearest evidence of predictive information above the prevalence-based reference.
- **MND:** Although PR-AUC was relatively high, it was only modestly above its prevalence-based reference.
- **PPPC:** Predictive performance was limited, with substantial uncertainty due to the small number of positive observations.
- **SMP:** Performance was only marginally above its prevalence-based reference on the held-out test set.

The test-set PR-AUC confidence intervals were also examined to understand uncertainty in the estimated performance.

### Repeated Random-Split Analysis

As an additional sensitivity analysis, the selected models were evaluated across ten random 80:20 splits.

| Target | Mean PR-AUC | Standard Deviation |
|---|---:|---:|
| HBD | 0.744 | 0.037 |
| MND | 0.773 | 0.022 |
| PPPC | 0.196 | 0.042 |
| SMP | 0.264 | 0.040 |

These results provide information about variability across different random splits.

However, they are not treated as a replacement for the original held-out test results because model selection was performed using the seed-42 training subset.

## 12. Key Findings

1. Molecular fingerprints and physicochemical descriptors provide different representations of drug structures, and their predictive usefulness varies across side-effect categories.
2. HBD demonstrated the strongest evidence of predictive information among the four selected targets.
3. MND showed relatively high PR-AUC but only modest improvement over its prevalence-based reference.
4. PPPC and SMP remained challenging prediction tasks because of limited positive observations and weaker predictive performance.
5. Class weighting affected minority-class performance differently across target variables.
6. High correlation between molecular descriptors highlighted the importance of examining multicollinearity during statistical inference.
7. Model selection uncertainty was observed because several leading feature-model combinations had comparable cross-validation performance.

## 13. Limitations

The findings should be interpreted in the context of the following limitations.

**Dataset limitations**

- The dataset contains only 1,427 drugs.
- Side-effect labels represent reported associations, not confirmed causal relationships.
- Patient-level information, dosage, timing and clinical mechanisms are unavailable.
- Molecular structures were not subjected to additional standardisation procedures such as desalting, charge neutralisation or largest-fragment selection.

**Feature limitations**

- Some molecular fingerprints were empty for certain drugs.
- Fingerprint bit collisions and saturation may affect molecular representations.
- Only two physicochemical descriptors were investigated.

**Class imbalance and uncertainty**

- PPPC and SMP have relatively few positive observations.
- Their test-set confidence intervals are consequently wide.
- Predictive performance is not equally reliable across all four categories.

**Validation limitations**

- A random train-test split was used rather than a scaffold-based split.
- Structurally similar molecules may therefore occur in both training and testing subsets.
- Performance on genuinely unseen chemical scaffolds may be lower.
- The additional repeated random-split analysis is subject to some model-selection optimism.

## 14. Tools and Technologies

| Category | Technologies |
|---|---|
| Programming | Python |
| Development environment | Google Colab |
| Data manipulation | Pandas, NumPy |
| Visualization | Matplotlib |
| Cheminformatics | RDKit |
| Machine Learning | Scikit-learn |
| Statistical Analysis | Statsmodels |
| Statistical Computation | SciPy |

## 15. How to Run the Project

1. Open `SIDER_ADR_Prediction.ipynb` in Google Colab.
2. Execute the initial installation cell to install RDKit and Statsmodels.
3. Run the notebook cells sequentially.
4. The SIDER dataset is downloaded directly from the DeepChem source.
5. Execute the data validation, feature engineering, statistical analysis, model comparison and evaluation sections.

The notebook generates intermediate result tables and visualizations during execution.

## 16. Conclusion

This project provides a statistical and machine learning investigation of the relationship between drug molecular structures and reported adverse drug reaction categories.

Different molecular fingerprints, physicochemical descriptors and classification algorithms were systematically compared.

The analysis also highlights the importance of class imbalance, multicollinearity, cross-validation, uncertainty estimation and independent test evaluation.

The findings suggest that the predictive information available from molecular structure differs substantially across side-effect categories. HBD showed the clearest predictive signal, whereas the remaining categories demonstrated weaker or more uncertain performance.

Overall, this project demonstrates an end-to-end analytical workflow integrating statistical reasoning, cheminformatics and machine learning while recognising the limitations of the available data.

**Note:** This project is an exploratory academic analysis and is not intended for clinical diagnosis, drug safety assessment or medical decision-making.

---

**Project File:** `SIDER_ADR_Prediction.ipynb`

**Dataset Source:** [DeepChem SIDER Dataset](https://deepchemdata.s3-us-west-1.amazonaws.com/datasets/sider.csv.gz)
