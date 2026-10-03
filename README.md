# Explainable XGBoost Framework for Phishing Detection using SHAP and LIME

## Overview

This repository contains the implementation of an explainable phishing detection framework using XGBoost, SHAP, and LIME. The framework combines machine learning-based phishing website classification with an exploratory, human-centred analysis of questionnaire responses about explanations, to study transparency, interpretability, and self-reported trust in AI-based cybersecurity systems.

The code accompanies the study *Explainable Phishing Detection with SHAP and LIME: An Exploratory Trust Evaluation at a Ghanaian University*.

**What the study does and does not claim.** The classifier results come from a five-seed evaluation on a de-duplicated version of the UCI benchmark and have not been validated on an independent dataset. The survey is cross-sectional, has no no-explanation comparison condition and no pre-explanation trust measure, and its sampling frame and explanation-exposure protocol are not documented. Survey results are therefore descriptive associations between questionnaire items, not evidence that explanations caused any change in trust.

## Methods

The framework has two parallel analytical streams that are never merged into one feature matrix, plus an evaluation stage:

- **Stream A (descriptive, unsupervised):** eight lexical features extracted from PhishTank URL strings, analysed with an Isolation Forest. No train/test split and no held-out prediction.
- **Stream B (supervised):** XGBoost classification of the 30 hand-crafted UCI Phishing Websites features, evaluated on a stratified 80/20 split under five random seeds (42, 7, 123, 2024, 99). Baselines (Logistic Regression, Decision Tree, Random Forest), a SMOTE ablation, a tree-depth ablation (3, 6, 9, 12, 15) and a top-four-feature ablation are included.
- **Explanation layer (Stream B only):** global and local TreeSHAP values from the native XGBoost booster, SHAP rank stability across tree depths, an error-driven SHAP diagnosis, and LIME tabular explanations (fixed configuration, see `Table_LIME_Configuration.csv`).
- **Human-centred evaluation:** descriptive statistics, a paired Wilcoxon test with effect size, a chi-square test with Cramér's V, and a Spearman correlation on a 299-respondent KNUST questionnaire. This data is used for evaluation only, never for training or tuning.

Data cleaning for the UCI branch: 357 rows belonging to label-conflicting feature vectors and 4,977 further exact duplicates are removed before splitting (11,055 → 5,721 rows; 2,766 legitimate, 2,955 phishing). The audit is exact-match only; near-duplicate vectors were not checked. The final reference model is trained without SMOTE at `max_depth = 6`.

## Datasets

| Dataset | Source | Used for | Notes |
| --- | --- | --- | --- |
| PhishTank verified online phishing URLs | PhishTank public feed (`verified_online.csv`) | Stream A | 12,489 rows, all phishing. The feed is regularly updated and has no DOI. The file carries `submission_time` (2008-11-07 to 2023-01-05) and `verification_time` (2011-02-19 to 2023-01-05) per URL; SHA-256 29b9945f…f893. |
| UCI Phishing Websites | UCI Machine Learning Repository, dataset 327, DOI 10.24432/C51W2X, CC BY 4.0 (`uci-ml-phishing-dataset.csv`) | Stream B | 11,055 raw rows, 30 features plus `Result` (−1 = phishing). |
| KNUST user evaluation survey | Collected by the authors (`knust_dataset.csv`) | Human-centred evaluation | 299 respondents, 36 items. **Not included in this repository** for privacy and ethical reasons. |

The two public datasets are not redistributed here. Download them from their sources and place them where the notebook expects them (see Reproducibility). SHA-256 checksums of all three files used in the reported run are in `Table_Data_Provenance_Checksums.csv`; the PhishTank time window is in `Table_PhishTank_Time_Window.csv`.

## Repository Structure

```
.
├── README.md                     This file
├── requirements.txt              Pinned dependencies for the reported run (Python 3.12.13)
├── cecilia-final.ipynb         Single end-to-end notebook: data audit, Stream A, Stream B,
│                                 SHAP/LIME, ablations, survey analysis, output tables and figures
└── revised_outputs/              Figures (*.png) and tables (Table_*.csv) written by the notebook,
                                  plus requirements_pinned.txt and Table_Runtime_Package_Versions.csv
```

Not in the repository: the KNUST survey file and the raw PhishTank and UCI CSVs.

## Technologies Used

Versions are those recorded by the final single-pass run (Kaggle notebook, Linux, Python 3.12.13, CPU only; `Table_Runtime_Package_Versions.csv`):

| Package | Version |
| --- | --- |
| Python | 3.12.13 |
| XGBoost | 3.2.0 |
| SHAP | 0.51.0 |
| LIME | 0.2.0.1 |
| scikit-learn | 1.6.1 |
| imbalanced-learn | 0.14.1 |
| pandas / NumPy / SciPy | 2.3.3 / 2.0.2 / 1.16.3 |
| Matplotlib | 3.10.0 |
| joblib | 1.5.3 |
| openpyxl | 3.1.5 |

## Reproducibility

1. Use Python 3.12 (the reported run used 3.12.13) and install the pinned dependencies:

   ```bash
   pip install -r requirements.txt
   ```

2. Obtain the datasets and set the three paths in the configuration cell near the top of the notebook (`PHISHTANK_PATH`, `UCI_PATH`, `KNUST_PATH`). The notebook checks that each path points at the right kind of file (PhishTank = URL file, UCI = 30-feature classification file, KNUST = survey file) and stops if not. The KNUST file is not public; without it, run Stream A and Stream B only, and skip the survey cells.

3. Run the notebook once, top to bottom, from a fresh kernel (Restart → Run All). Do not re-run individual cells and compare against the reported numbers: several cells depend on state created by earlier ones.

4. Outputs are written to `revised_outputs/` (on Kaggle, `/kaggle/working/revised_outputs`). A manifest of every file written is saved as `Output_Manifest.csv`.

Determinism notes: all five seeds (42, 7, 123, 2024, 99) control the train/test split, SMOTE draws and model initialisation; LIME uses `random_state = 42`. The reported standard deviations measure sensitivity to how the fixed 5,721-row dataset is partitioned, not sampling uncertainty about phishing websites in general. Small numerical differences across library versions or hardware are possible, so pin the versions above.

### Mapping manuscript items to output files

File names below are those listed in `Output_Manifest.csv` from the final run (all outputs are written to `revised_outputs/`).

| Manuscript item | File |
| --- | --- |
| Table 1b (UCI duplicate and label-conflict audit) | `Table_UCI_Duplicate_Audit.csv`, `Table_UCI_Conflicting_Feature_Labels.csv` |
| Table 1c (survey disposition) | `Table_Survey_Disposition.csv` |
| Table 4 (software versions) | `Table_Runtime_Package_Versions.csv` |
| Table 8a (run completion) | `Table_Run_Completion_Audit.csv` |
| Table 9 (contamination sensitivity) | `Table_IsolationForest_Contamination_Sensitivity.csv` |
| Table 10a (five-seed results) | `Table_Final_Vanilla_Mean_SD.csv`, `Table_XGBoost_Original_vs_Deduplicated.csv` |
| Table 10b (confusion matrix, seed 42) | `Table_Confusion_Matrix_Raw.csv` |
| Table 12 (LIME configuration) | `Table_LIME_Configuration.csv` |
| Tables 13, 13b (baseline comparison) | `Table_Controlled_Classifier_Comparison.csv` |
| Table 14a / 14c (SMOTE ablation) | `Table_SMOTE_Ablation_Mean_SD.csv` / `Table_SMOTE_Ablation.csv` |
| Table 14b (tree-depth ablation) | `Table_Tree_Depth_Ablation_Extended_Mean_SD.csv` |
| Table 14d (SHAP depth stability) | `Table_SHAP_Depth_Stability.csv` |
| Table 14e (feature ablation) | `Table_Feature_Ablation_Full_vs_Top4_Mean_SD.csv`, `Table_Feature_Ablation_Top4_By_Seed.csv` |
| Tables 14f, 14g (error-driven diagnosis) | `Table_Q21_Remediation_Plan.csv`, `Table_Error_Driven_SHAP_Profile.csv` |
| Table 15 (trust descriptives) | `Table_KNUST_Trust_Descriptive_Statistics.csv` |
| Table 16 (trust category × preference) | `Table_SHAP_Trust_Categories.csv`, `Table_ChiSquare_CramersV.csv` |
| Section 4.7 paired test and CI | `Table_Paired_Trust_Wilcoxon_Effect_CI.csv`, `Table_Trust_Increased_Proportion_CI.csv` |
| Section 4.9 Spearman correlation | `Table_Spearman_Trust_Correlation.csv` |
| Section 4.10.7 minimum detectable difference | `Table_Minimum_Detectable_Difference.csv` |
| Section 4.10.8 survey integrity screen | `Table_Survey_Screen_Open_Text.csv`, `Table_Survey_Screen_Implausible_Combinations.csv`, `Table_Survey_Screen_Straight_Lining.csv`, `Table_Survey_Screen_Duplicates.csv` |
| Data provenance (Section 3.2) | `Table_Data_Provenance_Checksums.csv`, `Table_PhishTank_Time_Window.csv` |

| Figure | File |
| --- | --- |
| 2 Anomaly score distribution | `Figure_PhishTank_Anomaly_Distribution.png` |
| 3 PhishTank feature variance | `Figure_PhishTank_Feature_Variance.png` |
| 4 Confusion matrix | `Figure_Confusion_Matrix_XGBoost.png` |
| 5 ROC curve | `Figure_ROC_Curve_XGBoost.png` |
| 6 Box plot across seeds | `Figure_MultiSeed_Stability_Boxplot.png` |
| 7 Violin plot across seeds | `Figure_MultiSeed_Stability_Violinplot.png` |
| 9 SHAP summary (beeswarm) | `Figure_SHAP_Summary.png` |
| 10 Global SHAP importance | `Figure_SHAP_Global_Importance.png` |
| 11 LIME explanation, instance #10 (all ten contributions) | `Figure_LIME_Local_Explanation.png` |
| 12 Trust-score distribution | `Figure_SHAP_Trust_Distribution.png` |
| 13 Trust comparison | `Figure_Trust_Comparison.png` |
| 14 Preferred explanation method | `Figure_Preferred_Explanation_Method.png` |

## Limitations

- No external validation: the frozen-model validation hook is not run because no independent 30-feature dataset is configured.
- The UCI benchmark dates from 2014 and may not reflect current phishing practice.
- De-duplication is exact-match only; near-duplicate leakage across the split has not been ruled out.
- Tree depth was selected on the same held-out partitions on which results are reported.
- The survey is a single-institution, non-probability, single-item-measure study with undocumented exposure to explanations. Its two open-text columns hold one identical value in all 299 rows, and some demographic combinations are implausible (e.g. 23 respondents under 18 in staff categories); no respondents were excluded.
- With 299 pairs the study could reliably detect a mean paired trust difference of about 0.32 Likert points; the observed difference was -0.17.
- Figure 2 shows scores as the negative of scikit-learn's `decision_function`, so higher means more anomalous.

## Citation and licence

If you use this code, please cite the accompanying manuscript. The UCI dataset is released under CC BY 4.0 (DOI 10.24432/C51W2X); PhishTank data are subject to the source's published terms. Add a `LICENSE` file for the code before making the repository public.
