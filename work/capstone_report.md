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

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
