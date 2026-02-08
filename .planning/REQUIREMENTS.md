# Requirements: get-shit-done-math

**Defined:** 2026-02-08
**Core Value:** Help mathematicians move from problem statement to rigorous LaTeX writeup by coordinating specialized agents for each phase of the research workflow (search, prove, compute, write).

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Problem Intake & State

- [ ] **INTK-01**: User can submit a mathematical problem with structured context (domain, known results, notation, desired output type)
- [ ] **INTK-02**: System captures problem statement in LaTeX format and parses it for downstream agents
- [ ] **INTK-03**: User can resume a research session with full state restored (proof progress, literature found, approaches tried)
- [ ] **INTK-04**: System maintains an approach journal tracking attempted strategies, outcomes, and failure reasons
- [ ] **INTK-05**: User can configure per-project notation profiles (additive vs multiplicative groups, epsilon-delta vs sequences, preferred LaTeX packages)
- [ ] **INTK-06**: Notation profiles are loaded into every agent context automatically

### Literature

- [ ] **LIT-01**: System searches arXiv API for papers relevant to the user's problem
- [ ] **LIT-02**: System searches Semantic Scholar API for papers relevant to the user's problem
- [ ] **LIT-03**: Search results include title, authors, year, abstract, arXiv ID/DOI, and relevance explanation
- [ ] **LIT-04**: Every citation is verified against the API (no hallucinated references)
- [ ] **LIT-05**: System synthesizes connections between found papers and the user's problem ("Theorem A from [Smith] + Lemma 3 from [Jones] gives you...")
- [ ] **LIT-06**: Literature results are stored in session state for reuse across proof and writing stages

### Proof Collaboration

- [ ] **PRUF-01**: System suggests proof strategies (direct, contradiction, induction, contrapositive) based on problem structure
- [ ] **PRUF-02**: System identifies applicable known theorems from literature search results
- [ ] **PRUF-03**: System fills routine computational/algebraic gaps in proofs
- [ ] **PRUF-04**: System checks logical flow of proof steps and flags questionable reasoning
- [ ] **PRUF-05**: Every proof step is marked with confidence tier (VERIFIED/SUGGESTED/SPECULATIVE)
- [ ] **PRUF-06**: Proofs are presented as structured reasoning chains (numbered steps with justifications)
- [ ] **PRUF-07**: Complex proofs can be decomposed into lemmas and farmed to parallel agents
- [ ] **PRUF-08**: User can paste partial proof in LaTeX and receive gap analysis with suggested completions

### Computation

- [ ] **COMP-01**: System executes Python/SymPy computations via local Bash execution
- [ ] **COMP-02**: SageMath is supported as optional computation backend (detected at runtime)
- [ ] **COMP-03**: System auto-generates test cases for conjectures based on problem domain
- [ ] **COMP-04**: Counterexamples are reported with explanation when found
- [ ] **COMP-05**: All computations produce saved, runnable Python artifacts with version info

### Output

- [ ] **OUTP-01**: System generates compilable LaTeX documents with proper theorem/lemma/proof environments
- [ ] **OUTP-02**: LaTeX output includes bibliography with verified references in BibTeX format
- [ ] **OUTP-03**: LaTeX uses standard packages (amsmath, amssymb, amsthm) and user's notation profile
- [ ] **OUTP-04**: Intermediate output in terminal uses clean LaTeX source notation

### Orchestration

- [ ] **ORCH-01**: Full pipeline connects search → prove → compute → write as coordinated agent flow
- [ ] **ORCH-02**: Pipeline supports backward loops (proof agent can request additional literature or computation)
- [ ] **ORCH-03**: System installs as Claude Code plugin via npx
- [ ] **ORCH-04**: Commands are available as /slash commands in Claude Code

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Enhanced Proof

- **PRUF-09**: Computation-proof bridge (computation reveals pattern → suggest proof technique)
- **PRUF-10**: Formal verification integration (Lean 4) as opt-in module

### Enhanced Literature

- **LIT-07**: Reference management integration (BibTeX/Zotero export)
- **LIT-08**: zbMATH Open API as additional search source

### Enhanced Computation

- **COMP-06**: Custom CAS backend support (Mathematica, Maple)
- **COMP-07**: Wolfram Language integration for users with Mathematica licenses

### Collaboration

- **COLLAB-01**: Export full research state for sharing with collaborators
- **COLLAB-02**: Generate self-contained handoff documents for research partners

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Formal proof verification (Lean/Coq/Isabelle) | Massive complexity, unsolved translation problem, <1% of target users. Consider as v2 opt-in module. |
| Real-time collaboration / multiplayer | Math research is deeply individual cognitive work. Export-friendly output covers collaboration needs. |
| Automated paper submission | Removes human judgment at the most consequential moment. Produce submission-ready LaTeX instead. |
| Teaching/tutoring mode | Fundamentally different product with different user base. Stay focused on research. |
| Visual/graphical proof construction | Requires full GUI, contradicts CLI-first model. Generate TikZ for visual output in LaTeX. |
| General-purpose AI features | Scope creep. Claude Code already does general tasks. |
| Plagiarism/novelty checking | Cannot reliably determine novelty. Thorough literature search is the alternative. |
| Wolfram Alpha wrapper | Paid API, black box results. Local SymPy/SageMath is free, inspectable, integrated. |
| Standalone CLI (non-Claude Code) | Claude Code first. Standalone later if demand exists. |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| INTK-01 | — | Pending |
| INTK-02 | — | Pending |
| INTK-03 | — | Pending |
| INTK-04 | — | Pending |
| INTK-05 | — | Pending |
| INTK-06 | — | Pending |
| LIT-01 | — | Pending |
| LIT-02 | — | Pending |
| LIT-03 | — | Pending |
| LIT-04 | — | Pending |
| LIT-05 | — | Pending |
| LIT-06 | — | Pending |
| PRUF-01 | — | Pending |
| PRUF-02 | — | Pending |
| PRUF-03 | — | Pending |
| PRUF-04 | — | Pending |
| PRUF-05 | — | Pending |
| PRUF-06 | — | Pending |
| PRUF-07 | — | Pending |
| PRUF-08 | — | Pending |
| COMP-01 | — | Pending |
| COMP-02 | — | Pending |
| COMP-03 | — | Pending |
| COMP-04 | — | Pending |
| COMP-05 | — | Pending |
| OUTP-01 | — | Pending |
| OUTP-02 | — | Pending |
| OUTP-03 | — | Pending |
| OUTP-04 | — | Pending |
| ORCH-01 | — | Pending |
| ORCH-02 | — | Pending |
| ORCH-03 | — | Pending |
| ORCH-04 | — | Pending |

**Coverage:**
- v1 requirements: 33 total
- Mapped to phases: 0
- Unmapped: 33

---
*Requirements defined: 2026-02-08*
*Last updated: 2026-02-08 after initial definition*
