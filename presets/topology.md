---
domain: topology
type: notation-preset
description: "Standard notation for general and algebraic topology: spaces, maps, homotopy, and homology"
---

# Topology Notation Preset

## Symbol Conventions

| Concept | Notation | LaTeX | Notes |
|---------|----------|-------|-------|
| Topological space | (X, \tau) | `(X, \tau)` | Space with topology; often just X when \tau is understood |
| Open sets | U, V, W | `U`, `V`, `W` | Capital Latin letters for open sets |
| Continuous map | f: X \to Y | `f: X \to Y` | Arrow notation for continuous maps |
| Homeomorphism | X \cong Y | `X \cong Y` | Congruence symbol for homeomorphism |
| Homotopy equivalence | X \simeq Y | `X \simeq Y` | Simeq for homotopy equivalence |
| Homotopic maps | f \simeq g | `f \simeq g` | Same symbol for homotopic maps |
| Fundamental group | \pi_1(X, x_0) | `\pi_1(X, x_0)` | First homotopy group with basepoint |
| Higher homotopy group | \pi_n(X) | `\pi_n(X)` | n-th homotopy group |
| Homology group | H_n(X; R) | `H_n(X; R)` | n-th homology with coefficients in R |
| Cohomology group | H^n(X; R) | `H^n(X; R)` | n-th cohomology with coefficients in R |
| Closure | \overline{A} | `\overline{A}` | Bar notation for closure |
| Interior | \operatorname{int}(A) or A^\circ | `\operatorname{int}(A)` or `A^\circ` | Interior of a set |
| Boundary | \partial A | `\partial A` | Boundary of a set |
| Exact sequence | \cdots \to A \to B \to C \to \cdots | `\cdots \to A \to B \to C \to \cdots` | Long exact sequences in homology/homotopy |
| Disjoint union | X \sqcup Y | `X \sqcup Y` | Square cup for disjoint union |

## LaTeX Packages

**Base (always included):**
- `amsmath` -- extended math environments
- `amssymb` -- blackboard bold and additional symbols
- `amsthm` -- theorem, lemma, proof environments

**Domain-specific:**
- `tikz-cd` -- commutative diagrams for exact sequences and functorial diagrams
- `xy` -- alternative commutative diagram package (optional)

## Presentation Style

- **Theorem numbering:** Numbered within section. All theorem-like environments share a single counter for readability in diagram-heavy papers.
- **Proof structure:** "Proof." in italics, end with \qed. Commutative diagrams appear as displayed figures within proofs.
- **Equation numbering:** Number sparingly. Commutative diagrams and exact sequences are displayed but typically unnumbered unless referenced.
- **Definition style:** State the construction, then name the object. Example: "A map f: X \to Y is a *covering map* if every point y \in Y has..."
- **Bibliography style:** Alphanumeric labels (e.g., [Hat02] for Hatcher 2002). Standard references: Hatcher's *Algebraic Topology*, Munkres, May, Bredon.
