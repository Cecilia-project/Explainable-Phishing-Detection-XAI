# Explainable XGBoost Framework for Phishing Detection using SHAP and LIME

## Overview

This repository contains the implementation of an explainable phishing detection framework using XGBoost, SHAP, and LIME. The framework combines machine learning-based phishing URL classification with human-centred explainability analysis to improve transparency, interpretability, and user trust in AI-based cybersecurity systems.

## Methods

The proposed framework consists of the following components:

- Feature extraction from phishing URLs
- XGBoost-based phishing classification
- SHAP-based global and local model explanations
- LIME-based local explanations
- Human-centred trust evaluation using questionnaire responses

## Datasets

The experiments use the following public datasets:

- PhishTank Verified Phishing URLs
- UCI Phishing Websites Dataset

The KNUST user evaluation dataset is excluded from this repository due to privacy and ethical considerations.

## Repository Structure

## Technologies Used

- Python
- XGBoost
- SHAP
- LIME
- Scikit-learn
- Pandas
- NumPy
- Matplotlib

## Reproducibility

To reproduce the experiments:

1. Install the required dependencies:

```bash
pip install -r requirements.txt
