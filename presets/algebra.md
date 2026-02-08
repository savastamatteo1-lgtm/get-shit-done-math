---
domain: algebra
type: notation-preset
description: "Standard notation for abstract algebra: groups, rings, fields, modules, and homomorphisms"
---

# Algebra Notation Preset

## Symbol Conventions

| Concept | Notation | LaTeX | Notes |
|---------|----------|-------|-------|
| Group | G, H, K | `G`, `H`, `K` | Capital Latin letters |
| Abelian group operation | + | `+` | Additive notation for commutative groups |
| General group operation | \cdot or juxtaposition | `\cdot` or `ab` | Multiplicative notation is default for non-abelian |
| Identity element | e or 1 | `e` or `1` | Use e for groups, 1 for rings |
| Ideal (fraktur) | \mathfrak{a}, \mathfrak{p}, \mathfrak{m} | `\mathfrak{a}`, `\mathfrak{p}`, `\mathfrak{m}` | Fraktur for ideals; \mathfrak{p} for prime, \mathfrak{m} for maximal |
| Field | \mathbb{F}, \mathbb{K}, k | `\mathbb{F}`, `\mathbb{K}`, `k` | Blackboard bold or lowercase for ground field |
| Ring homomorphism | \varphi, \psi | `\varphi`, `\psi` | Greek letters for morphisms |
| Normal subgroup | N \trianglelefteq G | `N \trianglelefteq G` | Triangle notation for normal subgroups |
| Direct sum | A \oplus B | `A \oplus B` | Circle-plus for external direct sum |
| Direct product | A \times B | `A \times B` | Cross product for direct product |
| Quotient group | G / N | `G / N` | Forward slash for quotients |
| Kernel | \ker(\varphi) | `\ker(\varphi)` | Operator name for kernel |
| Image | \operatorname{im}(\varphi) | `\operatorname{im}(\varphi)` | Operator name for image |
| Isomorphism | \cong | `\cong` | Congruence symbol for isomorphism |
| Subgroup | H \leq G | `H \leq G` | Less-than-or-equal for subgroup relation |

## LaTeX Packages

**Base (always included):**
- `amsmath` -- extended math environments
- `amssymb` -- blackboard bold, fraktur, and additional symbols
- `amsthm` -- theorem, lemma, proof environments

**Domain-specific:**
- None required beyond base for standard algebra

## Presentation Style

- **Theorem numbering:** Numbered within section (Theorem 2.1, Lemma 2.2, etc.)
- **Proof structure:** Begin with "Proof." in italics, end with \qed (tombstone). State proof strategy in first sentence when non-obvious.
- **Equation numbering:** Number only referenced equations. Use \tag for named results (e.g., \tag{First Isomorphism Theorem}).
- **Definition style:** Definitions use bold for the term being defined. Example: "A group **G** is *abelian* if..."
- **Bibliography style:** alphanumeric labels (e.g., [Lan02] for Lang 2002). Standard references: Lang's *Algebra*, Hungerford, Dummit-Foote.
