---
domain: number-theory
type: notation-preset
description: "Standard notation for algebraic and analytic number theory: integers, primes, congruences, and L-functions"
---

# Number Theory Notation Preset

## Symbol Conventions

| Concept | Notation | LaTeX | Notes |
|---------|----------|-------|-------|
| Integers | \mathbb{Z} | `\mathbb{Z}` | Blackboard bold Z |
| Rationals | \mathbb{Q} | `\mathbb{Q}` | Blackboard bold Q |
| Finite field | \mathbb{F}_p, \mathbb{F}_q | `\mathbb{F}_p`, `\mathbb{F}_q` | Blackboard bold F with prime or prime power |
| Congruence | a \equiv b \pmod{n} | `a \equiv b \pmod{n}` | Parenthesized mod notation |
| Legendre symbol | \left(\frac{a}{p}\right) | `\left(\frac{a}{p}\right)` | Fraction notation for quadratic residue symbol |
| Dirichlet character | \chi | `\chi` | Greek chi for characters |
| Riemann zeta | \zeta(s) | `\zeta(s)` | Zeta function of complex variable s |
| Dirichlet L-function | L(s, \chi) | `L(s, \chi)` | L-function attached to character \chi |
| Euler totient | \varphi(n) | `\varphi(n)` | Euler's phi function |
| Mobius function | \mu(n) | `\mu(n)` | Mobius function for inversion |
| Ring of integers | \mathcal{O}_K | `\mathcal{O}_K` | Ring of integers of number field K |
| Ideal class group | \operatorname{Cl}(K) | `\operatorname{Cl}(K)` | Class group of number field K |
| Galois group | \operatorname{Gal}(L/K) | `\operatorname{Gal}(L/K)` | Galois group of extension L/K |
| Norm | N_{L/K}(\alpha) | `N_{L/K}(\alpha)` | Field norm of extension |
| Divisor sum | \sum_{d \mid n} | `\sum_{d \mid n}` | Sum over divisors of n |

## LaTeX Packages

**Base (always included):**
- `amsmath` -- extended math environments
- `amssymb` -- blackboard bold and additional symbols
- `amsthm` -- theorem, lemma, proof environments

**Domain-specific:**
- None required beyond base for standard number theory

## Presentation Style

- **Theorem numbering:** Numbered within section. Separate numbering for Conjectures (Conjecture A, B, ...) from Theorems.
- **Proof structure:** "Proof." in italics, end with \qed. Proofs frequently use congruence chains and divisibility arguments displayed with aligned environments.
- **Equation numbering:** Number key identities and functional equations. Named results use \tag (e.g., \tag{Euler product}).
- **Definition style:** State the arithmetic property, then name. Example: "An integer p > 1 is *prime* if its only positive divisors are 1 and p."
- **Bibliography style:** Alphanumeric labels. Standard references: Ireland-Rosen, Neukirch, Serre's *A Course in Arithmetic*, Hardy-Wright.
