# Roadmap: get-shit-done-math

## Overview

This roadmap delivers an AI-powered mathematical research assistant as a Claude Code plugin. The build starts with the foundational document architecture (problem intake, notation registry, confidence tier system) that all downstream agents depend on, then adds session persistence, followed by each specialized agent in dependency order: literature search (most independent), proof collaboration (consumes literature), computation (supports proof), and finally the orchestrator that wires them together and LaTeX output as the presentation layer. Every phase delivers a coherent, testable capability.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Foundation & Plugin Setup** - Document architecture, problem intake, notation registry, and plugin installation (completed 2026-02-08)
- [x] **Phase 2: Session State & Research Journal** - Persistent research sessions and approach tracking across time (completed 2026-02-08)
- [ ] **Phase 3: Literature Search Agent** - arXiv and Semantic Scholar search with verified references and synthesis
- [ ] **Phase 4: Proof Collaboration Agent** - Proof strategy, structured reasoning, confidence tiers, and gap analysis
- [ ] **Phase 5: Computation Agent** - Sandboxed Python/SymPy/SageMath execution with reproducible artifacts
- [ ] **Phase 6: Math Orchestrator** - Full search-prove-compute-write pipeline with backward loops
- [ ] **Phase 7: LaTeX Output Agent** - Publication-quality LaTeX generation with bibliography and notation profiles

## Phase Details

### Phase 1: Foundation & Plugin Setup
**Goal**: User can install the plugin and submit a structured mathematical problem with notation preferences, establishing the document architecture that all agents will use
**Depends on**: Nothing (first phase)
**Requirements**: INTK-01, INTK-02, INTK-05, INTK-06, ORCH-03, ORCH-04
**Success Criteria** (what must be TRUE):
  1. User can install the plugin via npx and see math-specific slash commands in Claude Code
  2. User can submit a mathematical problem with domain, known results, notation preferences, and desired output type, and the system captures it in a structured PROBLEM.md
  3. Problem statement is stored in LaTeX format and parsed into structured data that downstream agents can consume
  4. User can define a notation profile (e.g., additive vs multiplicative groups, preferred LaTeX packages) and it is automatically loaded into every agent context
  5. Confidence tier system (VERIFIED/SUGGESTED/SPECULATIVE) is defined as a shared protocol available to all agents
**Plans**: 3 plans

Plans:
- [x] 01-01-PLAN.md -- Plugin scaffold, confidence tier protocol, and document templates
- [x] 01-02-PLAN.md -- Notation domain presets and disabled "coming soon" commands
- [x] 01-03-PLAN.md -- Active slash commands and problem intake agent

### Phase 2: Session State & Research Journal
**Goal**: User can leave a research session and return days later with full context restored, including a record of what approaches were tried and why they succeeded or failed
**Depends on**: Phase 1
**Requirements**: INTK-03, INTK-04
**Success Criteria** (what must be TRUE):
  1. User can close Claude Code, reopen it, and resume a research session with proof progress, literature found, and approaches tried fully restored
  2. System maintains an approach journal that records each attempted strategy, its outcome, and the reason it was abandoned or succeeded
  3. Approach journal prevents the user from unknowingly re-exploring dead ends by surfacing prior attempts on similar strategies
**Plans**: 3 plans

Plans:
- [x] 02-01-PLAN.md -- Document schema foundation (config.json, DASHBOARD.md, JOURNAL.md templates, journal protocol)
- [x] 02-02-PLAN.md -- Multi-problem commands (/math:init update, Phase 1 path resolution, /math:switch, /math:archive)
- [x] 02-03-PLAN.md -- Session restoration (/math:resume command, session-management agent)

### Phase 3: Literature Search Agent
**Goal**: User can search for relevant mathematical literature and receive verified, synthesized results that connect to their problem
**Depends on**: Phase 1 (PROBLEM.md format, notation registry)
**Requirements**: LIT-01, LIT-02, LIT-03, LIT-04, LIT-05, LIT-06
**Success Criteria** (what must be TRUE):
  1. User can trigger a literature search and receive results from both arXiv and Semantic Scholar with title, authors, year, abstract, identifiers, and relevance explanation
  2. Every citation in the results has been verified against the source API -- no hallucinated references exist in confirmed results
  3. System synthesizes connections between found papers and the user's problem (e.g., "Theorem A from [Smith] combined with Lemma 3 from [Jones] addresses your hypothesis")
  4. Literature results persist in session state (LITERATURE.md) and are available to proof and writing stages without re-searching
**Plans**: 3 plans

Plans:
- [x] 03-01-PLAN.md -- Literature protocol and LITERATURE.md template
- [ ] 03-02-PLAN.md -- Literature search agent and /math:search command
- [ ] 03-03-PLAN.md -- Integration updates (help, status, init)

### Phase 4: Proof Collaboration Agent
**Goal**: User can collaborate with the system on developing mathematical proofs, receiving structured reasoning with explicit confidence levels and gap identification
**Depends on**: Phase 1 (confidence tiers, notation), Phase 3 (LITERATURE.md for theorem lookup)
**Requirements**: PRUF-01, PRUF-02, PRUF-03, PRUF-04, PRUF-05, PRUF-06, PRUF-08
**Success Criteria** (what must be TRUE):
  1. User receives proof strategy suggestions (direct, contradiction, induction, contrapositive) that are informed by the problem structure and available literature
  2. Every proof step is presented in a structured chain (numbered steps with justifications) and marked with a confidence tier (VERIFIED/SUGGESTED/SPECULATIVE) so the user knows exactly what has been established vs. what is heuristic
  3. System identifies applicable known theorems from literature search results and cites them with precise statements
  4. System flags questionable reasoning and identifies logical gaps rather than silently skipping hard steps
  5. User can paste a partial proof in LaTeX and receive gap analysis with suggested completions for missing steps
**Plans**: TBD

Plans:
- [ ] 04-01: TBD
- [ ] 04-02: TBD
- [ ] 04-03: TBD

### Phase 5: Computation Agent
**Goal**: User can run symbolic and numerical computations to test conjectures and verify proof claims, with all results saved as reproducible artifacts
**Depends on**: Phase 1 (notation, artifact formats)
**Requirements**: COMP-01, COMP-02, COMP-03, COMP-04, COMP-05
**Success Criteria** (what must be TRUE):
  1. User can request computations that execute locally via Python/SymPy, with SageMath used as an optional backend when detected
  2. System auto-generates test cases for conjectures based on the problem domain and reports results including counterexamples with explanations when found
  3. Every computation produces a saved, runnable Python script with version info, purpose metadata, and precision statements that the user can independently verify
  4. Computation execution is sandboxed with import whitelisting, timeouts, and resource limits -- no arbitrary system access from generated code
**Plans**: TBD

Plans:
- [ ] 05-01: TBD
- [ ] 05-02: TBD
- [ ] 05-03: TBD

### Phase 6: Math Orchestrator
**Goal**: User can run the full research pipeline (search, prove, compute, write) as a coordinated flow with automatic routing and backward loops when agents need additional information
**Depends on**: Phase 3 (literature agent), Phase 4 (proof agent), Phase 5 (computation agent)
**Requirements**: ORCH-01, ORCH-02, PRUF-07
**Success Criteria** (what must be TRUE):
  1. User can invoke a single command that coordinates the full search-prove-compute-write pipeline, with the orchestrator routing between agents based on artifact state
  2. When the proof agent needs additional literature or computation, the system automatically loops back to the appropriate agent and resumes proof development with new information
  3. Complex proofs can be decomposed into lemmas with each lemma farmed to a parallel agent, and results recombined into the parent proof
  4. Orchestrator reads artifact status flags to make routing decisions -- it never evaluates mathematical content directly
**Plans**: TBD

Plans:
- [ ] 06-01: TBD
- [ ] 06-02: TBD
- [ ] 06-03: TBD

### Phase 7: LaTeX Output Agent
**Goal**: User receives publication-quality LaTeX documents that faithfully represent the accumulated proof, computation, and literature from the research session
**Depends on**: Phase 1 (notation profiles), Phase 3 (LITERATURE.md), Phase 4 (PROOF.md), Phase 5 (COMPUTATION.md)
**Requirements**: OUTP-01, OUTP-02, OUTP-03, OUTP-04
**Success Criteria** (what must be TRUE):
  1. System generates compilable LaTeX documents with proper theorem/lemma/proof environments (using amsmath, amssymb, amsthm) that compile without errors
  2. LaTeX output includes a bibliography with verified references in BibTeX format, sourced from the literature agent's confirmed references
  3. Generated LaTeX respects the user's notation profile and uses their preferred packages and conventions throughout
  4. Intermediate output in the terminal uses clean LaTeX source notation so the user can read mathematical content before final document generation
**Plans**: TBD

Plans:
- [ ] 07-01: TBD
- [ ] 07-02: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation & Plugin Setup | 3/3 | Complete | 2026-02-08 |
| 2. Session State & Research Journal | 3/3 | Complete | 2026-02-08 |
| 3. Literature Search Agent | 1/3 | In progress | - |
| 4. Proof Collaboration Agent | 0/TBD | Not started | - |
| 5. Computation Agent | 0/TBD | Not started | - |
| 6. Math Orchestrator | 0/TBD | Not started | - |
| 7. LaTeX Output Agent | 0/TBD | Not started | - |
