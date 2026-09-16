```markdown
# asd-hybrid-microbiome-ai

A multimodal hybrid AI architecture (GNN correlation networks + deep autoencoders + tuned stacked ensemble) with SHAP interpretability for ASD biomarker discovery from 16S gut microbiome data[cite: 1].

---

# Multimodal Hybrid AI Framework for ASD Gut Microbiome Classification and Biomarker Discovery

An end-to-end bioinformatics and machine learning pipeline for classifying **Autism Spectrum Disorder (ASD)** from gut microbiome 16S rRNA gene sequencing data[cite: 1]. The framework implements **Microbial Correlation Graph Neural Networks (GNN)**, **Deep Feature Autoencoders**, and an **Optuna-tuned Stacked Ensemble** paired with statistical validation and post-hoc **SHAP interpretability**[cite: 1].

---

## 🔬 Pipeline Architecture

```text
16S rRNA OTU / Genus Abundances
                │
                ▼
┌────────────────────────────────────────────────────────┐
│             Bioinformatics Preprocessing               │
│  • Low-Prevalence Filtering (Prevalence ≥ 10%)         │
│  • Centered Log-Ratio (CLR) Transformation             │
│  • Diversity Profiling (Shannon & Simpson Indices)     │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│             Biostatistical Validation                  │
│  • Alpha Diversity: Mann-Whitney U Test                │
│  • Beta Diversity: Bray-Curtis Distance + PERMANOVA    │
│  • Differential Taxa: Wilcoxon Rank-Sum + BH-FDR       │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│      Stability Feature Selection (5-Fold Stratified)   │
│  • Mutual Information + Lasso (L1) + ReliefF           │
│  • Log-Ratio Feature Engineering                       │
└─────────────┬────────────────────────────┬─────────────┘
              │                            │
              ▼                            ▼
┌───────────────────────────┐ ┌──────────────────────────┐
│   Deep Autoencoder (DL)   │ │  Graph Neural Net (GNN)  │
│  • 128 -> 64 -> 16 Dense  │ │  • Spearman Graph (|r|>0.5)│
│  • BatchNorm + Dropout    │ │  • 2-Layer GCNConv       │
│  • 16-D Latent Embeddings │ │  • Global Mean Pooling   │
│                           │ │  • 16-D Graph Embeddings │
└─────────────┬─────────────┘ └────────────┬─────────────┘
              │                            │
              └──────────────┬─────────────┘
                             ▼
              ┌─────────────────────────────┐
              │  Multimodal Fusion (32-D)   │
              │  [DL Embeddings || GNN]     │
              └──────────────┬──────────────┘
                             ▼
              ┌─────────────────────────────┐
              │  Optuna-Tuned Meta-Stacking │
              │  • LightGBM + XGBoost       │
              │  • CatBoost + RBF-SVM       │
              │  • Meta: Logistic Regression│
              └──────────────┬──────────────┘
                             │
            ┌────────────────┴────────────────┐
            ▼                                 ▼
┌───────────────────────┐         ┌───────────────────────┐
│ Evaluation & Testing  │         │   Clinical SHAP XAI   │
│ • ROC-AUC, F1, Brier  │         │ • TreeExplainer       │
│ • McNemar Test        │         │ • Global & Local      │
│ • 1000x Bootstrap CI  │         │   Biomarker Drivers   │
└───────────────────────┘         └───────────────────────┘

```

---

## 📊 Dataset Summary

* **Dataset Accession**: NCBI GEO `GSE113690` (Human Gut Microbiome with ASD).


* **Cohort Size**: 254 samples (143 ASD, 111 Control).


* **Feature Processing**: 364 raw genus features filtered down to 198 features with $\ge 10\%$ prevalence.


* **Validation Split**: Stratified 80% training / 20% holdout test set.



---

## 📈 Benchmark & Ablation Study (20% Holdout)

| Model Name | Type | ROC-AUC | Accuracy | F1-Score |
| --- | --- | --- | --- | --- |
| **Random Forest** | Classical ML | 0.9397

 | 0.8627

 | 0.8852

 |
| **SVM (RBF)** | Classical ML | 0.9655

 | 0.8824

 | 0.9062

 |
| **XGBoost** | Gradient Boosting | 0.9357

 | 0.8627

 | 0.8814

 |
| **DL (Autoencoder+ML)** | Deep Representation | 0.9169

 | 0.8627

 | 0.8814

 |
| **GNN (Graph+ML)** | Graph Representation | 0.8197

 | 0.7059

 | 0.7619

 |
| **SGHD (Hybrid GNN+DL+ML)** | Multimodal Fusion | 0.9216

 | 0.8824

 | 0.9000

 |

---

### Key Statistical Validation Metrics From Run

* **Final Hybrid ROC-AUC**: 0.922 (95% Bootstrapped CI: **0.811 – 0.992**)


* **McNemar Test ($p$-value)**: **1.0000** (Hybrid vs. Random Forest Baseline)


* **Best Hyperparameters (Optuna)**:


* **XGBoost**: `n_estimators: 82`, `learning_rate: 0.0437`, `max_depth: 10`

* **LightGBM**: `n_estimators: 96`, `learning_rate: 0.0325`, `max_depth: 6`




---

## 🧬 Discovered Biomarkers

Top differential taxa identified via Wilcoxon rank-sum analysis with Benjamini-Hochberg FDR correction ($q < 0.05$):

* **Ruminiclostridium_6**: Log2FC = -2.22, FDR = $9.07 \times 10^{-16}$ (Depleted in ASD)


* **Prevotella_2**: Log2FC = +4.66, FDR = $5.91 \times 10^{-14}$ (Enriched in ASD)


* **Alloprevotella**: Log2FC = +6.86, FDR = $1.44 \times 10^{-10}$ (Enriched in ASD)


* **Comamonas**: Log2FC = -5.86, FDR = $1.82 \times 10^{-9}$ (Depleted in ASD)


* **[Eubacterium] xylanophilum group**: Log2FC = -1.25, FDR = $1.82 \times 10^{-9}$ (Depleted in ASD)


* **Ruminococcaceae UCG-014**: Log2FC = -0.74, FDR = $9.47 \times 10^{-8}$ (Depleted in ASD)



```

```
