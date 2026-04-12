# multicenter-randomization

> **Selecting a Randomization Method for a Multi-Center Clinical Trial with Stochastic Recruitment Considerations**

Quarto-based simulation reports and source code accompanying the BMC Medical Research Methodology (2024) paper by Sverdlov, Ryeznik, Anisimov, Kuznetsova, Knight, Carter, Drescher, and Zhao. The repository contains the full Monte Carlo simulation study comparing **16 randomization methods** for a 1:1 multi-center randomized controlled trial (RCT) under a **Poisson-gamma stochastic recruitment model**.

[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-green)](LICENSE)

---

## Reference

> Sverdlov O†, Ryeznik Y†, Anisimov V, Kuznetsova OM, Knight R, Carter K, Drescher S, Zhao W.  
> *Selecting a randomization method for a multi-center clinical trial with stochastic recruitment considerations.*  
> **BMC Medical Research Methodology**, 24:52, 2024.  
> [doi:10.1186/s12874-023-02131-z](https://doi.org/10.1186/s12874-023-02131-z) (Open Access)  
> †Equal contribution.

---

## Background

Designing a multi-center RCT requires simultaneous decisions about sample size, number and location of clinical sites, patient recruitment strategy, and the randomization method. Patient enrollment in multi-center trials is inherently **stochastic**: centers are activated over time, have heterogeneous recruitment rates, and under competitive recruitment the number of patients per center is a random variable — not predetermined.

This repository addresses the question: **which randomization method performs best for a multi-center 1:1 RCT when patient recruitment is stochastic?**

The answer depends on a three-way trade-off:

- **Balance at the trial level** — overall 1:1 split across all enrolled patients
- **Balance at the region level** — 1:1 split within each geographic region
- **Balance at the center level** — 1:1 split within each site
- **Allocation randomness** — unpredictability of treatment assignments to guard against selection bias

No single stratification approach can simultaneously optimize all three levels of balance, and this repository provides the simulation evidence to reason about the trade-offs.

---

## Stochastic Recruitment Model

Patient enrollment is governed by a **Poisson-gamma (hierarchical) model**:

- Each center *i* recruits patients according to a Poisson process with rate λᵢ
- Rates λᵢ are drawn independently from a Gamma(α, β) distribution, capturing inter-site heterogeneity
- Centers are activated at uniformly distributed random times within the first *AP* months of the trial
- The overall enrollment time T(n, N) is the random time at which the *n*th patient is enrolled

Under this model, the number of patients per center follows a Beta-Binomial distribution at the end of recruitment — capturing both the between-center variability in rates and the within-center Poisson variability.

---

## The 16 Randomization Methods

All 16 methods target a **1:1 allocation ratio** and enforce a **Maximum Tolerated Imbalance (MTI)** constraint of *b* = 2 (unless otherwise stated). They are organized across four stratification strategies:

### Unstratified (I–IV)
A single global randomization sequence for all *n* = 500 patients, regardless of center or region. Controls imbalance at the trial level only.

| # | Method |
|---|--------|
| I | Unstratified PBD (*b* = 2) |
| II | Unstratified BUD (*b* = 2) |
| III | Unstratified EUD (*b* = 2) |
| IV | Unstratified BSD (*b* = 2) |

### Region-Stratified (V–VIII)
Separate randomization sequences within each geographic region (*G* = 5). Controls imbalance at the region and trial levels.

| # | Method |
|---|--------|
| V | Region-stratified PBD (*b* = 2) |
| VI | Region-stratified BUD (*b* = 2) |
| VII | Region-stratified EUD (*b* = 2) |
| VIII | Region-stratified BSD (*b* = 2) |

### Center-Stratified (IX–XII)
Separate randomization sequences within each center (*N* = 80). Controls imbalance at the center level directly.

| # | Method |
|---|--------|
| IX | Center-stratified PBD (*b* = 2) |
| X | Center-stratified BUD (*b* = 2) |
| XI | Center-stratified EUD (*b* = 2) |
| XII | Center-stratified BSD (*b* = 2) |

### Dynamic Balancing Randomization — DBR (XIII–XV)
Covariate-adaptive method that enforces imbalance bounds simultaneously at the center (*b₁*), region (*b₂*), and trial (*b₃*) levels using a three-step deterministic override hierarchy.

| # | Method |
|---|--------|
| XIII | DBR (*b₁* = 2, *b₂* = 2, *b₃* = 2) |
| XIV | DBR (*b₁* = 2, *b₂* = 4, *b₃* = 4) |
| XV | DBR (*b₁* = 2, *b₂* = 4, *b₃* = 8) |

### Benchmark (XVI)
| # | Method |
|---|--------|
| XVI | Complete Randomization Design (CRD) — no imbalance control |

**Randomization procedures used within strata:**

| Abbreviation | Full Name | Mechanism |
|---|---|---|
| **PBD** | Permuted Block Design | Fixed blocks; perfectly balanced within each block |
| **BSD** | Big Stick Design | Coin-flip until imbalance hits *b*; then forces balance |
| **EUD** | Ehrenfest Urn Design | Urn with 2*b* balls; probabilistic rebalancing at each step |
| **BUD** | Block Urn Design | Urn that resets when a matched pair is removed; reduces predictability vs. PBD |

---

## Simulation Scenarios

The simulation is run under four scenarios varying the Poisson-gamma hyperparameters and trial geometry:

| Scenario | Description | α | β | *N* centers | MTI *b* |
|----------|-------------|---|---|-------------|---------|
| **1** | Base case — near-deterministic rates | 120 | 5800 | 80 | 2 |
| **2** | Realistic rate variability | 1.2 | 58 | 80 | 2 |
| **3** | More centers (accelerated recruitment) | 1.2 | 58 | 160 | 2 |
| **4** | Larger MTI threshold | 1.2 | 58 | 80 | 4 |

All scenarios assume *n* = 500 patients, *G* = 5 geographic regions, a 12-month target recruitment period with centers activated uniformly in the first 4 months, and **10,000 Monte Carlo replications**.

---

## Operating Characteristics Evaluated

### Operational Efficiency
- Distribution of **time to complete recruitment** (enroll the 500th patient)
- Average **number of centers** recruiting exactly *j* patients (*j* = 0, 1, 2, …)

### Balance / Statistical Efficiency
Imbalance is quantified through the **loss** framework — the number of patients' worth of information lost due to randomization-induced treatment imbalance relative to a perfectly balanced design:

| Loss | Definition | Interpretation |
|------|-----------|---------------|
| *L₁* (trial) | [D(n)]² / n | Imbalance penalty under a model with no covariates |
| *L₂* (region) | Σ_g [D̃_g(n)]² / ñ_g | Imbalance penalty when region is a covariate |
| *L₃* (center) | Σ_i [D_i(n)]² / n_i | Imbalance penalty when center is a covariate |

Additional balance metrics:
- **Relative efficiency** *RE* = 1 − *L/n* (compared to the idealized balanced design)
- **SD|D(n)|** — standard deviation of absolute overall imbalance
- **P_skewed** — expected proportion of centers with allocation more extreme than 2:1 or 1:2
- **Pr(|Imbalance| ≥ d)** at trial, region, and center levels for *d* = 0, 1, 2, …

### Randomness / Selection Bias Protection
- **PCG_c** — expected proportion of correct guesses under the *convergence strategy* (site-level observer who tracks running allocation within their center)
- **PCG_d** — expected proportion of correct guesses under the *deterministic strategy* (observer guesses only when next allocation is forced)
- **PD** — expected proportion of deterministic assignments in the sequence

---

## Key Findings

- **MTI methods** (BSD, EUD, BUD) achieve a substantially better balance–randomness trade-off than the conventional PBD at every level of stratification
- **Unstratified** and **region-stratified** randomization control imbalance well at their target level but fail at the other two levels (particularly at the center level)
- **Center-stratified** randomization provides the best center-level balance but still accumulates meaningful imbalance at the region and trial levels
- **DBR** is the only method that controls imbalance at all three levels simultaneously while maintaining allocation randomness; it is the recommended strategy for trials with competitive recruitment
- Adding more centers accelerates recruitment but increases the proportion of centers with very few (or zero) patients, which tends to increase center-level imbalances for center-stratified and DBR procedures
- Increasing the MTI threshold (*b* = 4 vs. *b* = 2) reduces the proportion of deterministic allocations and improves randomness at the cost of slightly wider imbalance distributions

---

## Repository Structure

```
multicenter-randomization/
├── qmd/                                    # Quarto source files for each scenario
│   ├── scenario01-simulation.qmd           # Scenario 1: base case
│   ├── scenario02-simulation.qmd           # Scenario 2: realistic recruitment variability
│   ├── scenario03-simulation.qmd           # Scenario 3: more centers
│   └── scenario04-simulation.qmd           # Scenario 4: larger MTI threshold
├── 20230731-scenario01-simulation-report.html   # Rendered HTML report, Scenario 1
├── 20230731-scenario02-simulation-report.html   # Rendered HTML report, Scenario 2
├── 20230731-scenario03-simulation-report.html   # Rendered HTML report, Scenario 3
├── 20230731-scenario04-simulation-report.html   # Rendered HTML report, Scenario 4
├── LICENSE                                 # GPL-3.0
└── README.md
```

The `qmd/` files are self-contained [Quarto](https://quarto.org/) documents that embed the simulation code (R and/or Julia) alongside prose, tables, and figures. The rendered HTML reports (dated 2023-07-31) are the final outputs submitted with the paper.

---

## Prerequisites

### Quarto
Install [Quarto CLI](https://quarto.org/docs/get-started/) to render the `.qmd` source files.

### R / Julia
The simulation code within the Quarto files uses R and/or Julia. Required packages are specified at the top of each `.qmd` file.

### Rendering a Report

```bash
quarto render qmd/scenario01-simulation.qmd
```

This will reproduce the HTML simulation report for Scenario 1.

---

## Language

The reports are rendered as **100% HTML** (pre-built outputs). The simulation source embedded in the `.qmd` files uses R (and optionally Julia via `JuliaCall`). The stochastic recruitment model and all 16 randomization procedures are implemented from scratch within the Quarto documents.

---

## Authors

| Name | Affiliation |
|------|-------------|
| [Oleksandr Sverdlov†](https://orcid.org/0000-0002-1626-2588) | Novartis Pharmaceuticals Corporation, East Hanover, NJ, USA |
| [Yevgen Ryeznik†](https://github.com/yevgenryeznik) | Department of Pharmacy, Uppsala University, Uppsala, Sweden |
| Volodymyr Anisimov | Quanticate, Hertfordshire, UK |
| Olga M. Kuznetsova | Merck & Co., Rahway, NJ, USA |
| Ruth Knight | University of Warwick, Coventry, UK |
| Kerstine Carter | Boehringer Ingelheim, Ridgefield, CT, USA |
| Sonja Drescher | Boehringer Ingelheim, Biberach, Germany |
| Wenle Zhao | Medical University of South Carolina, Charleston, SC, USA |

†Equal contribution.

---

## License

Released under the [GNU General Public License v3.0](LICENSE).

---

## Citation

```bibtex
@article{sverdlov2024multicenter,
  author  = {Sverdlov, Oleksandr and Ryeznik, Yevgen and Anisimov, Volodymyr and
             Kuznetsova, Olga M. and Knight, Ruth and Carter, Kerstine and
             Drescher, Sonja and Zhao, Wenle},
  title   = {Selecting a randomization method for a multi-center clinical trial
             with stochastic recruitment considerations},
  journal = {BMC Medical Research Methodology},
  volume  = {24},
  pages   = {52},
  year    = {2024},
  doi     = {10.1186/s12874-023-02131-z}
}
```
