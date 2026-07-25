# 📊 Credit Default Risk Modeling — Système de Scoring Crédit

> Logistic regression-based credit scoring model to predict client default probability — built on a real banking dataset with class imbalance handling, Information Value feature selection, and full model evaluation.

**Author:** Hiba Zormati  


---

## 📌 Project Overview

This project develops a **credit scoring model** to estimate the probability that a bank client will default within a 12-month horizon, based on the SNI database containing financial and demographic client data.

The pipeline covers the full credit risk modeling workflow:
- Data quality control and cleaning
- Exploratory data analysis (EDA)
- Feature engineering and outlier treatment
- Feature selection via **Information Value (IV)**
- Class imbalance handling with **SMOTE**
- **Logistic regression** modeling
- Performance evaluation (AUC, GINI, confusion matrix)
- Coefficient interpretation for operational use

---

## 🎯 Key Results

| Metric | Value | Interpretation |
|---|---|---|
| AUC | 0.7149 | Acceptable discrimination |
| GINI | 0.4298 | Moderate discriminating power |
| Accuracy | 97.85% | High (class-imbalance driven) |
| Recall (non-default) | 99.88% | Excellent |

---

## 🗂️ Repository Structure

```
credit-scoring-default-risk/
│
├── gestionderisque.ipynb          # Main notebook: full pipeline
├── gestion_de_risque.pdf          # Project report (French)
│
├── data/
│   └── base_SNI.xlsx              # Source dataset (not included — see note below)
│
└── README.md
```

> ⚠️ **Data privacy:** The `base_SNI.xlsx` dataset contains banking client data and is **not included** in this repository. The notebook is structured so that all steps are reproducible once the dataset is provided.

---

## ⚙️ Tech Stack

| Purpose | Tool |
|---|---|
| Language | Python 3.x |
| Data manipulation | `pandas`, `numpy` |
| Visualization | `matplotlib`, `seaborn` |
| Modeling | `scikit-learn` |
| Class imbalance | `imbalanced-learn` (SMOTE) |
| Environment | Google Colab / Jupyter Notebook |

---

## 🚀 Getting Started

### 1. Clone the repository
```bash
git clone https://github.com/your-username/credit-scoring-default-risk.git
cd credit-scoring-default-risk
```

### 2. Install dependencies
```bash
pip install pandas numpy scikit-learn imbalanced-learn matplotlib seaborn openpyxl
```

### 3. Add the dataset
Place your `base_SNI.xlsx` file (with sheet `base_sni`) inside the `data/` folder, then update the file path in the notebook:
```python
file_path = 'data/base_SNI.xlsx'
```

### 4. Run the notebook
Open `gestionderisque.ipynb` in Jupyter or Google Colab and run all cells in order.

---

## 🔄 Pipeline Description

### Step 1 — Data Loading & Quality Control
- Dataset: **5,752 observations × 29 variables**
- 27 duplicate rows removed → **5,725 final observations**
- High missingness variables identified and handled:

| Variable | Missing Rate | Treatment |
|---|---|---|
| Code tribunal | 89.0% | Dropped |
| Interdiction chèque | 98.8% | Dropped |
| Profession | 79.8% | Replaced with "Inconnu" |
| Type de revenu | 34.4% | Replaced with "Inconnu" |
| Revenus mensuels | 39.4% | Replaced with median |

### Step 2 — Feature Engineering
- **Age** computed from date of birth; grouped into age bands (18–24, 25–34, …, 65+)
- **Flag_defaut** (target variable): 1 if DPD 12M > 90 days, else 0
- Log transformation applied to heavily skewed variables: `CRD`, `Solde moyen`, `Revenus mensuels`, `Engagement`, `Solde arrêté`
- Winsorization (1st–99th percentile) for `nb_pret` and `CRD/gar`
- Standardization (StandardScaler) for all numerical variables
- One-Hot Encoding for categorical variables (first category dropped to avoid multicollinearity)

### Step 3 — Target Variable Distribution
- **Non-defaulters (0):** 5,600 — 97.9%
- **Defaulters (1):** 116 — 2.1%
- → Severe class imbalance, addressed with SMOTE

### Step 4 — Feature Selection via Information Value (IV)
Only variables with **IV > 0.02** were retained. Top predictors:

| Variable | IV | Predictive Power |
|---|---|---|
| Solde arrêté | 1.887 | Strong |
| Solde moyen | 1.171 | Strong |
| CRD/eng | 0.524 | Medium |
| Maturité | 0.513 | Medium |
| Age relation | 0.384 | Medium |

### Step 5 — Modeling
- Train/test split: **70% / 30%**, stratified
- SMOTE applied on training set only → balanced 50/50 distribution
- Logistic Regression (solver: LBFGS, max_iter: 1000) on 13 selected variables

### Step 6 — Coefficient Interpretation

**Variables increasing default risk:**
- `CRD` (coef: +0.665) — higher outstanding credit → higher risk
- `Engagement` (coef: +0.375) — higher financial commitments → higher risk
- `Age relation` (coef: +0.295) — paradoxically, longer banking relationships show more risk

**Variables reducing default risk:**
- `Maturité` (coef: −0.849) — shorter-term loans carry less risk
- `Solde moyen` (coef: −0.551) — higher average balance → lower risk
- `Garantie max` (coef: −0.310) — presence of guarantees reduces risk

---

## ⚠️ Limitations

- **Zero true positives:** The model predicts no actual defaults on the test set (0 TP, 35 FN) — the decision threshold of 0.5 is too conservative for this imbalance ratio
- **SMOTE limitations:** Generates synthetic samples that may not reflect real default patterns
- **Data quality:** Key variables like `Profession` and `Revenus mensuels` have very high missing rates, limiting predictive power
- **Static model:** No temporal or behavioral variables included

---

## 🔭 Recommended Improvements

1. **Threshold tuning** — lower classification threshold (e.g., 0.2–0.3) to improve recall on the minority class
2. **Try other algorithms** — Random Forest, XGBoost, LightGBM for non-linear patterns
3. **Alternative resampling** — ADASYN, Random Under-Sampling, or cost-sensitive learning
4. **Add behavioral variables** — transaction history, payment patterns, external credit bureau data
5. **Model monitoring** — track performance drift over time with PSI (Population Stability Index)

---

## 📚 References

- [scikit-learn Documentation](https://scikit-learn.org/)
- [imbalanced-learn (SMOTE)](https://imbalanced-learn.org/)
- Basel II/III Credit Risk Framework
- Siddiqi, N. — *Credit Risk Scorecards* (Wiley, 2006)
