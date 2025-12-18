# Financial Health Prediction Challenge - Comprehensive Work Guide

## 📋 Project Overview

**Challenge**: Predict the Financial Health Index (FHI) for micro, small, and medium enterprises (MSMEs) across Southern Africa.

**Geography**: Zimbabwe, Malawi, Eswatini, and Lesotho

**Data Points**: ~9,620 training records with 38 features + 1 target variable

**Target Variable**: Financial Health Index (3 classes: Low, Medium, High) - **Classification Problem**

---

## 📊 Current Notebook Structure

Current workspace notebook now includes:
1. **Data Loading (case-safe)** – robust loading of `Train.csv` / `Test.csv`
2. **Basic EDA** – shapes, dtypes, missingness report, duplicates, target/country plots, quick crosstabs
3. **Preprocessing Pipeline** – numeric mean imputation + categorical most-frequent imputation + one-hot encoding
4. **Model Training + Comparison** – XGBoost vs (fallback) LightGBM, scored on a validation split
5. **Submission Pipeline** – builds `submission_best.csv` to match `SampleSubmission.csv`

**Current State**: Data analysis + baseline modeling + submission generation are implemented.

---

## ✅ What’s Already Implemented (So We Don’t Repeat It)

- Data loading that works on macOS file casing.
- Baseline EDA and sanity checks.
- A preprocessing pipeline:
   - Categorical: `SimpleImputer(strategy="most_frequent")` + `OneHotEncoder(handle_unknown="ignore")`
   - Numeric: `SimpleImputer(strategy="mean")`
- Baseline models:
   - XGBoost and LightGBM (CatBoost not installed in current Python 3.14 environment)
- Model selection by validation `f1_macro` and submission file creation.

**Observed class imbalance** (train): Low ≈ 65%, Medium ≈ 30%, High ≈ 5%.

---

## 🔍 Dataset Characteristics

### Feature Categories

| Category | Count | Examples |
|----------|-------|----------|
| **Owner Demographics** | 2 | owner_age, owner_sex |
| **Business Characteristics** | 8 | business_age_years, business_turnover, business_expenses |
| **Financial Behaviors** | 15 | has_loan_account, has_insurance, has_mobile_money |
| **Attitudes & Perceptions** | 7 | attitude_stable_business_environment, perception_insurance_important |
| **Problems & Risks** | 4 | current_problem_cash_flow, future_risk_theft_stock |
| **Location** | 1 | country |
| **Other** | 1 | covid_essential_service |

### Data Type Distribution
- **Categorical (Binary/Multi-class)**: ~30 features (Yes/No, "Have now"/"Never had", attitudes)
- **Numerical**: ~8 features (income, expenses, turnover, ages)
- **Mixed Content**: Many categorical features with inconsistent formatting

---

## ⚠️ Expected Data Quality Issues

### 1. **Missing Values** (HIGH PRIORITY)
- **Severity**: Widespread across many columns
- **Pattern**: Some features have >50% missing data
- **Root Cause**: Not all questions may apply to all businesses
- **Strategy**: 
  - Use domain knowledge to understand missingness
  - Distinguish between "missing" and "not applicable"
  - Consider creating "missing" as a category for some features

### 2. **Inconsistent Categorical Values**
- Different spellings/formats for the same response (e.g., "Don?t know", "Don't know or N/A")
- Various naming conventions across columns
- **Strategy**:
  - Standardize all categorical values
  - Create mapping dictionaries for normalization

### 3. **Class Imbalance** (EXPECTED)
- Target variable is imbalanced (Low/Medium/High)
- **Strategy**:
   - Check class distribution early
   - Prefer stratified CV + class weights; use resampling carefully

### 4. **Numerical Features**
- May contain outliers (income, expenses, turnover vary widely)
- Different units/currencies across countries (ZWL, MWK, SZL, LSL)
- **Strategy**:
  - Normalize by country or create country-specific features
  - Perform outlier analysis

---

## 🛠️ Improvement Strategies (Phased Approach)

### **PHASE 1: Data Cleaning & Preprocessing (Next Iteration)** ⭐ HIGH IMPACT

**Objectives**:
- Handle missing values appropriately
- Standardize categorical variables
- Create clean feature set

**Tasks**:
1. **Categorical Value Normalization (Do this first)**
   - [ ] Standardize encoding of “don’t know / doesn’t apply” across all columns
   - [ ] Fix apostrophe/encoding issues (e.g., `Don?t know`)
   - [ ] Merge near-duplicates (e.g., `Used to have but don’t have now` vs `Used to have but don't have now`)
   - [ ] Decide whether “Don’t know” should be treated as a real category vs missing

2. **Smarter Missing-Value Handling (Beyond mean/most-frequent)**
   - [ ] For categorical: compare `most_frequent` vs explicit `"Missing"` category (often better)
   - [ ] For numeric: compare mean vs median; consider adding missing-indicator flags
   - [ ] Consider dropping features with extremely high missingness if they add noise

3. **Financial Feature Stabilization**
   - [x] Log-transform heavy-tailed financial columns: `log1p(personal_income)`, `log1p(business_turnover)`, `log1p(business_expenses)`
   - [x] Add ratio features (handle divide-by-zero safely):
     - expenses/turnover, turnover/income, turnover per business age
   - [ ] Country-normalize key money features (z-score or rank within country)

4. **Leakage & Consistency Checks**
   - [ ] Ensure ID is never used as a feature
   - [ ] Ensure preprocessing is fit on train folds only (already true in pipelines; keep it that way)

---

### **PHASE 2: Feature Engineering & Selection (Performance Focused)**

**Objectives**:
- Create meaningful features
- Reduce dimensionality
- Improve model interpretability

**Tasks**:
1. **Domain-Driven Features**
   - [ ] Financial access index: sum/weighted sum of banking access + insurance + mobile money
   - [ ] Formality index: tax compliance + record-keeping + formal credit indicators
   - [ ] Risk index: cash-flow problems + theft risk + worried shutdown

2. **Interaction Features**
   - [ ] owner_age × business_age_years (experience proxy)
   - [ ] cash_flow_problem × has_loan_account (stress-with-leverage)
   - [ ] insurance_important × has_insurance (alignment signal)

3. **Feature Selection Methods**
   - [x] Correlation analysis with target
   - [x] Feature importance from tree-based models
   - [x] Permutation importance
   - [x] Remove highly correlated features (multicollinearity)
   - **Expected Issue**: Need to balance number of features with model complexity

4. **Categorical Encoding**
   - [ ] Keep One-Hot as baseline
   - [ ] Try Ordinal encoding for ordered responses (e.g., “Never had” < “Used to have” < “Have now”)
   - [ ] Consider target encoding for very high-cardinality features (if any appear after cleaning)

---

### **PHASE 3: EDA (Targeted, Not Exploratory-for-its-own-sake)**

**Objectives**:
- Understand relationships
- Identify patterns
- Validate hypotheses

**Tasks**:
1. **Univariate Analysis**
   - [x] Target variable distribution checked (imbalanced)
   - [ ] Identify which features are most predictive for the minority class `High`

2. **Bivariate Analysis**
   - [ ] Compute per-feature uplift for `High` vs others (e.g., `P(High|feature=value)`)
   - [ ] Plot top 10 drivers for `High` to guide feature engineering

3. **Multivariate Analysis**
   - [ ] Correlation matrix between features
   - [ ] PCA for dimensionality reduction
   - [ ] Clustering patterns (do businesses naturally group?)

4. **Country-Level Insights**
   - [ ] Distribution differences by country
   - [ ] FHI distribution by country
   - [ ] Feature importance by country (geographic variations)

---

### **PHASE 4: Modeling (Already Have a Strong Baseline — Now Tune)**

**Objectives**:
- Establish performance baseline
- Understand model behaviors
- Identify what works

**Tasks**:
1. **Data Splitting**
   - [x] Stratified train/valid split is implemented
   - [x] Upgrade to StratifiedKFold CV for more reliable comparison

2. **Baseline Models**
   - [x] Gradient boosting baseline trained (XGBoost / LightGBM)
   - [ ] Optional: add a simple linear baseline for sanity (log-reg) only if needed

3. **Evaluation Metrics**
   - [x] Macro F1 + confusion matrix are in use
   - [ ] Track `High` recall explicitly (if leaderboard rewards minority performance)

4. **Model Interpretation**
   - [ ] Feature importance rankings
   - [ ] SHAP values (feature contribution to predictions)
   - [ ] Decision rules (what drives Low vs. High FHI?)

---

### **PHASE 5: Advanced Modeling & Optimization (Main Work Now)**

**Objectives**:
- Improve performance beyond baseline
- Handle class imbalance
- Fine-tune hyperparameters

**Tasks**:
1. **Handle Class Imbalance**
   - [x] Class imbalance measured
   - [x] Class weights/sample weights used
   - [ ] Try focal-like weighting (custom sample weights) emphasizing `High`
   - [ ] Try SMOTE only after encoding strategy is stable (risk of overfitting)

2. **Advanced Models**
   - [x] XGBoost + LightGBM are running
   - [ ] CatBoost (recommended for messy categoricals) — requires Python 3.11/3.12 env
   - [ ] Consider ordinal encoding + tree models to reduce one-hot noise

3. **Hyperparameter Tuning**
   - [x] Use StratifiedKFold (5 folds) + RandomizedSearchCV
   - [x] Tune specifically for macro-F1
   - [ ] Enable early stopping for boosting models

4. **Ensemble Methods**
   - [x] Blend XGBoost + LightGBM probabilities (simple weighted average)
   - [ ] Only ensemble once single-model tuning is solid

---

### **PHASE 6: Testing & Validation**

**Objectives**:
- Validate on unseen data
- Prepare submission
- Ensure robustness

**Tasks**:
1. **Test Set Evaluation**
   - [x] Best model selection by macro-F1
   - [x] Submission generation matches sample format
   - [ ] Verify prediction distribution vs train (too few/many `High` can be a red flag)

2. **Cross-Validation Analysis**
   - [ ] Report CV scores (mean ± std)
   - [ ] Detect significant variance (overfitting indicators)
   - [ ] Compare train vs. validation performance

3. **Error Analysis**
   - [ ] Identify most common misclassifications
   - [ ] Analyze false positives vs. false negatives
   - [ ] Check performance by country (any geographic bias?)

4. **Final Submission**
   - [x] Format predictions according to SampleSubmission.csv
   - [x] ID alignment preserved
   - [ ] Submit and record leaderboard score + configuration for reproducibility

---

## 🚨 Key Risks & Issues to Monitor

### **High Priority Issues**

| Issue | Impact | Mitigation |
|-------|--------|-----------|
| **Missing Data (>50% in some features)** | Model training failure | Early imputation/removal strategy |
| **Class Imbalance** | Biased predictions toward majority class | Use balanced metrics, SMOTE, class weights |
| **High Dimensionality** | Overfitting, computational cost | Feature selection, dimensionality reduction |
| **Categorical Inconsistencies** | Poor model learning | Standardization before modeling |

### **Medium Priority Issues**

| Issue | Impact | Mitigation |
|-------|--------|-----------|
| **Outliers in Financial Metrics** | Skewed distributions | Log transform, robust scaling |
| **Multicollinearity** | Model instability | Remove correlated features, use regularization |
| **Country-Specific Bias** | Poor generalization | Country-stratified analysis, country features |
| **Feature Importance Concentration** | Model reliance on few features | Add interaction features, domain engineering |

### **Low Priority Issues**

| Issue | Impact | Mitigation |
|-------|--------|-----------|
| **Computational Performance** | Slow iterations | Use sampling for EDA, optimize code |
| **Visualization Clarity** | Interpretation difficulty | Create clear, labeled plots |

---

## 📈 Expected Performance Progression

```
Phase 1-2 (Data Cleaning):    ✓ Enables downstream work
Phase 3 (EDA):                ✓ Completed
Phase 4 (Baseline Models):    ✓ Completed (XGBoost/LightGBM baseline)
Phase 5 (Advanced Models):    → Hyperparameter tuning + better encoding/features
Phase 6 (Optimization):       → Ensembles + CV stabilization
```

**Note**: Actual performance depends heavily on:
- Target class distribution
- Feature quality and completeness
- Model-target relationship strength

---

## 🎯 Success Criteria

✅ **Data Quality Score**: <10% invalid entries after cleaning
✅ **Feature Count**: 20-40 features in final model (balance complexity vs. information)
✅ **Cross-Validation Score**: Low variance (<5% std across folds)
✅ **Test Performance**: Consistent with validation performance (no sudden drops)
✅ **Feature Interpretability**: Top 5 features clearly explain FHI drivers
✅ **Submission Format**: Correctly formatted with all test set predictions

---

## 📚 Quick Reference Checklist

### Data Preparation
- [x] Load all data files
- [x] Check data types and dimensions
- [x] Profile missing values
- [ ] Standardize categorical values (next)
- [x] Handle outliers with transforms (next)
- [x] Create derived features (next)

### Modeling Setup
- [x] Define evaluation metrics (macro-F1, balanced accuracy)
- [x] Create train/validation split
- [x] Establish baseline models
- [x] Generate baseline submission

### Model Improvement
- [x] Handle class imbalance (sample weights)
- [ ] Test feature combinations (planned)
- [x] Tune hyperparameters (next)
- [ ] Compare encoding strategies (next)
- [x] Ensemble top performers (later)

### Final Steps
- [x] Generate test predictions (baseline)
- [x] Validate submission format (baseline)
- [ ] Check prediction distributions (baseline + tuned)
- [x] Create final submission file (baseline)
- [ ] Document approach and insights (track experiments + scores)

---

## 🔗 Recommended Tools & Libraries

- **Data Handling**: pandas, numpy
- **Visualization**: matplotlib, seaborn, plotly
- **Modeling**: scikit-learn, XGBoost, LightGBM
- **Imbalance Handling**: imbalanced-learn (SMOTE)
- **Feature Selection**: scikit-learn SelectKBest, permutation importance
- **Interpretation**: SHAP, eli5
- **Validation**: sklearn.model_selection

---

## 💡 Tips for Success

1. **Start Simple**: Baseline models often reveal the most about data quality
2. **Iterate Rapidly**: Test ideas frequently rather than perfecting each step
3. **Document Everything**: Record what works and why for future reference
4. **Think Domain-First**: Features that make business sense often outperform pure statistical features
5. **Validate Assumptions**: Test hypotheses about feature importance with data
6. **Monitor Leakage**: Ensure no test information leaks into training
7. **Focus on Robustness**: A 75% stable model beats an 85% unstable one

---

## 📖 Next Steps

### Suggested Next Experiments (In Priority Order)

1. **Normalize categorical values** (fix `don’t know` variants and encoding issues), then retrain.
2. **Replace “most_frequent” with explicit “Missing” category for categoricals** and compare macro-F1.
3. **Log + ratio + country-normalized money features**, then retrain. (Partially done: Log + Ratio)
4. **StratifiedKFold CV** for robust model selection (macro-F1 mean ± std). (Done)
5. **RandomizedSearchCV on XGBoost** (depth, min_child_weight, subsample, colsample, reg_lambda, learning_rate, n_estimators with early stopping). (Done)
6. **Try ordinal encoding** for ordered status columns to reduce one-hot explosion.
7. **Probability blending**: weighted average of XGBoost + LightGBM probabilities. (Done)
8. **(Optional) CatBoost** by switching to Python 3.11/3.12 environment.

**Goal**: Improve macro-F1 (and especially `High` recall) while keeping CV stable.

---

**Challenge ID**: Data.org Financial Health Prediction Challenge
**Last Updated**: December 2025
**Status**: Baseline complete; ready for optimization
