# Machine Learning Approaches to Predicting Cancer Cell Line Sensitivity to Ulixertinib

**COMP4030 Machine Learning Coursework Project — University of Nottingham, 2026**

This repository is a curated portfolio record of a machine learning coursework project investigating whether cancer cell line sensitivity to the drug **ulixertinib** can be predicted from biological and experimental features contained within the **Genomics of Drug Sensitivity in Cancer (GDSC)** dataset.

The problem was formulated as a **binary classification task**, with cell lines classified as either sensitive or resistant according to their measured `LN_IC50` value. Logistic Regression and Random Forest models were trained and evaluated, followed by feature-impact analysis to investigate which characteristics contributed most to predictions.

The accompanying [`COMP4030CWPaper.pdf`](./COMP4030CWPaper.pdf) contains the original academic report, including the related work, methodology, results, limitations, discussion, and references. [`COMP4030CWCode.ipynb`](./COMP4030CWCode.ipynb) contains the complete analysis and modelling workflow used for the project.

---

## Project Overview

Precision medicine aims to use information about an individual's biological characteristics to support more targeted treatment decisions. In oncology, predicting whether cancer cells are likely to respond to a particular drug is one potential application of this principle.

This project investigated whether supervised machine learning could distinguish between cancer cell lines that were relatively sensitive or resistant to **ulixertinib**, using information available within the GDSC dataset.

Due to the project timeframe and the desire to keep the investigation focused, modelling was restricted to a **single drug**. Ulixertinib was selected because it had the largest number of observations in the supplied dataset, providing 1,698 samples before preprocessing.

The workflow was:

1. Load and inspect the GDSC dataset.
2. Select observations corresponding to ulixertinib.
3. Examine the distribution of `LN_IC50` and create a binary sensitivity target using the median as the threshold.
4. Remove identifiers, drug metadata, and response-related variables that were not appropriate as predictive inputs.
5. Remove a high-missingness feature and rows containing remaining missing values.
6. One-hot encode categorical variables.
7. Split the data into training and held-out test sets.
8. Train Logistic Regression as a baseline linear model and Random Forest as a non-linear ensemble model.
9. Evaluate predictions using accuracy, confusion matrices, classification reports, and 10-fold cross-validation.
10. Analyse feature contributions using Logistic Regression coefficients and Random Forest feature importance.

---

## Research Question

> Can machine learning models use biological and experimental characteristics from the GDSC dataset to classify cancer cell lines as sensitive or resistant to ulixertinib?

The project was intended as a focused **proof of concept** rather than a clinical prediction system. The report therefore places particular emphasis on the distinction between predictive performance on cell-line data and the considerably more complex requirements of real clinical decision-making.

---

## Methodology

### Dataset and Drug Selection

The source data contained **242,035 observations covering 286 drugs**. Ulixertinib had the largest number of observations, with **1,698 samples**, and was therefore selected as the single drug investigated.

The response measure used was `LN_IC50`, a logarithmic representation of the concentration required to inhibit 50% of cell growth. Lower values indicate greater sensitivity to the drug.

The `LN_IC50` distribution for ulixertinib was approximately symmetric. A median threshold was therefore used to create the binary target:

- `1` — sensitive (`LN_IC50` below the median)
- `0` — resistant (`LN_IC50` at or above the median)

This produced an exactly balanced target of **849 sensitive and 849 resistant samples** before further preprocessing.

### Data Preparation

The following variables were removed because they were identifiers, drug-specific metadata, or otherwise unsuitable for the intended predictive task:

- `COSMIC_ID`
- `CELL_LINE_NAME`
- `DRUG_ID`
- `DRUG_NAME`
- `LN_IC50`
- `AUC`
- `Z_SCORE`
- `TARGET`
- `TARGET_PATHWAY`

`AUC` and `Z_SCORE` were specifically investigated for correlation with the target. Their high correlation with `LN_IC50` supported removing them to avoid target leakage.

The feature `Cancer Type (matching TCGA label)` was removed because it contained a relatively high proportion of missing values. Remaining rows containing missing values were then removed.

Categorical variables were converted to numerical representations using one-hot encoding. The resulting dataset contained **1,609 samples and 109 predictive features**.

### Models

Two supervised classification models were trained:

**Logistic Regression**  
Used as a relatively simple, interpretable linear baseline.

**Random Forest**  
Used as a non-linear ensemble model capable of learning more complex relationships between the available features.

A fixed random state was used for the train-test split and Random Forest to support repeatability.

### Evaluation

An **80:20 train-test split** was used, with the test set held out as unseen data.

The primary metric was **accuracy**, supported by confusion matrices and classification reports. Because the final classes were approximately perfectly balanced, accuracy provided a useful initial measure without being dominated by a majority class.

Both models were also evaluated using **10-fold cross-validation**. The cross-validation accuracies were compared using a paired t-test to assess whether the observed difference between the models was statistically significant.

Feature-impact analysis was additionally conducted using:

- standardised Logistic Regression coefficients
- Random Forest feature importance

---

## Results

### Model Performance

| Model | Test Accuracy | 10-Fold CV Accuracy |
| --- | ---: | ---: |
| Logistic Regression | 71.43% | 66.19% |
| Random Forest | 72.98% | 67.24% |

The Random Forest achieved slightly higher accuracy on both the held-out test set and cross-validation.

However, the paired t-test found **no statistically significant difference** between the cross-validation accuracies:

> `p = 0.47`

The results therefore suggest similar predictive performance from the two approaches within this experiment rather than a clear advantage for the more complex Random Forest model.

### Feature Analysis

Feature analysis showed that both models used a mixture of **biological and experimental characteristics** when making predictions.

For Logistic Regression, some of the largest-magnitude standardised coefficients included:

- `TCGA_DESC_KIRC`
- `GDSC Tissue descriptor 2_myeloma`
- `TCGA_DESC_BRCA`
- `TCGA_DESC_HNSC`
- `GDSC Tissue descriptor 2_head and neck`
- `Growth Properties_Semi-Adherent`

The Random Forest feature-importance analysis identified a related but not identical set of influential variables.

The difference between the two feature rankings also illustrates that model choice can affect how predictive information is represented and interpreted.

---

## Key Findings

The project provides an initial indication that information contained within the GDSC dataset can be used to classify cancer cell lines according to their observed sensitivity to ulixertinib with **moderate predictive performance**.

The results also demonstrate that:

- A relatively simple Logistic Regression model performed similarly to Random Forest.
- The higher Random Forest accuracy was **not statistically significant**.
- Test-set performance was broadly consistent with cross-validation performance, suggesting that the reported results were not entirely dependent on one particular split.
- Both models identified a mixture of biological and experimental variables as influential predictors.

These findings support the existence of meaningful predictive patterns within the dataset, whilst also showing that drug-response prediction remains a challenging problem.

---

## Limitations and Future Work

The project was deliberately scoped as a focused coursework investigation, and several limitations restrict the conclusions that can be drawn.

### Clinical Translation

The binary sensitive/resistant classification is a simplification of real treatment response. There is no single `IC50` threshold at which a treatment suddenly becomes clinically effective, and real treatment decisions depend on substantially more biological and clinical information.

### Cell-Line Data

The models were trained on **in vitro cancer cell lines**, which may behave differently from tumours in real patients. Consequently, the results should not be interpreted as evidence that the models are ready for clinical use.

### Single-Drug Scope

Only ulixertinib was investigated. Similar performance cannot automatically be assumed for other drugs.

### Model Development

The experiment used relatively straightforward preprocessing and default model configurations. Further work could investigate hyperparameter optimisation, alternative missing-data strategies, deep learning approaches, and regression-based prediction of continuous `IC50` values.

### Biological Interpretation

Some influential features represented experimental conditions rather than biological characteristics. This may limit the biological interpretability of the learned relationships and suggests that future work should prioritise features that would be available in realistic clinical settings.

### Future Directions

The report identifies several natural extensions:

- testing the approach across multiple drugs
- predicting continuous `IC50` values rather than a binary label
- comparing additional machine learning methods
- investigating deep learning approaches
- evaluating alternative missing-data and preprocessing strategies
- validating results using independent datasets
- ultimately moving from cell-line data towards data derived from real patient samples

---

## Technical Stack

| Area | Technologies |
| --- | --- |
| Language | Python |
| Data analysis | pandas, NumPy |
| Visualisation | Matplotlib, Seaborn |
| Machine learning | scikit-learn |
| Statistical testing | SciPy |
| Development environment | Jupyter Notebook |

---

## Repository Guide

| File | Purpose |
| --- | --- |
| [`COMP4030CWCode.ipynb`](./COMP4030CWCode.ipynb) | Complete notebook containing data preparation, modelling, evaluation, visualisation, and feature analysis. |
| [`COMP4030CWPaper.pdf`](./COMP4030CWPaper.pdf) | Original four-page academic report describing the project, methodology, results, discussion, limitations, and references. |
| `GDSC_DATASET.csv` | Required input dataset for the notebook; not included in this portfolio repository. |

The notebook was originally developed with the dataset stored in the same directory and loaded using:

```python
pd.read_csv("GDSC_DATASET.csv")
```

---

## Reproducing the Analysis

The project can be reproduced by obtaining the GDSC dataset used for the coursework and placing the required `GDSC_DATASET.csv` file alongside the notebook.

The notebook can then be opened in Jupyter and executed sequentially.

The main workflow performs:

```text
GDSC dataset
      │
      ▼
Select Ulixertinib
      │
      ▼
Create binary sensitivity target
      │
      ▼
Remove leakage / irrelevant variables
      │
      ▼
Handle missing values
      │
      ▼
One-hot encode categorical variables
      │
      ▼
Train/test split
      │
      ├───────────────┐
      ▼               ▼
Logistic Regression  Random Forest
      │               │
      └───────┬───────┘
              ▼
     Evaluation & analysis
```

---

## Data Availability

The dataset used for this project was obtained from a **Kaggle distribution of the Genomics of Drug Sensitivity in Cancer (GDSC) dataset** and was used for the original academic coursework.

The full dataset is **not included in this portfolio repository**. This keeps the repository focused on the work produced for the project and avoids redistributing a third-party dataset unnecessarily.

The notebook expects the downloaded file to be named:

```text
GDSC_DATASET.csv
```

and placed in the same directory as `COMP4030CWCode.ipynb`.

For the original project, the dataset was downloaded from Kaggle. The underlying GDSC project is maintained by the Wellcome Sanger Institute and collaborators, with its data supplied under stated usage terms. The official GDSC website indicates that its data files are available for internal proprietary research and educational purposes, subject to restrictions; users should therefore obtain the data from the source rather than relying on a redistributed copy.  

For reference, the Kaggle dataset used in the coursework is:

**Alipour, S. (2024), “Genomics of Drug Sensitivity in Cancer (GDSC)”, Kaggle.**

See the project report for the dataset citation and academic references.

---

## Academic Context

This repository is a **curated portfolio representation of completed university coursework**, rather than an actively maintained research project.

The code and report are preserved to demonstrate practical experience with:

- exploratory data analysis
- data cleaning and preprocessing
- categorical feature encoding
- supervised machine learning
- model evaluation and cross-validation
- statistical comparison
- feature-importance analysis
- communicating technical results in an academic format

The accompanying paper provides the full academic context and discussion behind the implementation.

---

## About This Repository

This project was completed as part of the **COMP4030 Machine Learning** module at the University of Nottingham.

The repository contains the original notebook and final academic paper in a curated form suitable for portfolio presentation. The intention is to preserve a clear record of the work undertaken during the MSc programme without unnecessarily reproducing external datasets or unrelated course material.
