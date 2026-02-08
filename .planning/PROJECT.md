# get-shit-done-math

## What This Is

An open-source Claude Code plugin that orchestrates specialized AI agents for mathematical research. Given a mathematical problem, it guides the user through literature search, proof collaboration, computation, and formal LaTeX output — using an orchestrator-agent-document architecture inspired by GSD.

## Core Value

Help mathematicians move from problem statement to rigorous LaTeX writeup by coordinating specialized agents for each phase of the research workflow (search, prove, compute, write).

## Requirements

### Validated

- ✓ Multi-agent orchestration with fresh context per agent — existing
- ✓ Document-driven state management (.planning/ markdown files) — existing
- ✓ Claude Code plugin distribution (npx install pattern) — existing
- ✓ Command system (/slash commands as entry points) — existing
- ✓ Utility tooling (gsd-tools.js for config, git, state) — existing
- ✓ Multi-runtime installer (Claude Code, OpenCode, Gemini) — existing

### Active

- [ ] Literature search agent (arXiv, Google Scholar, reference textbooks)
- [ ] Literature synthesis and review (connect results, identify relevant theorems)
- [ ] Proof collaboration agent (suggest approaches, fill gaps, check reasoning)
- [ ] Computation agent (Python/SageMath/Mathematica integration)
- [ ] LaTeX output generation (formal proofs, papers, research notes)
- [ ] Problem intake workflow (capture problem statement, domain, known results)
- [ ] Math-specific orchestrator workflows (search → prove → compute → write)
- [ ] Research state tracking (what's been found, what's proven, what's pending)

### Out of Scope

- Formal proof verification (Lean/Coq/Isabelle integration) — high complexity, consider for v2
- Standalone CLI independent of AI IDEs — Claude Code first, standalone later
- Automated paper submission — out of scope entirely
- Teaching/tutoring mode — this is a research tool, not an educational tool

## Context

- **Existing codebase:** GSD orchestrator-agent system with proven patterns for multi-step AI workflows
- **Pivot approach:** Inspired by GSD architecture but workflows redesigned from scratch for mathematical research
- **Target users:** Mathematicians (pure and applied) who want AI assistance in their research workflow
- **Computation ecosystem:** Python/SymPy/SageMath and Mathematica/Wolfram are the primary computation backends
- **Literature sources:** arXiv preprints, Google Scholar, and domain-specific textbooks
- **Output format:** LaTeX documents (proofs, papers, research notes)

## Constraints

- **Distribution:** Must install as Claude Code plugin via npx — same model as current GSD
- **Architecture:** Orchestrator-agent pattern with document-as-interface — proven pattern, keep the paradigm
- **Agent context:** Each agent gets fresh 200k context — design workflows around this constraint
- **Computation:** Python/SageMath run locally via Bash tool; Mathematica requires local install

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Redesign workflows from scratch for math | Math research has fundamentally different flow than software engineering | — Pending |
| Keep orchestrator-agent-document paradigm | Proven pattern for multi-step AI work, maps well to research phases | — Pending |
| Claude Code plugin first | Leverage existing distribution model, standalone CLI later if needed | — Pending |
| LaTeX as primary output format | Standard for mathematical writing, universally expected | — Pending |
| arXiv + Google Scholar for literature | Most accessible sources, good coverage of pure and applied math | — Pending |

---
*Last updated: 2026-02-08 after initialization*
