---
domain: combinatorics
type: notation-preset
description: "Standard notation for enumerative and algebraic combinatorics: counting, generating functions, and graph theory"
---

# Combinatorics Notation Preset

## Symbol Conventions

| Concept | Notation | LaTeX | Notes |
|---------|----------|-------|-------|
| Binomial coefficient | \binom{n}{k} | `\binom{n}{k}` | "n choose k" |
| Falling factorial | (n)_k or n^{\underline{k}} | `(n)_k` or `n^{\underline{k}}` | Pochhammer or underline notation |
| Rising factorial | n^{\overline{k}} | `n^{\overline{k}}` | Overline notation for rising factorial |
| Stirling (2nd kind) | S(n,k) or \left\{  {n \atop k} \right\} | `S(n,k)` or `\left\{ {n \atop k} \right\}` | Stirling numbers of the second kind |
| Stirling (1st kind) | s(n,k) or \left[ {n \atop k} \right] | `s(n,k)` or `\left[ {n \atop k} \right]` | Stirling numbers of the first kind (signed) |
| Generating function | F(x) = \sum_{n \geq 0} a_n x^n | `F(x) = \sum_{n \geq 0} a_n x^n` | Ordinary generating function |
| Exponential GF | \hat{F}(x) = \sum_{n \geq 0} a_n \frac{x^n}{n!} | `\hat{F}(x) = \sum_{n \geq 0} a_n \frac{x^n}{n!}` | Exponential generating function |
| Graph | G = (V, E) | `G = (V, E)` | Graph with vertex and edge sets |
| Chromatic polynomial | \chi(G, k) | `\chi(G, k)` | Number of proper k-colorings |
| Partition | \lambda \vdash n | `\lambda \vdash n` | Partition \lambda of integer n |
| Young diagram | \lambda = (\lambda_1, \lambda_2, \ldots) | `\lambda = (\lambda_1, \lambda_2, \ldots)` | Weakly decreasing sequence |
| Multiset coefficient | \left(\!\binom{n}{k}\!\right) | `\left(\!\binom{n}{k}\!\right)` | Multiset coefficient (stars and bars) |
| Degree of vertex | \deg(v) | `\deg(v)` | Degree of vertex v in a graph |
| Adjacency | u \sim v | `u \sim v` | Vertices u and v are adjacent |

## LaTeX Packages

**Base (always included):**
- `amsmath` -- extended math environments
- `amssymb` -- blackboard bold and additional symbols
- `amsthm` -- theorem, lemma, proof environments

**Domain-specific:**
- `tikz` -- graph diagrams, Young tableaux visualizations
- `ytableau` -- Young tableaux and Young diagrams (optional)

## Presentation Style

- **Theorem numbering:** Numbered within section. Propositions and Corollaries numbered with the same counter as Theorems for linear readability.
- **Proof structure:** "Proof." in italics, end with \qed. Bijective proofs describe the bijection explicitly. Generating function proofs display the algebraic manipulations step by step.
- **Equation numbering:** Number key identities (e.g., Vandermonde identity, hockey stick identity). Use \tag for named identities.
- **Definition style:** Constructive definitions preferred. Example: "A *partition* of n is a weakly decreasing sequence \lambda_1 \geq \lambda_2 \geq \cdots \geq \lambda_k > 0 with \sum \lambda_i = n."
- **Bibliography style:** Alphanumeric labels. Standard references: Stanley's *Enumerative Combinatorics* (vol. 1 & 2), van Lint-Wilson, Aigner.
