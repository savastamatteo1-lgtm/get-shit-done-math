---
domain: differential-geometry
type: notation-preset
description: "Standard notation for differential and Riemannian geometry: manifolds, bundles, connections, and curvature"
---

# Differential Geometry Notation Preset

## Symbol Conventions

| Concept | Notation | LaTeX | Notes |
|---------|----------|-------|-------|
| Smooth manifold | M, N | `M`, `N` | Capital Latin letters for manifolds |
| Tangent bundle | TM | `TM` | Tangent bundle of manifold M |
| Cotangent bundle | T^*M | `T^*M` | Cotangent bundle (dual) |
| Tangent space at point | T_pM | `T_pM` | Tangent space at point p |
| Differential form | \omega, \eta, \alpha | `\omega`, `\eta`, `\alpha` | Greek letters for forms |
| k-form | \Omega^k(M) | `\Omega^k(M)` | Space of smooth k-forms on M |
| Exterior derivative | d\omega | `d\omega` | Exterior derivative of form |
| Covariant derivative | \nabla_X Y | `\nabla_X Y` | Covariant derivative of Y along X |
| Lie bracket | [X, Y] | `[X, Y]` | Lie bracket of vector fields |
| Lie derivative | \mathcal{L}_X | `\mathcal{L}_X` | Lie derivative along vector field X |
| Riemannian metric | g or \langle \cdot, \cdot \rangle | `g` or `\langle \cdot, \cdot \rangle` | Metric tensor |
| Riemann curvature | R(X,Y)Z or R^i{}_{jkl} | `R(X,Y)Z` or `R^i{}_{jkl}` | Curvature tensor (coordinate-free or index) |
| Ricci curvature | \operatorname{Ric} or R_{ij} | `\operatorname{Ric}` or `R_{ij}` | Ricci tensor |
| Christoffel symbols | \Gamma^k_{ij} | `\Gamma^k_{ij}` | Connection coefficients |
| Wedge product | \alpha \wedge \beta | `\alpha \wedge \beta` | Exterior product of forms |

## LaTeX Packages

**Base (always included):**
- `amsmath` -- extended math environments
- `amssymb` -- blackboard bold and additional symbols
- `amsthm` -- theorem, lemma, proof environments

**Domain-specific:**
- `tensor` -- index notation with proper placement (optional)
- `tikz` -- manifold and bundle diagrams (optional)

## Presentation Style

- **Theorem numbering:** Numbered within section. Separate counters for Theorems/Propositions and Definitions.
- **Proof structure:** "Proof." in italics, end with \qed. Computations involving curvature and connections use aligned environments for multi-line index calculations.
- **Equation numbering:** Number key geometric identities and formulas. Use \tag for named results (e.g., \tag{Gauss-Bonnet}, \tag{Bianchi identity}).
- **Definition style:** Intrinsic (coordinate-free) definition first, local coordinate expression second. Example: "A *connection* on a vector bundle E is a map \nabla: \Gamma(E) \to \Gamma(T^*M \otimes E) satisfying..."
- **Bibliography style:** Alphanumeric labels. Standard references: do Carmo, Lee's *Riemannian Manifolds* and *Introduction to Smooth Manifolds*, Spivak, Kobayashi-Nomizu.
