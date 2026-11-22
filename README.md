# MSCS-634 Project — Deliverable 3: Classification, Clustering, and Pattern Mining

**Course:** Advanced Big Data and Data Mining (MSCS-634-M50)  
**Institution:** University of the Cumberlands  
**Authors:** Ranju Khanal & Anusha Sharma  
**Instructor:** Professor Satish Penmatsa  
**Date:** November 2026  

---

## Overview

This repository contains Deliverable 3 of the Advanced Data Mining project. Building on the cleaned Breast Cancer Wisconsin (Diagnostic) dataset prepared in Deliverable 1 and the regression analysis performed in Deliverable 2, this phase focuses on:

- Supervised **classification** of diagnosis (benign vs. malignant)
- Unsupervised **clustering** to discover natural patient subgroups
- **Association rule mining** to extract clinically interpretable co-occurrence patterns

All modeling follows a leakage-safe workflow with stratified splits and proper evaluation metrics suited to class imbalance.

---

## Dataset

- **Source:** Breast Cancer Wisconsin (Diagnostic) — UCI / scikit-learn
- **Records:** 569
- **Features used:** 10 mean morphological measurements  
  (`mean radius`, `mean texture`, `mean perimeter`, `mean area`, `mean smoothness`, `mean compactness`, `mean concavity`, `mean concave points`, `mean symmetry`, `mean fractal dimension`)
- **Target:** Binary diagnosis (`benign` ≈ 62.7%, `malignant` ≈ 37.3%)
- **Preprocessing:** Data cleaned and validated in Deliverable 1 (no missing values, no exact duplicates, outliers retained after clinical review)

---

## Methodology Summary

### Classification
- Models: Tuned Decision Tree + Tuned k-Nearest Neighbors
- Hyperparameter tuning: 5-fold stratified GridSearchCV (optimized on F1-score)
- Evaluation: Accuracy, Precision, Recall, F1, ROC-AUC, Confusion Matrix, ROC curves
- Feature importance extracted from the Decision Tree

### Clustering
- Algorithm: K-Means on standardized features
- Cluster selection: Elbow method + Silhouette score (k = 2 selected)
- Visualization: 2D PCA projection with cluster labels and ground-truth diagnosis overlay
- Cluster profiling: Mean feature values and diagnosis composition per cluster

### Association Rule Mining
- Discretization: Tertiles (low / medium / high) for each measurement
- Algorithm: Apriori (min_support = 0.10)
- Focus: Rules with consequent = `diagnosis = malignant`
- Ranking: Lift and confidence

---

## Key Results

### Classification Performance (Held-out Test Set)

| Model                  | Accuracy | Precision | Recall | F1    | ROC-AUC |
|------------------------|----------|-----------|--------|-------|---------|
| Decision Tree (tuned)  | 0.921    | 0.923     | 0.857  | 0.889 | 0.928   |
| k-NN (tuned)           | 0.956    | 0.974     | 0.905  | 0.938 | 0.986   |

- Best Decision Tree parameters: `criterion=gini`, `max_depth=3`, `min_samples_leaf=2`
- Best k-NN parameters: `n_neighbors=21`, `weights=uniform`, `p=2`
- Dominant feature (Decision Tree): **mean concave points** (importance ≈ 0.84)

### Clustering
- Optimal k = 2 (supported by both elbow and silhouette)
- Silhouette score ≈ 0.395
- Strong alignment with ground-truth diagnosis:
  - Cluster 0: ~88% benign (smaller size / lower irregularity)
  - Cluster 1: ~96% malignant (larger size / higher irregularity)

### Association Rules
- Strongest rules combine elevated size measures (high radius / area / perimeter) with elevated shape-complexity measures (high concave points / concavity / compactness)
- Many top rules achieve confidence = 1.0 and lift > 2.6
- Patterns are clinically intuitive and actionable

---

## Key Insights

1. The same morphological features that produced near-deterministic regression results in Deliverable 2 also yield highly effective classification and clustering performance.
2. **Mean concave points** is the single most discriminative feature, consistent with exploratory findings from Deliverable 1.
3. Supervised, unsupervised, and rule-based methods converge on the same core story: large + morphologically irregular masses are strongly associated with malignancy.
4. Class imbalance was successfully managed through stratified sampling and F1 / ROC-AUC focused evaluation.

---

## Practical Relevance

- **Clinical triage:** The interpretable Decision Tree can serve as a transparent, low-cost decision aid.
- **Case-based reasoning:** k-NN provides neighbor-similarity support useful in tumor-board discussions.
- **Patient subgroup discovery:** K-Means identifies cohorts that may benefit from differentiated follow-up protocols.
- **Rule-based alerts:** High-lift association rules can be embedded in EHR dashboards as clear “if size is large AND shape is irregular → high suspicion” notifications.

---

## Challenges Encountered & Resolutions

| Challenge                        | Resolution                                                                 |
|----------------------------------|----------------------------------------------------------------------------|
| Class imbalance                  | Stratified train/test splits + F1-based model selection + ROC-AUC reporting |
| k-NN scale sensitivity           | StandardScaler placed inside the modeling pipeline                         |
| Continuous data → association rules | Tertile discretization with robust handling of ties                     |
| Hyperparameter search cost       | Modest, focused grids to keep runtime practical                            |
| Risk of optimistic metrics       | Strict held-out test set used only once; CV performed only on training data |

---

