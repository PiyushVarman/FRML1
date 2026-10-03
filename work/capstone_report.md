# Capstone Report — <your lane>

- **Author:** Piyush Chandra Varman
- **Lane:** Machine Learning
- **Repo:** https://github.com/PiyushVarman/FRML1
- **Date:** 3rd October 2026


> Copy this file to `work/capstone_report.md` and fill it in as you build. The eight
> sections mirror the Pass / Needs-Work rubric axes, so nothing here is optional.

## 0. Abstract
Digital enterprises publish thousands of webpages, making manual maintenance triage operationally unsustainable under limited editorial bandwidth. This work formulates content maintenance as a supervised scoring and ranking problem, evaluating observable historical telemetry (impressions, click-through rates, ranking positions, and update recency) to prioritize review queues. Utilizing an anonymized dataset of 30,000 mature web pages, we train a Random Forest model under honest client-grouped splits to predict content decay and traffic slippage. Our empirical evaluation demonstrates that learned ranking models outperform static heuristic baselines in Precision@50 efficiency. Finally, we translate model outputs into a human-reviewed Content Action Playbook with explicit operational guardrails, establishing a reproducible framework for search intelligence triage.

## 1. Problem framing

**Decision:** Prioritizing which published webpages need content refresh or optimization triage under limited weekly editorial review bandwidth.

**Unit of analysis:** Individual webpage (content_id grouped by client domains).

**Output:** Continuous opportunity score and ranked action queue.

**Human action:** Editors execute full refreshes, snippet optimizations, or pruning.

**Cost of wrong call:** Wasting editor time on healthy pages vs. leaving high-traffic decaying pages unaddressed.

**Why ML helps:** Manual auditing of 30,000+ pages is intractable; ML captures non-linear interactions across impressions, CTR, and staleness.

## 2. Data safety

**Data used:** Anonymized search intelligence telemetry from the FlyRank content refresh warehouse (`content_refresh_anonymized.csv`), covering 30,000 mature pages.

**Excluded columns:** Client names, raw URLs, and revenue metrics excluded for privacy.

**Leakage risks:** Excluded future targets and label-derived fields (`trend_direction`, `trend_pct`). Used client_id only for group splitting (`GroupShuffleSplit`), never as a feature.

**Confirmation:** Confirmed zero client-identifying info in `work/`.

## 3. Baseline

**Transparent rule:** A composite heuristic scoring rule combining percentile ranks of impressions, staleness, position, and inverse CTR.

**Fairness:** Evaluated on the exact same test split and metrics.

**Numbers:** Precision@50 = `0.8200` | ROC-AUC = `0.7450`.

## 4. Model / analysis

**Method:** Supervised Random Forest Classifier (`RandomForestClassifier`) for non-linear robustness.

**Feature list:** `content_age_days`, `days_since_last_update`, `impressions_90d`, `clicks_90d`, `ctr`, `avg_position`.

**Target definition:** Binary indicator identifying high-priority refresh candidates exhibiting traffic decline with significant search volume ($\ge 500$ impressions).

## 5. Evaluation

**Split:** Client-grouped split (`GroupShuffleSplit`, 20% test size) to test out-of-domain generalization.

**Model vs. Baseline:** Random Forest achieved Precision@50 = `0.8450` and ROC-AUC = `0.7820` (outperforming baseline).

**Error analysis:** False positives happen on seasonal/volatile impression spikes; false negatives on niche low-impression pages.

## 6. Interpretation

**Findings:** `days_since_last_update` and `impressions_90d` carry the highest feature importances (high-traffic stale pages are the core bottleneck).

**Negative results:** Short-term CTR variances yielded negligible lift compared to stable 90-day aggregates.

## 7. Recommendation

**Ranked actions:** 1) Full Refresh, 2) Snippet Optimization, 3) Prune / Canonicalize, 4) Continue Monitoring.

**Editor workflow:** Ingested every Monday morning.

**Limits:** Decision-support tool with directional ranking; observational telemetry prevents causal claims.

## 8. Reproducibility

**Commands:**

```
git clone https://github.com/PiyushVarman/FRML1
cd FRML1
pip install -r requirements.txt
```
**Seeds & Environment:** Random seed `42`. Python 3.13 with pandas, scikit-learn, and matplotlib.

**Data Credit:** Built on the FlyRank ML Internship dataset (FlyRank).

---

##  Acknowledgments & Data Credit
Built on the **FlyRank ML Internship dataset**. For more information on search intelligence and enterprise telemetry workflows, visit [FlyRank](https://flyrank.ai).

## 8. Showcase Demo Outline (5 Minutes)
- **Minute 1: The Question & Problem** — Present the real-world FlyRank content triage challenge: manually auditing thousands of stale pages is intractable under limited editorial bandwidth.
- **Minute 2: The Method & Data** — Introduce the anonymized 30,000-page mature telemetry dataset and the Random Forest classification approach using decision-moment features (`content_age_days`, `impressions_90d`, `ctr`, `avg_position`).
- **Minute 3: The Core Chart** — Walk through the action distribution chart (`work/figures/action_distribution.png`), showing how pages break down across Continue Monitoring, Snippet Optimization, Full Refresh, and Pruning.
- **Minute 4: Honest Results** — Present the honest comparison table showing the Random Forest model outperforming the heuristic baseline in Precision@50 (0.8450 vs. 0.8200) under an honest client-grouped split.
- **Minute 5: Ranked Recommendations & Wrap-up** — Showcase the Monday morning actionable triage queue and conclude with strict observational limitations and the FlyRank data credit.

---

### Shareable Cuts for Portfolio & Network

#### 1. Short Social Post (Methodology Focus)
> "Excited to share my capstone research for the FlyRank ML Internship! 🚀 I tackled enterprise content maintenance triage by formulating it as a supervised ranking problem on 30,000 anonymized mature web pages. By training a Random Forest model with honest client-grouped validation splits, we successfully outperformed heuristic baselines in Precision@50 efficiency—turning unstructured search telemetry into an actionable Monday morning editorial playbook. Check out the live paper & code: https://github.com/PiyushVarman/FRML1 #MachineLearning #SearchIntelligence #FlyRank #DataScience"

#### 2. Three-Sentence Employer-Facing Summary
> "I built an end-to-end machine learning triage system for digital publishing platforms that predicts webpage decay and prioritizes maintenance queues. Utilizing an anonymized dataset of 30,000 mature web pages with search telemetry features, I trained a Random Forest classifier validated through strict client-grouped splits. The model achieves superior Precision@50 over heuristic baselines, delivering an automated, human-reviewed Content Action Playbook that optimizes editorial review bandwidth."

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
