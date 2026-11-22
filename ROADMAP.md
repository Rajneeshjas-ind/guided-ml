# guided-ml roadmap

**North star:** a beginner can do machine learning the right way — framing, EDA, cleaning, modelling, evaluation and reporting — by making choices instead of writing code, and learns *why* at every step.

**First user:** a beginner analyst with a tabular dataset (CSV, Excel, Parquet) doing regression or classification.

**Out of scope for v1:** images, text, time-series forecasting, deep learning.

## Principles

- The tool suggests, the human decides. Every suggestion comes with a reason and alternatives.
- Every option is backed by evidence: distribution change and cross-validated score (mean ± std).
- Split before any statistic that drives a decision. All fitting happens on training data only.
- The original data is never modified. Every decision is logged in plain English.
- Every choice shows the matching Python code, so users learn while doing.

## Flow

| # | Stage | What the user decides |
|---|---|---|
| 0 | Frame the problem | Business decision, target, success metric, prediction time |
| 1 | Load + data quality | Which strings mean missing, data-type fixes, duplicates |
| 2 | Target + leakage screen | Which columns are known at prediction time |
| 3 | Split strategy | Random, stratified, time-based or group-based split |
| 4 | EDA | Univariate, bivariate, multivariate views and what to act on |
| 5 | Preprocessing | Missing values, outliers, encoding, scaling, with evidence |
| 6 | Baseline + model comparison | Which models to try and which to keep |
| 7 | Evaluation + error analysis | Metric tied to business cost, where the model fails |
| 8 | Explain + report | Feature importance, decision log, final report |

## Milestones

- [x] **M0 Setup + charter** (target: 23–29 Nov 2026): repo, README, this roadmap, flow agreed
- [ ] **M1 Design frozen** (30 Nov – 13 Dec 2026): a step card for every stage in `docs/DESIGN.md`
- [ ] **M2 v0.1 Data in** (14 Dec 2026 – 10 Jan 2027): stages 0–3 working in a notebook on two datasets, with tests
- [ ] **M3 v0.2 EDA** (Jan – Feb 2027): stage 4 with statistical test guidance and a recommended next action per view
- [ ] **M4 v0.3 Prepare** (Feb – Mar 2027): stage 5 with CV evidence and the decision log
- [ ] **M5 v0.4 Model + evaluate** (Mar – Apr 2027): stages 6–7
- [ ] **M6 v1.0 Report + app** (Apr – May 2027): stage 8, Streamlit app, release

Target windows are estimates, not deadlines. A milestone is done only when its "done when" criteria are met.

## How progress is tracked

- One branch and pull request per piece of work.
- Sunday review: tick milestones here, add a short note to `docs/LOG.md`.
