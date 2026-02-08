---
domain: algebraic-geometry
type: notation-preset
description: "Standard notation for algebraic geometry: varieties, schemes, sheaves, and cohomology"
---

# Algebraic Geometry Notation Preset

## Symbol Conventions

| Concept | Notation | LaTeX | Notes |
|---------|----------|-------|-------|
| Variety | X, Y, Z | `X`, `Y`, `Z` | Capital Latin letters for varieties/schemes |
| Morphism | \varphi: X \to Y | `\varphi: X \to Y` | Greek letters for morphisms |
| Sheaf (calligraphic) | \mathcal{F}, \mathcal{G} | `\mathcal{F}`, `\mathcal{G}` | Calligraphic letters for sheaves |
| Structure sheaf | \mathcal{O}_X | `\mathcal{O}_X` | Structure sheaf of scheme X |
| Ideal sheaf | \mathcal{I}_Z | `\mathcal{I}_Z` | Ideal sheaf of closed subscheme Z |
| Sheaf cohomology | H^i(X, \mathcal{F}) | `H^i(X, \mathcal{F})` | i-th cohomology of sheaf \mathcal{F} on X |
| Spectrum | \operatorname{Spec}(R) | `\operatorname{Spec}(R)` | Prime spectrum of ring R |
| Projective spectrum | \operatorname{Proj}(S) | `\operatorname{Proj}(S)` | Proj construction for graded ring S |
| Projective space | \mathbb{P}^n | `\mathbb{P}^n` | Projective n-space |
| Affine space | \mathbb{A}^n | `\mathbb{A}^n` | Affine n-space |
| Divisor (Weil) | D = \sum n_i D_i | `D = \sum n_i D_i` | Formal sum of codimension-1 subvarieties |
| Line bundle | \mathcal{L} | `\mathcal{L}` | Calligraphic L for line bundles |
| Tensor product of sheaves | \mathcal{F} \otimes \mathcal{G} | `\mathcal{F} \otimes \mathcal{G}` | Tensor product over \mathcal{O}_X |
| Global sections | \Gamma(X, \mathcal{F}) or H^0(X, \mathcal{F}) | `\Gamma(X, \mathcal{F})` | Space of global sections |
| Chern class | c_i(\mathcal{E}) | `c_i(\mathcal{E})` | i-th Chern class of vector bundle |

## LaTeX Packages

**Base (always included):**
- `amsmath` -- extended math environments
- `amssymb` -- blackboard bold and additional symbols
- `amsthm` -- theorem, lemma, proof environments

**Domain-specific:**
- `mathrsfs` -- script letters (\mathscr) for sheaf Hom, etc.
- `tikz-cd` -- commutative diagrams for exact sequences and functorial constructions

## Presentation Style

- **Theorem numbering:** Numbered within section. Single shared counter for all theorem-like environments.
- **Proof structure:** "Proof." in italics, end with \qed. Commutative diagrams (via tikz-cd) appear frequently in proofs of functoriality and exact sequences.
- **Equation numbering:** Number exact sequences and key isomorphisms. Short exact sequences are displayed and often numbered.
- **Definition style:** State the categorical/scheme-theoretic definition first, then the classical interpretation. Example: "A *closed subscheme* of X is a morphism i: Z \hookrightarrow X that is a closed immersion."
- **Bibliography style:** Alphanumeric labels. Standard references: Hartshorne, Vakil's *The Rising Sea*, Grothendieck's EGA/SGA, Liu.
