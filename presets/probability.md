---
domain: probability
type: notation-preset
description: "Standard notation for probability theory and stochastic processes: measures, expectations, and convergence"
---

# Probability Notation Preset

## Symbol Conventions

| Concept | Notation | LaTeX | Notes |
|---------|----------|-------|-------|
| Probability measure | \mathbb{P} | `\mathbb{P}` | Blackboard bold P for probability |
| Expectation | \mathbb{E}[X] | `\mathbb{E}[X]` | Blackboard bold E for expectation |
| Variance | \operatorname{Var}(X) | `\operatorname{Var}(X)` | Variance of random variable |
| Covariance | \operatorname{Cov}(X, Y) | `\operatorname{Cov}(X, Y)` | Covariance of two random variables |
| Probability space | (\Omega, \mathcal{F}, \mathbb{P}) | `(\Omega, \mathcal{F}, \mathbb{P})` | Sample space, sigma-algebra, measure |
| Sigma-algebra | \mathcal{F}, \mathcal{G} | `\mathcal{F}`, `\mathcal{G}` | Calligraphic letters for sigma-algebras |
| Filtration | (\mathcal{F}_t)_{t \geq 0} | `(\mathcal{F}_t)_{t \geq 0}` | Indexed family of sigma-algebras |
| Random variable | X, Y, Z | `X`, `Y`, `Z` | Capital Latin letters for random variables |
| Conditional expectation | \mathbb{E}[X \mid \mathcal{F}] | `\mathbb{E}[X \mid \mathcal{F}]` | Expectation conditioned on sigma-algebra |
| Convergence in distribution | X_n \xrightarrow{d} X | `X_n \xrightarrow{d} X` | Weak convergence |
| Convergence in probability | X_n \xrightarrow{P} X | `X_n \xrightarrow{P} X` | Convergence in probability |
| Almost sure convergence | X_n \xrightarrow{a.s.} X | `X_n \xrightarrow{a.s.} X` | Almost sure convergence |
| L^p convergence | X_n \xrightarrow{L^p} X | `X_n \xrightarrow{L^p} X` | Convergence in L^p norm |
| Distribution | X \sim \mu or X \sim F | `X \sim \mu` or `X \sim F` | Random variable distributed as \mu |
| Indicator function | \mathbf{1}_A or \mathbb{1}_A | `\mathbf{1}_A` or `\mathbb{1}_A` | Indicator of event A |

## LaTeX Packages

**Base (always included):**
- `amsmath` -- extended math environments
- `amssymb` -- blackboard bold and additional symbols
- `amsthm` -- theorem, lemma, proof environments

**Domain-specific:**
- `mathtools` -- \coloneqq, paired delimiters for conditional expectations
- `bbm` -- \mathbbm{1} for indicator functions (optional)

## Presentation Style

- **Theorem numbering:** Numbered within section. Separate numbering for Theorems and Definitions.
- **Proof structure:** "Proof." in italics, end with \qed. Probability proofs frequently use inequalities (Markov, Chebyshev, Jensen) as displayed equations. Conditioning arguments are displayed step by step.
- **Equation numbering:** Number key inequalities and moment calculations. Named results use \tag (e.g., \tag{Borel-Cantelli}).
- **Definition style:** State the measurability requirement, then the property. Example: "A random variable X is *integrable* if X is \mathcal{F}-measurable and \mathbb{E}[|X|] < \infty."
- **Bibliography style:** Numeric labels. Standard references: Billingsley, Durrett, Williams' *Probability with Martingales*, Kallenberg.
