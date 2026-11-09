# Proving incrementality at smaller scale

**Dashboard:** https://krisgit00.github.io/incrementality-at-small-scale/ · **Slides:** [docs/deck.pdf](docs/deck.pdf)

A short analysis on Criteo's public randomized uplift dataset (~14M users) showing why advertisers
need a true-lift view alongside attribution before they scale spend.

> An incrementality view doesn't replace attribution. It adds the confidence advertisers need to scale.

## Questions answered
1. **Do ads drive real, measurable lift?** Randomized (intent-to-treat) lift with 95% CIs.
2. **How do attribution and true lift differ?** Exposed-vs-unexposed comparison (attribution-style,
   not randomized) next to the randomized lift, plus effect per exposed user (Wald/IV estimator).
3. **How much scale does a clear read take?** Users needed for 80% power at 5/10/15% holdouts,
   validated by simulating thousands of "advertisers" of different sizes from the data.
4. **Where is lift concentrated?** T-learner uplift model (gradient boosting), held-out decile
   breakdown, Qini curve, and incremental outcomes per 1,000 users when targeting the top 30%.

Credibility checks: covariate balance (standardized mean differences) and A/A placebo tests
(random splits of the control group should show ~5% false positives).

## Run
```bash
python src/ingest.py --source criteo-research-uplift-v2.1.csv.gz   # validate + cache
python src/analysis.py                                              # writes outputs/
```
Small-test simulation and composite estimate (dashboard's sliders):
```bash
python src/small_holdout.py --sims 1000
python src/composite.py --sims 1000
```
`python tools/build_standalone.py` rebuilds the dashboard page in `docs/` from `outputs/dash/`.
`experiments/` holds dropped approaches; `experiments/NOTES.md` explains what was tried and why.
`sql/core_metrics.sql` reproduces the core readout in DuckDB SQL.
`src/make_synthetic.py` makes schema-matched fake data for testing the pipeline only.

## Caveats
- Criteo subsampled the data non-uniformly for privacy, so absolute rates and lift sizes are not the
  original real-world levels. Read results as direction and structure (is there lift, where does it
  concentrate, how do the two views differ), not as industry benchmarks.
- Data pools several advertisers' tests, rebalanced to one 85/15 treatment ratio (v2.1 fixes a v1 leak).
- Criteo display-ad data, not any specific platform's data; the method is what transfers.
- Features are anonymized, so segments are unnamed.
- No cost/revenue fields: results are in conversions and visits, and any dollar figures are labeled assumptions.
- No timestamps: "time to a clear read" is expressed as users needed.

## Data and license
Data: Diemert, Betlei, Renaudin, Amini, "A Large Scale Benchmark for Uplift Modeling," AdKDD & TargetAd
Workshop, KDD 2018. Criteo AI Lab, https://ailab.criteo.com/criteo-uplift-prediction-dataset/
Licensed under Creative Commons BY-NC-SA 4.0: non-commercial use, attribution required, and derived
data must be shared under the same license. The raw data is not included in this repository.
