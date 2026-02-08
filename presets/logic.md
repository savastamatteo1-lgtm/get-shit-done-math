---
domain: logic
type: notation-preset
description: "Standard notation for mathematical logic: proof theory, model theory, set theory, and computability"
---

# Logic Notation Preset

## Symbol Conventions

| Concept | Notation | LaTeX | Notes |
|---------|----------|-------|-------|
| Syntactic entailment | \Gamma \vdash \varphi | `\Gamma \vdash \varphi` | Provability from assumptions \Gamma |
| Semantic entailment | \Gamma \models \varphi | `\Gamma \models \varphi` | Truth in all models of \Gamma |
| Universal quantifier | \forall x | `\forall x` | For all x |
| Existential quantifier | \exists x | `\exists x` | There exists x |
| Negation | \neg \varphi | `\neg \varphi` | Logical negation |
| Conjunction | \varphi \wedge \psi | `\varphi \wedge \psi` | Logical and |
| Disjunction | \varphi \vee \psi | `\varphi \vee \psi` | Logical or |
| Implication | \varphi \to \psi | `\varphi \to \psi` | Material conditional |
| Biconditional | \varphi \leftrightarrow \psi | `\varphi \leftrightarrow \psi` | If and only if |
| Ordinal | \alpha, \beta, \gamma | `\alpha`, `\beta`, `\gamma` | Greek letters for ordinals |
| Cardinal | \aleph_0, \aleph_1 | `\aleph_0`, `\aleph_1` | Aleph notation for infinite cardinals |
| Forcing | p \Vdash \varphi | `p \Vdash \varphi` | Condition p forces \varphi |
| Turing reducibility | A \leq_T B | `A \leq_T B` | A is Turing reducible to B |
| Sequent | \Gamma \vdash \Delta | `\Gamma \vdash \Delta` | Sequent with multisets of formulas |
| Satisfaction | \mathcal{M} \models \varphi | `\mathcal{M} \models \varphi` | Structure \mathcal{M} satisfies formula \varphi |

## LaTeX Packages

**Base (always included):**
- `amsmath` -- extended math environments
- `amssymb` -- blackboard bold and additional symbols
- `amsthm` -- theorem, lemma, proof environments

**Domain-specific:**
- `bussproofs` -- proof trees for natural deduction and sequent calculus (optional)
- `stmaryrd` -- additional logical symbols such as \llbracket, \rrbracket (optional)

## Presentation Style

- **Theorem numbering:** Numbered within section. Metatheorems (about the logic) distinguished from object-level theorems (within the logic).
- **Proof structure:** "Proof." in italics, end with \qed. Formal derivations are displayed as proof trees (using bussproofs) or sequent chains. Informal metamathematical arguments use standard prose.
- **Equation numbering:** Number axiom schemas and key equivalences. Formal systems list axioms with labels (e.g., (A1), (A2), ...).
- **Definition style:** Recursive/inductive definitions are standard. Example: "The set of *formulas* is defined inductively: (i) every atomic formula is a formula; (ii) if \varphi is a formula, then \neg \varphi is a formula; ..."
- **Bibliography style:** Alphanumeric labels. Standard references: Enderton, Marker's *Model Theory: An Introduction*, Kunen's *Set Theory*, Soare.
