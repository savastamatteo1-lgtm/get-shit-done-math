---
domain: analysis
type: notation-preset
description: "Standard notation for real and functional analysis: limits, norms, measure theory, and function spaces"
---

# Analysis Notation Preset

## Symbol Conventions

| Concept | Notation | LaTeX | Notes |
|---------|----------|-------|-------|
| Epsilon-delta | \forall \epsilon > 0, \exists \delta > 0 | `\forall \epsilon > 0, \exists \delta > 0` | Standard limit definition format |
| Norm | \|f\|, \|f\|_p | `\|f\|`, `\|f\|_p` | Double bars for norms; subscript for specific norm |
| L^p space | L^p(\Omega) | `L^p(\Omega)` | Lebesgue spaces with domain |
| Measure | d\mu, d\lambda | `d\mu`, `d\lambda` | Differential notation for measures; \lambda for Lebesgue |
| Supremum / Infimum | \sup, \inf | `\sup`, `\inf` | Operator names |
| Limit superior / inferior | \limsup, \liminf | `\limsup`, `\liminf` | Operator names for limit superior and inferior |
| Smooth functions | C^k(\Omega), C^\infty | `C^k(\Omega)`, `C^\infty` | k-times differentiable function spaces |
| Sobolev space | W^{k,p}(\Omega) | `W^{k,p}(\Omega)` | Sobolev spaces with differentiability and integrability |
| Inner product | \langle f, g \rangle | `\langle f, g \rangle` | Angle brackets for inner products |
| Weak convergence | f_n \rightharpoonup f | `f_n \rightharpoonup f` | Half-arrow for weak convergence |
| Strong convergence | f_n \to f | `f_n \to f` | Standard arrow for norm convergence |
| Almost everywhere | a.e. | `\text{a.e.}` | Abbreviation for almost everywhere |
| Essential supremum | \operatorname{ess\,sup} | `\operatorname{ess\,sup}` | For L^\infty norm definition |
| Integral | \int_\Omega f \, d\mu | `\int_\Omega f \, d\mu` | Thin space before differential |
| Partial derivative | \partial f / \partial x | `\partial f / \partial x` | Curly d for partial derivatives |

## LaTeX Packages

**Base (always included):**
- `amsmath` -- extended math environments
- `amssymb` -- blackboard bold and additional symbols
- `amsthm` -- theorem, lemma, proof environments

**Domain-specific:**
- `mathtools` -- \coloneqq, paired delimiters, improved matrices
- `esint` -- additional integral signs (optional, for contour integrals)

## Presentation Style

- **Theorem numbering:** Numbered within section. Separate counters for Theorems and Definitions; Lemmas, Propositions, Corollaries share the Theorem counter.
- **Proof structure:** "Proof." in italics, end with \qed. Estimates and inequalities are displayed equations. Chain inequalities vertically with aligned environments.
- **Equation numbering:** Number all displayed equations in proofs involving estimates. Use \label/\ref for back-references.
- **Definition style:** Definitions state the space, then the property. Example: "Let f \in L^p(\Omega). We say f is *essentially bounded* if..."
- **Bibliography style:** Numeric labels [1], [2], etc. Standard references: Rudin's *Real and Complex Analysis*, Folland, Brezis, Evans.
