# sider-side-effect-prediction
Predicting drug side-effect categories from chemical structure using SIDER, RDKit and scikit-learn
# Predicting Drug Side-Effect Categories from Chemical Structure (SIDER)

A cheminformatics / machine-learning study that asks a simple question:
**how much of a drug's side-effect profile can be predicted from its molecular structure alone?**

Four SIDER side-effect categories (MedDRA system organ classes) are modelled as separate binary tasks:

| Code | Category | Prevalence |
|------|----------|-----------|
| HBD  | Hepatobiliary disorders | ~52% |
| MND  | Metabolism and nutrition disorders | ~70% |
| PPPC | Pregnancy, puerperium and perinatal conditions | ~9% |
| SMP  | Surgical and medical procedures | ~15% |

## Data
- SIDER (1,427 drugs, 27 side-effect categories), loaded via the DeepChem mirror.
- No nulls, no duplicate rows, no duplicate SMILES, all SMILES parse in RDKit.
- SIDER licence: CC BY-NC-SA 4.0. Please cite Kuhn et al., Nucleic Acids Res. 2016 (SIDER 4).

## Methods
- **Features:** Morgan (r=2), AtomPair, Topological Torsion, RDKit Pattern fingerprints (2048 bits each), 2 descriptors (MolWt, NumValenceElectrons), and a Morgan + descriptors hybrid.
- **Models:** Logistic Regression, SVM (RBF), AdaBoost.
- **Split:** 80/20 stratified on the two rare labels jointly (PPPC, SMP), then repeated over 10 random seeds.
- **Metrics:** PR-AUC (primary, compared with prevalence = random level), skill score, MCC, recall. Accuracy is reported only next to the majority-class baseline because it is misleading on imbalanced labels.
- **Uncertainty:** bootstrap 95% CIs, paired bootstrap for model differences, mean ± SD over 10 splits.
- **Imbalance:** balanced sample weights, compared before/after.
- **Tuning:** RandomizedSearchCV (5-fold, PR-AUC) on the training set only; test set used once.
- **Descriptor analysis:** logistic regression with odds ratios per doubling of MolWt / valence electrons, VIF, Bonferroni threshold.

## Key results (Morgan + SVM, 10 random splits, mean ± SD)

| Label | Prevalence | PR-AUC | Skill | MCC (baseline) | MCC (weighted) |
|-------|-----------|--------|-------|----------------|----------------|
| HBD  | 0.521 | 0.747 ± 0.039 | 0.475 ± 0.062 | 0.390 ± 0.059 | 0.392 ± 0.069 |
| MND  | 0.698 | 0.765 ± 0.031 | 0.208 ± 0.082 | 0.081 ± 0.050 | 0.206 ± 0.056 |
| PPPC | 0.088 | 0.182 ± 0.043 | 0.104 ± 0.048 | 0.017 ± 0.061 | 0.165 ± 0.071 |
| SMP  | 0.149 | 0.270 ± 0.029 | 0.144 ± 0.034 | 0.023 ± 0.049 | 0.123 ± 0.059 |

## Findings
1. Structure carries moderate signal for hepatobiliary disorders and only weak signal for the other three labels.
2. High accuracy on PPPC/SMP is an artefact of imbalance: unweighted models predict almost no positives (recall close to 0).
3. Class weighting trades accuracy for recall and MCC; it does not raise PR-AUC.
4. Hyper-parameter tuning gave no reliable improvement over defaults.
5. Differences between most model pairs are within bootstrap noise; a single split can mislead.
6. MolWt and valence electrons are highly collinear (r = 0.94 on log scale, VIF about 8), so their individual odds ratios should not be interpreted separately.

## Limitations
- Research exercise, not a clinical or regulatory tool.
- SIDER labels come from drug labels (reporting), not proven causality; no dose or exposure information.
- Random split only; a scaffold split would likely be harder.
- No salt stripping or parent-molecule standardisation; some entries are very large.

## Author
Aarushi Singh
