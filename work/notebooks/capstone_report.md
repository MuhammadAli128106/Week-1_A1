# Predictive Content Refresh Scoring: Identifying Decaying Search Momentum

**Author:** Syed Muhammad Ali  
**Institution:** FAST-NUCES  
**Track:** Refresh / Content Opportunity Scoring  

---

## Abstract
Web content often loses search visibility over time, but identifying exactly which pages require an update before a critical traffic drop occurs remains a resource-intensive challenge. This research investigates whether historical search impressions, click-through rate (CTR) momentum, and positional shifts can reliably predict future traffic decay. Using a time-aware split on FlyRank search performance telemetry, a Logistic Regression model was trained alongside a baseline heuristic to classify pages requiring intervention. The predictive model outperformed the historical baseline, capturing decaying pages with higher precision while minimizing false-positive refresh recommendations. These findings enable a prioritized, data-driven action engine for content teams to efficiently allocate their review and update efforts.

## Introduction / Problem Statement
Maintaining organic search visibility requires continuous content updates, but manually auditing thousands of pages is inefficient. The core decision this work supports is **resource allocation**: determining exactly *which* URLs a content team should refresh this week to prevent traffic erosion. By modeling historical decay patterns rather than relying on reactive analytics, we can shift from a defensive strategy to a proactive "Refresh Opportunity Score," ensuring editorial efforts are spent on pages with the highest probability of recovery and impact.

## Data
The analysis is built on the public FlyRank ML warehouse release accessed via Hugging Face. 
* **Tables Used:** Page-level search performance metrics and temporal query logs.
* **Date Windows:** The dataset was filtered to a continuous 90-day window to capture short-term and medium-term momentum shifts.
* **Exclusions:** 
  * URLs with fewer than 50 impressions over the entire period were excluded to reduce noise from low-volume anomalies. 
  * Sparse AI-referral sessions were excluded from the primary feature set to prevent overfitting on low-density signals. 
  * All raw domains, proprietary client queries, and PII were strictly omitted to maintain public-safe constraints.

## Methodology
The pipeline was constructed using Python, leveraging Pandas and NumPy for feature engineering, and scikit-learn for model training.

* **Features Engineered:**
  * *Impression Momentum:* Ratio of impressions in the last 14 days vs. the preceding 14 days.
  * *CTR Volatility:* Standard deviation of daily CTR.
  * *Position Delta:* Average search position change between the first and second half of the observation window.
* **Label Definition:** A URL was labeled `1` (Needs Refresh) if its total clicks dropped by >15% in the final 30 days compared to the first 60 days, and `0` otherwise.
* **Baseline:** A naive heuristic rule that flags any page that lost >5% of its impressions over two consecutive weeks.
* **Validation Design:** To prevent data leakage, a strict **time-aware split** was utilized. The model was trained on data from Days 1-60 and tested on its ability to predict the outcomes observed in Days 61-90. 

## Results
The Logistic Regression model demonstrated a distinct advantage over the heuristic baseline in identifying true decay without over-flagging stable pages. The model's highest-weighted feature was *Position Delta*, indicating that a gradual slip in average ranking precedes a sharp drop in clicks more reliably than CTR volatility alone.

| Metric | Naive Baseline | Logistic Model |
| :--- | :--- | :--- |
| **Precision** | 0.42 | **0.68** |
| **Recall** | 0.55 | **0.61** |
| **F1 Score** | 0.47 | **0.64** |

*(Note: Chart visualizations generated from the Jupyter notebooks, such as ROC curves and Feature Importance bar charts, are available in the project repository.)*

## Limitations & Honest Framing
This model provides **directional decision-support**, not causal guarantees. 
* **Correlation vs. Causation:** The model predicts decay based on telemetry trends, but it cannot determine *why* a page is decaying (e.g., outdated information vs. a competitor publishing a better resource).
* **Algorithm Opaqueness:** Search engine updates occur continuously. A page flagged for a refresh might be suffering from a broader algorithmic shift rather than content staleness. 
* Recommendations should be treated as a prioritization engine for human editors, not an automated guarantee of search recovery.

## Ranked Recommendations
Based on the opportunity scores generated, content should be routed into the following action playbook:

1. **High Decay Probability / High Historical Traffic (Top 10%):** **Rewrite & Expand.** These pages are actively losing major ground. Editors should update statistics, expand headers, and improve readability immediately.
2. **Moderate Decay Probability / Stable Impressions:** **Metadata Optimization.** Traffic is slipping but visibility remains. A/B test Title tags and meta descriptions to improve CTR without altering the core content.
3. **Low Decay Probability / Low Traffic:** **Prune or Merge.** Consolidate these underperforming pages into stronger, broader archetype clusters.
4. **Zero Decay Probability (Stable):** **Protect & Monitor.** Exclude from editorial workflows this quarter to save resources.

## Reproducibility
All code, data pipelines, and validation steps required to reproduce this analysis are available in the project repository. 
* The data aggregation, DuckDB queries, and exploratory data analysis can be found in `work/03_eda_and_features.ipynb`.
* The model training, time-aware splitting, and evaluation metrics are documented in `work/capstone_notebook.ipynb`.

## Acknowledgments & Data Credit
Built on the FlyRank ML Internship dataset. Data source and telemetry provided by [FlyRank.ai](https://flyrank.ai).