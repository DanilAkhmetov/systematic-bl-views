# Research Design

## Project Title

Systematic View Generation for Black–Litterman Portfolios

## Research Question

How does the method used to systematically generate Black–Litterman views affect out-of-sample forecast qualit & portfolio performance ?

---

## Main View-Generation Methods

1. Historical expected-return views
2. Momentum-based views
3. Factor / predictive-regression views

---

## Stage 2 Extensions

### Transaction Costs

Transaction costs are not included in Stage 1. The initial evaluation focuses on gross portfolio performance. Transaction-cost-adjusted performance will be maybe considerd in Stage 2.

### Views

Stage 2 will test increasingly sophisticated ways to generate views:

**Simple LSTM views**  
Raw historical industry returns → LSTM → forecast future returns → convert forecasts into $Q$.

**EMD-LSTM**  
Returns → EMD → IMFs → LSTM → forecast future returns.  
Purpose: test whether separating different time-scale components improves prediction.

**HHT-LSTM**  
Returns → EMD → IMFs → Hilbert analysis → amplitude, phase, instantaneous frequency → LSTM → forecasts → BL views.

### Covariance

As a robustness test, the project may later compare the current covariance specification with a data-driven shrinkage estimator such as **Ledoit–Wolf**.

### View Uncertainty Matrix $\Omega$

In a separate experiment, model-specific values of $\Omega_t$ may be estimated using historical forecast-error variance.

---

## Core Principle

All view-generation methods should be compared within the same Black–Litterman framework, keeping the rest of the portfolio construction process as constant as possible.

---

## Asset Universe

The main investment universe consists of the **Kenneth French 12 Industry Portfolios**, using value-weighted monthly returns, that is "vw".

The 12 portfolios are:

- NoDur — Consumer Nondurables
- Durbl — Consumer Durables
- Manuf — Manufacturing
- Enrgy — Energy
- Chems — Chemicals
- BusEq — Business Equipment / Technology
- Telcm — Telecommunications
- Utils — Utilities
- Shops — Wholesale and Retail
- Hlth — Healthcare
- Money — Finance
- Other — Remaining Industries

### Rationale

This choice is intended to:

- preserve broad representation of the equity market;
- avoid selection bias;
- provide a manageable number of assets for covariance estimation, forecasting, Black–Litterman allocation, and later LSTM modelling.

More granular Kenneth French industry universes, such as the 30- or 49-industry portfolios, may later be considered as robustness tests.

A stock-level universe, such as constituents of the S&P 500, may also be considered in a later stage, especially taking into account the factor model SMB aspect covered in the notebook for the first stage.

---

## Estimation Window

The classical models use a **60-month rolling estimation window**.

At each rebalance date, the most recent 60 monthly observations are used for covariance estimation and the main rolling portfolio inputs.

Some view-generation methods may additionally use expanding historical information where explicitly specified.

In the performance & forecast evaluation section, we consider sensitivity of the results achieved to the estimation windows size and rebalancing frequency. 


---

## Sample Period

The main out-of-sample backtest begins around **1960** and extends through the most recent available data.

Therefore, data from roughly **1955 onward** are used, with the first five years serving only as the initial estimation history.

### Rationale

It allows the analysis to span a wide range of market environments, including:

- the period before the Great Inflation;
- the inflationary 1970s;
- the Volcker disinflation period;
- the 1987 crash;
- the dot-com boom and bust;
- the Global Financial Crisis;
- the low-interest-rate environment of the 2010s;
- the COVID shock;
- the recent inflation and monetary-tightening cycle.

This long sample exposes the strategies to a wide range of market regimes and provides scope for a more explicit regime analysis in later stages.

---

## Benchmarks

The BL strategies are evaluated against a common set of benchmark portfolios over the same out-of-sample period.

### Main Benchmarks

- Market-cap-weighted industry portfolio
- Global Minimum Variance (GMV) portfolio
- Risk Parity / Equal Risk Contribution portfolio

---

## Portfolio Constraints

The main portfolio optimization is subject to the following constraints:

$$
0 \leq w_i \leq 0.30
$$

for every industry $i$, and

$$
\sum_{i=1}^{12} w_i = 1.
$$

The same 30% upper bound is imposed on the constrained GMV and Risk Parity benchmarks, by doing so we improve comparability across strategies.

---

## Covariance Estimation

The main experiment uses a **constant-correlation shrinkage covariance estimator**.

The estimator combines the sample covariance matrix with the Elton–Gruber constant-correlation covariance matrix:

$$
\hat{\Sigma}_t = (1-\lambda)S_t + \lambda F_{CC,t}
$$

where:

- $S_t$ is the sample covariance matrix;
- $F_{CC,t}$ is the constant-correlation covariance target;
- $\lambda = 0.5$ is the shrinkage intensity.

The covariance matrix is estimated using the most recent 60 months of monthly industry returns.

### Rationale

The sample covariance matrix can be noisy, especially with a limited rolling estimation window.

Shrinking toward a constant-correlation structure improves stability while preserving more cross-sectional structure than imposing a purely diagonal covariance matrix.

### Possible Extension

The fixed shrinkage intensity $\lambda = 0.5$ is a simplifying assumption.

As a robustness test, the project may later compare this specification with a data-driven shrinkage estimator such as **Ledoit–Wolf**.

---

## Black–Litterman Parameters

### Risk Aversion Parameter $\delta$

The main experiment uses a fixed market risk-aversion parameter:

$$
\delta = 2.5
$$

The parameter is used to derive the market-implied equilibrium return vector (based on mean-variance utility function):

$$
\Pi_t = \delta \Sigma_t w_t^{mkt}
$$

Surely, there is no imperical evidence that we should preserve this risk aversion constant over the long period.

So, a natural extension is to estimate (rolling estimate of risk aversion) the risk-aversion parameter dynamically:

$$
\delta_t =
\frac{\widehat{E}_t[R_m-R_f]}
{\widehat{\sigma}_{m,t}^2}
$$

---

### Tau $\tau$ (Views Certainty)

The main specification uses:

$$
\tau = \frac{1}{60} \approx 0.0167
$$

where 60 corresponds to the number of monthly observations in the rolling estimation window.

The parameter scales the uncertainty of the market-implied equilibrium return vector:

$$
\mathrm{Var}(\Pi_t) = \tau \Sigma_t
$$

A fixed value of $\tau$ is used across all view-generation methods to keep the Black–Litterman prior specification consistent.

The choice $\tau = 1/T$ follows a common practice, where $T$ is the estimation-window length.

However, there is, of course, no evidence it cannot vary across functions. IN fact, it would be very logical to assume it does as we implement different view generation methods. 

---

### Linkage Matrix $P$ and View Vector $Q$

In the Black–Litterman model, the matrix $P$ is the **linkage matrix**. It specifies which assets each view refers to and how those assets enter the view.

The vector $Q$ contains the numerical values of the views.

For the main experiment, the model uses **absolute views** on all 12 industry portfolios. Therefore,

$$
P = I_{12}
$$

where $I_{12}$ is the 12-dimensional identity matrix.

At each portfolio decision date $t$, the view vector is:

$$
Q_t =
\begin{bmatrix}
q_{1,t} \\
q_{2,t} \\
\vdots \\
q_{12,t}
\end{bmatrix}
$$

where $q_{i,t}$ is the forecast 12-month excess return for industry $i$.

---

### View Uncertainty Matrix $\Omega$

The primary experiment uses a common covariance-based specification of view uncertainty across all view-generation methods:

$$
\Omega_t = \tau \operatorname{diag}\left(P\Sigma_tP^\top\right)
$$

Because the main experiment uses absolute views with $P = I_{12}$, this simplifies to:

$$
\Omega_t = \tau \operatorname{diag}(\Sigma_t)
$$

This keeps the confidence specification consistent, allowing the main comparison to focus on differences in the generated view vector $Q_t$.

In Stage 2, model-specific uncertainty may be estimated from historical forecast-error variance.

---

## Portfolio Construction & Project Pipeline

The Black–Litterman model is used to generate both posterior expected returns and an adjusted covariance matrix, rather than final portfolio weights directly.

For each view-generation method, the process is:

```text
View vector Q_t and uncertainty matrix Ω_t
        ↓
Black–Litterman posterior expected returns μ_t^BL
and posterior predictive covariance Σ_t^BL
        ↓
Mean–variance utility optimization
        ↓
Portfolio weights w_t^BL
```
### Whole Pipeline

```text
Market and factor data
        ↓
Rolling information set
        ↓
Estimate common parameters
(Σ_t, market weights, δ, τ, P, Ω_t)
        ↓
Generate views Q_t
(Historical / Momentum / Factor)
        ↓
Black–Litterman update
        ↓
Posterior expected returns and covariance
        ↓
Portfolio optimization
        ↓
Portfolio weights
        ↓
Apply weights to next-period returns
        ↓
Walk-forward backtest
        ↓
Evaluate forecast quality and portfolio performance
        ↓
Compare with benchmarks and robustness tests