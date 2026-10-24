# Systematic View Generation for Black–Litterman Portfolios

Welcome to my research project on systematic view generation within the Black–Litterman framework.

The main question is simple: **how much does the way we generate views actually matter for portfolio outcomes?**

To explore this, the first stage compares historical, momentum-based, and factor-based views within the same Black–Litterman setup and evaluates both forecast quality and portfolio performance.
The second stage will concern the same question under different methods. 

## Repository Structure

```text
.
├── README.md
├── research_design.md
├── notebooks/
├── src/
├── data/
└── results/
```

## Repository Guide

### `research_design.md`

Documents the research framework behind the project.

It contains the main research question, hypotheses, asset universe, Black–Litterman assumptions, portfolio constraints, benchmark definitions, evaluation criteria, and planned extensions for later stages of the project.

---

### `notebooks/00_setup_check.ipynb`

Initial project setup and environment checks.

Used to verify that the required libraries, local modules, and project paths are working correctly before running the main analysis.

---

### `notebooks/01_data.ipynb`

Data preparation notebook.

Responsible for loading, cleaning, aligning, and preparing the datasets used in the project, including:

- Kenneth French industry portfolio returns;
- industry market-cap information;
- Fama–French factors;
- the risk-free rate;
- processed inputs used by the portfolio backtests.

Processed datasets are exported to `data/processed/`.

---

### `notebooks/02_bl_baseline.ipynb`

Main Stage 1 research notebook.

This notebook develops and evaluates the classical Black–Litterman framework used in the project.

It includes:

- validation of the Black–Litterman implementation;
- construction of the market-implied prior;
- historical expected-return views;
- momentum-based views;
- factor-based views;
- walk-forward portfolio backtesting;
- forecast-quality diagnostics;
- robustness tests across estimation windows and rebalance frequencies;
- comparison with Market, Global Minimum Variance, and Risk Parity benchmarks;
- discussion of alternative specifications and empirical findings.

This notebook contains the main results of Stage 1.

---

### `src/data_toolkit.py`

Utility functions used for loading and preparing financial datasets.

The module separates data-handling logic from the research notebooks so that the analysis remains easier to read and reproduce.

---

### `src/portfolio_toolkit.py`

Core portfolio-analysis library used throughout the project.

It contains reusable functions for tasks such as:

- covariance estimation;
- portfolio optimization;
- Black–Litterman calculations;
- systematic view generation;
- walk-forward backtesting;
- Risk Parity / Equal Risk Contribution;
- portfolio performance and risk statistics.

---

### `data/raw/`

Original source datasets used in the project.

These files are kept separate from transformed datasets so that the data-preparation process remains reproducible.

---

### `data/processed/`

Datasets generated during the data-preparation stage and used directly by the research notebooks.

Examples include aligned industry returns, market weights, the risk-free rate, and forward excess-return targets.

---

### `data/results/`

Exported outputs from the empirical analysis.

The directory contains selected:

- performance tables;
- forecast-quality results;
- robustness-test results;
- cumulative-wealth figures;
- benchmark comparisons.

The full analysis and interpretation remain in the research notebooks.
