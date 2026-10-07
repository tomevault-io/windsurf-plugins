---
trigger: always_on
description: >-
---


# Conventions, notation and glossary

*Author: [Artur Sepp](https://github.com/ArturSepp)*

This page defines, once, the conventions that every page of the
[OptimalPortfolios](https://github.com/ArturSepp/OptimalPortfolios) documentation relies on.
Articles link here instead of restating them, and each methodology article summarises the ones
it uses in a convention card at the start of its inputs section. The conventions of the qis
analytics and of FactorLasso estimation are linked, not repeated.

Software citation: [CITATION.cff](https://github.com/ArturSepp/OptimalPortfolios/blob/main/CITATION.cff).

## Notation

The symbols below keep one meaning on every page. A page declares any further symbol in its own
notation table and does not reuse a reserved one. Pages revised from 27 September 2026 onwards
use this notation; some earlier pages still write the benchmark as $w_b$ or $b$ and the
annualisation factor as $a$, and they are brought in line when they are revised.

| Symbol | Meaning |
|---|---|
| $N$, $M$ | Number of assets and of factors |
| $t$, $t_k$ | A date; the $k$-th rebalancing (decision) date |
| $r$ | Returns; each page states whether they are simple or log returns |
| $w$ | Target weights, as fractions of net asset value |
| $w_0$ | Pre-trade weights: the previous targets drifted to the decision date |
| $w^{\mathrm{bm}}$ | Benchmark weights |
| $d = w - w^{\mathrm{bm}}$ | Active weights |
| $\mu$ | Expected returns |
| $\alpha$ | Alphas or signal scores used as expected active returns |
| $\Sigma$, $\sigma_i$ | Asset covariance matrix; asset volatility $\sigma_i = \sqrt{\Sigma_{ii}}$ |
| $\beta$, $\Sigma_F$, $D$ | Factor loadings ($N \times M$), factor covariance and residual covariance |
| $\sigma(w)$ | Portfolio volatility $\sqrt{w^{\top} \Sigma w}$ |
| $\mathrm{RC}_i$ | Risk contribution $w_i (\Sigma w)_i / \sigma(w)$ of asset $i$ |
| $b$ | Risk budgets, non-negative and summing to one |
| $\mathrm{TE}(w)$ | Ex-ante tracking error $\sqrt{d^{\top} \Sigma d}$ |
| $s$, $\lambda$ | EWMA span and the decay it implies |
| $\mathrm{af}$ | Annualisation factor, set upright as one symbol and named after the `af` argument of qis |
| $\gamma$ | Risk aversion |

The transpose is written $x^{\top}$. Covariance matrices are $N \times N$ and indexed by asset
labels; loadings are indexed by asset and then by factor, as in
[FactorLasso's conventions](https://factorlasso.readthedocs.io/en/latest/conventions.html).

## Returns, grids and estimation dates

Estimation samples prices on a return grid and produces estimates on a rebalancing grid. The
defaults are:

| Setting | Default | Used by |
|---|---|---|
| `returns_freq` | `'W-WED'` (weekly, Wednesday close) | `EwmaCovarEstimator`, and the dispatcher's expected returns and CARA mixture |
| `factor_returns_freq` | `'W-WED'` | `FactorCovarEstimator` |
| `rebalancing_freq` | `'QE'` (quarter ends) | Both estimators, which key their estimates by these dates |
| `span`, `factor_covar_span` | `52` observations | EWMA covariance and EWMA means |

The rebalancing frequency only selects which dates carry an estimate; it does not change the
return frequency. Assets observed at different cadences are handled as in
[mixed-frequency data](mixed_frequency_data.md).

**Return basis.** Estimation uses log returns: `compute_returns_from_prices` defaults to
`is_log_returns=True`, subtracts an EWMA mean and drops the first row, and the factor model and
the dispatcher's expected returns also use log returns. Weight drift and the qis backtest use
simple price ratios, and qis's own `to_returns` defaults to simple returns. Each page states the
basis of every return it shows.

**Estimation date.** An estimate dated $t$ uses information available at $t$: the EWMA estimate
keyed by $t$ includes the sampled return ending at $t$, and factor-model inputs are sliced
through $t$ inclusive. The weights decided at $t$ are executed afterwards, as described under
[timing](#decision-execution-and-drift). Nothing in a rolling path looks ahead.

## EWMA spans

A span $s$ sets the decay

$$
\lambda = 1 - \frac{2}{s + 1},
$$

and the half-life $h$ with $\lambda^{h} = 1/2$ is $h = \ln(1/2) / \ln(\lambda)$. The default span
of 52 weekly observations gives $\lambda = 51/53 \approx 0.962$ and a half-life of about 18
weeks. This is the convention of
[qis](https://quantinveststrats.readthedocs.io/en/stable/ewm_estimators.html) and FactorLasso.

> **Pitfall.** A span is neither a half-life nor a hard look-back window. `span=52` weights
> the most recent 18 weeks as heavily as all earlier observations together. Some example scripts
> and the ROSAA replication folder still call the span a half-life; they are wrong on this point.

## Units

**Covariance.** The estimators return annualised covariance. `EwmaCovarEstimator` multiplies the
per-observation estimate by the annualisation factor $\mathrm{af}$ inferred from the sampled
index: 52 for weekly, 12 for monthly and 4 for quarterly returns, with a fallback of 252 and a
warning when the frequency cannot be inferred. `estimate_current_ewma_covar` with
`apply_an_factor=False` keeps per-observation units. `FactorCovarEstimator` annualises the
factor covariance and the residual variances; loadings are not scaled. The optimisers never

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArturSepp/OptimalPortfolios](https://github.com/ArturSepp/OptimalPortfolios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
