# Fairness Audit Stability Experiments

Code for **“Stable Metrics, Unstable Verdicts: Reproducibility in Algorithmic Fairness Auditing.”**

This repository studies whether categorical fairness-audit conclusions remain reproducible when the same modeling pipeline is retrained, the operational decision threshold changes, or the audit tolerance changes.

## Core Formulation

For retraining realization $s$, operational threshold $t$, and audit tolerance $\tau$,

$$
C(s,t,\tau)=\mathbb{I}[V(s,t)\leq\tau].
$$

All fairness criteria are represented as nonnegative violation scores. For demographic parity ratio (DPR), the symmetric ratio is converted to

$$
V^{\mathrm{DPR}}=1-\mathrm{DPR}_{\mathrm{sym}},
$$

so the same $V\leq\tau$ convention applies to every metric.

The within-tolerance probability is

$$
p(t,\tau)=\Pr_s(C=1),
$$

and pairwise verdict instability is

$$
VI(t,\tau)=2p(t,\tau)\left[1-p(t,\tau)\right].
$$

The Audit Stability Surface is

$$
\mathcal{A}(t,\tau)=p(t,\tau).
$$

With $\epsilon=0.10$, a surface point is classified as:

- **stable-outside:** $p\leq\epsilon$
- **unstable:** $\epsilon<p<1-\epsilon$
- **stable-within:** $p\geq1-\epsilon$

## Experiments

1. **Retraining reproducibility** at the primary operational threshold.
2. **Metric stability versus verdict stability**, including standardized audit-boundary distance.
3. **Audit Stability Surface** over operational thresholds and audit tolerances.
4. **Predictive stability, subgroup support, and audit instability**, including clustered OLS and GEE analyses.

## Data and Models

| Component | Configuration |
|---|---|
| Datasets | Adult, German Credit, COMPAS, ACSIncome |
| Models | Logistic Regression, Random Forest, HistGradientBoostingClassifier |
| Retrainings | 100 per dataset--model combination |
| Split | 70% train, 15% validation, 15% test |
| Operational thresholds | 0.30 to 0.70 in steps of 0.05 |
| Primary threshold | 0.50 |
| Fairness criteria | DPD, DPR, EOD, Equalized Odds |

Dataset sources:

- **Adult:** OpenML
- **German Credit / credit-g:** OpenML
- **COMPAS:** ProPublica public data
- **ACSIncome:** Folktables / U.S. Census American Community Survey

ACSIncome uses California 2018 1-Year ACS data and is capped at 20,000 rows by default in `config.yaml`. The cap is deterministic and approximately target-stratified. Set `max_rows: null` to use the full state sample.

## Analysis Safeguards

The implementation separates mathematical definedness from analysis-support requirements.

Default support thresholds are:

- minimum protected-group size: 20
- minimum positive-label group size: 10
- minimum negative-label group size: 10

Metric-specific analysis requirements are:

- **DPD/DPR:** protected-group support
- **EOD:** protected-group and positive-label support
- **Equalized Odds:** protected-group, positive-label, and negative-label support

The German Credit loader also handles both decoded OpenML sex labels and raw Statlog personal-status codes.

## Repository Structure

The cleaned repository should have the following structure:

```text
fairness_audit_stability/
├── README.md
├── .gitignore
├── requirements.txt
├── config.yaml
├── validate_setup.py
├── run_experiments.py
├── diagnose_quick_test.py
├── analyze_experiments.py
├── generate_figures.py
│
├── src/
│   ├── __init__.py
│   ├── datasets.py
│   ├── metrics.py
│   └── models.py
│
├── results/
│   ├── threshold_metrics.csv
│   ├── run_manifest.csv
│   ├── metadata.json
│   ├── experiment1_retraining_reproducibility.csv
│   ├── experiment2_metric_vs_verdict_stability.csv
│   ├── experiment3_audit_stability_surface.csv
│   ├── experiment3_sensitivities.csv
│   ├── experiment3_step_changes.csv
│   ├── experiment3_step_change_summary.csv
│   ├── experiment4_predictive_vs_audit_stability.csv
│   ├── experiment4_model_comparison.csv
│   ├── experiment4_spearman_correlations.csv
│   ├── experiment4_clustered_ols_contextual.txt
│   ├── experiment4_clustered_ols_mechanism_adjusted.txt
│   ├── experiment4_fractional_gee.txt
│   └── experiment4_binary_unstable_gee.txt
│
└──  figures/
    ├── figure2_retraining_reproducibility.pdf
    ├── figure3a_metric_vs_verdict_stability.pdf
    ├── figure3b_boundary_proximity_vs_instability.pdf
    ├── figure4_audit_stability_surface.pdf
    ├── figure5a_predictive_stability_vs_unstable_share.pdf
    └── figure6_region_composition.pdf

```

Raw downloaded datasets, virtual environments, caches, large intermediate outputs, PNG duplicates, and plotting helper CSVs should remain untracked.

## Setup

Clone the repository and move into it:

```bash
git clone <repository-url>
cd fairness_audit_stability
```

### Windows PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```



### Linux / macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## Run Order

Run the following from the repository root.

### 1. Validate the setup

```bash
python validate_setup.py
```

A successful check prints:

```text
Basic validation passed.
```

### 2. Run a smoke test

```bash
python run_experiments.py --config config.yaml --seeds 3 --datasets adult german
```

### 3. Diagnose the smoke-test output

```bash
python diagnose_quick_test.py
```

This reports dataset/model coverage, subgroup support, validity counts, and undefined fairness counts from `results/threshold_metrics.csv`.

### 4. Run the full experiment corpus

```bash
python run_experiments.py --config config.yaml
```

This creates the primary retraining-level outputs in `results/`, including `threshold_metrics.csv`, `run_manifest.csv`, and `metadata.json`.

### 5. Run the analysis

```bash
python analyze_experiments.py --config config.yaml
```

This produces the Experiment 1--4 analysis tables, sensitivity summaries, statistical-model outputs, and intermediate analysis files.

### 6. Generate publication figures

```bash
python generate_figures.py --results results --out figures
```

The figure script also supports an optional DPI setting:

```bash
python generate_figures.py --results results --out figures --dpi 300
```

## Configuration

The main experiment settings are controlled through `config.yaml`.

The default configuration uses:

```yaml
random_seeds: 100

split:
  train: 0.70
  validation: 0.15
  test: 0.15

primary_operating_threshold: 0.50

analysis:
  instability_epsilon: 0.10
  min_group_n: 20
  min_positive_group_n: 10
  min_negative_group_n: 10
```

The operational-threshold grid, audit-tolerance grids, enabled datasets, enabled models, and ACSIncome row cap can also be changed there.

## Reproducibility

Random seeds and train/validation/test partitions are part of the experimental design. To reproduce the reported analysis, preserve the configuration, protected-group definitions in `src/datasets.py`, model definitions in `src/models.py`, fairness definitions and support rules in `src/metrics.py`, and the analysis code.

The same fitted model is evaluated at every operational threshold for a given retraining realization, and fairness metrics are computed on the held-out test set.

## Interpretation

This repository evaluates **audit-verdict reproducibility**, not whether a fairness criterion or tolerance is normatively appropriate. Stable-within and stable-outside conclusions both describe reproducibility relative to a specified audit rule. The statistical models characterize associations and should not be interpreted as causal effects.

<!--
## Citation

If you use this code, please cite the accompanying manuscript:

```text
Mohammad Ishtiaque Rahman.
“Stable Metrics, Unstable Verdicts: Reproducibility in Algorithmic Fairness Auditing.”
Manuscript under review.
```
-->
