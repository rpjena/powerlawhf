# Workforce Scaling with AUM in Hedge Funds

**Pareto / Power-Law Analysis of Headcount–AUM Scaling across Hedge Fund Strategies**

This repository contains the full replication package for the paper:

> **Workforce Scaling with Assets Under Management in Hedge Funds: Evidence for Two Organisational Regimes**
> Rudra Jena — Risk Management, CFM LLP

---

## Repository Structure

```
powerlawhf/
├── data/                          # Input data and model outputs
│   ├── insample_funds.csv         # 15 in-sample funds, 92 observations (2005–2024)
│   ├── oos_funds.csv              # 26 out-of-sample funds, 119 observations
│   ├── strategy_metadata.csv      # Strategy codes, labels, colours, markers
│   ├── insample_fit_results.csv   # Generated: OLS fit results for in-sample funds
│   ├── oos_fit_results.csv        # Generated: OLS fit results for OOS funds
│   ├── cluster_assignments.csv    # Generated: K-means cluster labels (K=3)
│   ├── cluster_criterion_scores.csv  # Generated: Silhouette/CH/DB scores vs K
│   └── survivorship_correction.csv   # Generated: Paper vs full-universe fractions
│
├── figures/                       # Plots (PDF + PNG)
│   ├── fig1_loglog.pdf/png        # Log-log scatter (in-sample)
│   ├── fig2_alpha.pdf/png         # Alpha bar chart with bootstrap CIs
│   ├── fig3_timeseries.pdf/png    # AUM & headcount time series
│   ├── fig4_residuals.pdf/png     # Residual diagnostics
│   ├── fig5_clusters.pdf/png      # Cluster partition + silhouette
│   ├── fig6_cluster_evolution.pdf/png  # Temporal cluster evolution
│   ├── fig6b_full_universe.pdf/png     # Full universe (survivorship correction)
│   ├── fig7_trajectories.pdf/png  # Phase-space trajectories
│   ├── fig8_rolling_params.pdf/png    # Rolling parameter estimates
│   ├── fig9_oos_clusters.pdf/png  # OOS cluster assignments
│   ├── fig10_alpha_bands.pdf/png  # OOS alpha bands
│   ├── fig11_oos_residuals.pdf/png    # OOS residuals
│   └── fig12_combined_loglog.pdf/png  # Combined IS+OOS log-log
│
├── notebooks/                     # Jupyter notebooks (fully executable)
│   ├── notebook1_data_exploration.ipynb   # Data loading, EDA, coverage heatmap
│   ├── notebook2_model_fitting.ipynb      # OLS, bootstrap CIs, AIC, LRT, LOO-CV
│   └── notebook3_clustering_oos_figures.ipynb  # K-means, OOS validation, paper figures
│
├── paper/                         # LaTeX source
│   ├── pareto_scaling_hedgefunds.tex   # Main paper (compile from this directory)
│   ├── backtest_section.tex            # OOS validation section
│   ├── survivorship_section.tex        # Survivorship bias section
│   ├── refs.bib                        # BibTeX references
│   └── pareto_scaling_hedgefunds.pdf   # Compiled output
│
└── README.md
```

---

## Data Sources

| Source | Coverage | Uncertainty |
|--------|----------|-------------|
| Man Group Annual Reports (audited) | 2006–2024 | < 2% |
| SEC Form ADV Item 5.B | 2005–2024 | ± 15% (est.) |
| Bloomberg Intelligence | Selected years | ± 15% (est.) |
| Pensions & Investments AUM Rankings | Selected years | ± 15% (est.) |
| Hedgeweek | Selected years | ± 15% (est.) |

**Strategies:** `Q` = Systematic Quant · `P` = Pod Shop/Multi-Manager · `M` = Global Macro · `F` = Fundamental L/S · `A` = Activist

---

## Model Summary

The core model is a Pareto power law in log-space:

```
N_s = C · A^α     =>     log(N_s) = log(C) + α · log(A)
```

where `N_s` is headcount and `A` is AUM (USD billion). Key findings:

- **Systematic Quant funds**: `α ≈ 0.4–0.7` (sub-proportional scaling; technology substitutes labour)
- **Pod-Shop platforms**: `α ≈ 1.0–1.5` (near-linear scaling; headcount proportional to PM teams)
- Separation is significant: Mann–Whitney U-test p = 0.005, permutation test p = 0.003
- OOS cluster hit rate: 79% on 26 held-out funds

---

## Quickstart

### 1. Install dependencies

```bash
pip install pandas numpy scipy scikit-learn matplotlib jupyter nbconvert
```

### 2. Run the notebooks (in order)

```bash
cd notebooks
jupyter nbconvert --to notebook --execute --inplace notebook1_data_exploration.ipynb
jupyter nbconvert --to notebook --execute --inplace notebook2_model_fitting.ipynb
jupyter nbconvert --to notebook --execute --inplace notebook3_clustering_oos_figures.ipynb
```

Or launch interactively:
```bash
jupyter lab
```

### 3. Compile the paper

```bash
cd paper
pdflatex pareto_scaling_hedgefunds
bibtex   pareto_scaling_hedgefunds
pdflatex pareto_scaling_hedgefunds
pdflatex pareto_scaling_hedgefunds
```

Output: `paper/pareto_scaling_hedgefunds.pdf`

> **Note:** Requires a standard TeX Live installation (`texlive-latex-recommended`, `texlive-fonts-recommended`, `texlive-bibtex-extra`).

---

## Notebook Descriptions

### `notebook1_data_exploration.ipynb`
- Loads and summarises both in-sample and OOS datasets
- Produces log₁₀ histograms of AUM and headcount
- Temporal coverage heatmap (which fund × year cells are observed)
- Capital efficiency (AUM per employee) bar chart by fund

### `notebook2_model_fitting.ipynb`
- OLS power-law fit with HC1 heteroskedasticity-robust standard errors
- Non-parametric percentile bootstrap CIs (B = 5,000, with measurement-noise injection)
- AIC-based model selection: power law vs log-normal vs linear
- Likelihood-ratio test (chi-squared, df = 1) of AUM significance
- Leave-one-out cross-validation MSE
- Produces `data/insample_fit_results.csv`

### `notebook3_clustering_oos_figures.ipynb`
- K-means clustering in (α̂, ln Ĉ, ln efficiency) feature space
- Multi-criterion number-of-clusters evaluation (silhouette, Calinski–Harabász, Davies–Bouldin)
- Cluster partition plot with confidence ellipses and silhouette bar chart
- Expanding-window temporal evolution of cluster membership
- Phase-space trajectories in (α̂(t), ln Ĉ(t))
- OOS validation: predicted vs observed headcount, cluster hit rate, MAE benchmark comparison
- Survivorship-bias correction: paper sample vs full-universe cluster fractions

---

## Key Statistical Methods

| Method | Purpose | Implementation |
|--------|---------|---------------|
| OLS (log-space) | Estimate α̂, Ĉ per fund | `ols_powerlaw()` in NB2 |
| HC1 robust SE | Heteroskedasticity-robust uncertainty | Eicker–Huber–White sandwich |
| Bootstrap CI | Propagate measurement noise | B=5,000 resamples + noise injection |
| AIC | Model selection (power law vs null) | ΔAIC = AIC_PL − AIC_lognormal |
| LRT | Significance of AUM predictor | chi²(1), p < 0.05 |
| LOO-CV | Out-of-sample fit quality | Leave-one-out MSE in log-space |
| K-means (K=3) | Organisational archetype clustering | scikit-learn, silhouette + CH criteria |
| Mann–Whitney U | Two-sample test, quant vs pod | scipy.stats.mannwhitneyu |

---

## Citation

If you use this code or data, please cite:

```
Jena, R. (2025). Workforce Scaling with Assets Under Management in Hedge Funds:
Evidence for Two Organisational Regimes. Risk Management, CFM LLP.
```

---

## License

All original code in this repository is released under the MIT License.
Data derived from public SEC ADV filings and industry reports; original source attribution is noted in the paper and data files.
