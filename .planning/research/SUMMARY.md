# Project Research Summary

**Project:** get-shit-done-math
**Domain:** AI-powered mathematical research assistant (Claude Code plugin)
**Researched:** 2026-02-08
**Confidence:** MEDIUM (training data through early 2025; web verification unavailable; architectural patterns HIGH confidence)

## Executive Summary

Get-shit-done-math is a Claude Code plugin that orchestrates mathematical research workflows by connecting literature search, proof collaboration, symbolic computation, and LaTeX output generation. Unlike existing tools that excel in isolated capabilities (Wolfram Alpha for computation, arXiv for search, raw LLMs for reasoning), this tool provides the first integrated pipeline where each stage feeds the next: literature informs proof strategy, computation validates claims, and everything flows into publication-ready LaTeX.

The recommended approach extends GSD's proven orchestrator-agent-document architecture into the mathematical domain. Five specialized agents (literature, proof, computation, LaTeX, plus a thin orchestrator) communicate exclusively through structured markdown files. The technology stack is intentionally minimal: no new npm dependencies, Python/SymPy for local computation via Bash, arXiv/Semantic Scholar APIs for literature via WebFetch, and Claude's native reasoning enhanced by computation verification loops. This "thin glue, fat agents" philosophy keeps the barrier to entry low while delivering genuine research value through workflow orchestration.

The critical risk is LLM hallucination of false mathematical statements. Mathematicians demand rigor, and any tool that presents unverified claims as fact will be immediately dismissed. Mitigation requires a tiered confidence system (VERIFIED/SUGGESTED/SPECULATIVE) baked into every agent's output format from day one, computational spot-checking wherever possible, and API-verified references for all citations. The tool must position itself as a "proof collaborator" that aids human mathematicians, never claiming to replace human verification. Success hinges on honest uncertainty communication and preserving mathematical precision across agent boundaries.

## Key Findings

### Recommended Stack

This project is fundamentally a **Claude Code plugin**, not a standalone application. The "stack" is primarily agent prompts plus Python prerequisites that users install separately. No server, no database, no bundler -- just orchestrated workflows that delegate computation and API calls to external tools via GSD's Bash/WebFetch capabilities.

**Core technologies:**
- **Node.js (>= 16.7.0)**: Plugin installer and gsd-tools.js utilities -- inherited from GSD, zero new npm dependencies needed
- **Python (>= 3.10) + SymPy (>= 1.13)**: Symbolic computation via Bash tool -- SymPy is pip-installable, lightweight (~30MB), covers 80% of computational needs
- **arXiv API (Atom/RSS)**: Primary literature source -- free, no auth required, covers all math preprints via direct HTTP GET to `export.arxiv.org/api/query`
- **Semantic Scholar API**: Citation graphs and structured paper data -- free tier, 100 req/sec unauthenticated, better for related work discovery than arXiv keyword search
- **Claude (native)**: Reasoning engine -- we ARE inside Claude Code; the plugin's intelligence lives in prompt engineering, not external LLM APIs
- **SageMath (>= 10.3)**: Optional power-user CAS -- for algebraic number theory, elliptic curves, modular forms beyond SymPy's scope; significant install burden, support as optional backend

**Key insight:** LaTeX generation and mathematical reasoning are **prompt engineering problems**, not library problems. Claude writes LaTeX natively; adding template libraries reduces quality. SymPy's `latex()` function handles expression conversion. The innovation is workflow orchestration, not technology novelty.

**Version flag:** All Python library versions are from training data (mid-2025 cutoff) and need verification before finalizing requirements.

### Expected Features

**Must have (table stakes):**
- **Structured problem intake** -- capture domain, known results, notation conventions, desired output type (proof/computation/survey)
- **LaTeX output generation** -- compilable documents with amsmath/amsthm environments, not Markdown fragments
- **Computation integration (SymPy)** -- verify conjectures numerically, compute examples, handle symbolic algebra via Bash-invoked Python
- **Literature search (arXiv + Semantic Scholar)** -- find relevant papers/theorems with verified references; without this it's just a chatbot
- **Proof collaboration** -- suggest strategies, identify applicable theorems, fill routine gaps, check logical structure (but never claim "verified" -- LLMs hallucinate)
- **Structured reasoning chains** -- step-by-step logic with clear justifications, not stream-of-consciousness paragraphs
- **Session state and continuity** -- mathematical research spans days/weeks; must resume where left off via `.planning/` state management

**Should have (competitive differentiators):**
- **Orchestrated research pipeline** -- connects search → prove → compute → write in a single flow; replaces fragmented alt-tab workflow between arXiv/Wolfram/LaTeX editor/ChatGPT
- **Multi-agent proof decomposition** -- complex proofs decompose into lemmas; farm each to specialized agents with fresh 200k context
- **Literature synthesis** -- not just finding papers but connecting them: "Theorem A from [Smith 2019] + Lemma 3 from [Jones 2021] gives you the key ingredient"
- **Approach journal** -- persistent record of what was tried, what failed, why; prevents repeating dead ends when returning days later
- **Conjecture testing via computation** -- auto-generate test cases, run computations, report "holds for 10k cases" or "counterexample at n=17"
- **Bidirectional LaTeX workflow** -- paste existing partial proof in LaTeX, tool identifies gaps and suggests completions
- **Computation-proof bridge** -- when computation reveals a pattern, suggest why it's true and what proof technique could establish it

**Defer (v2+):**
- **Formal verification (Lean/Coq)** -- HIGH complexity, niche audience (<1% of users), would consume 10x the effort of everything else
- **Real-time collaboration / multiplayer** -- massive infrastructure for a workflow that's naturally asynchronous
- **Teaching/tutoring mode** -- fundamentally different product; dilutes research focus
- **Automated paper submission** -- removes human judgment at the most consequential moment

**Anti-features (deliberately exclude):**
- Formal proof verification in v1 (defer to v2 opt-in module)
- Wolfram Alpha wrapper (local SymPy is free and inspectable)
- General-purpose AI assistant features (scope creep)

### Architecture Approach

The system extends GSD's orchestrator-agent-document pattern into mathematical research. Five specialized agents operate under a thin math orchestrator, communicating **exclusively through markdown files** in `.planning/math/`. The orchestrator never does math -- it reads YAML frontmatter status flags and routes accordingly.

**Major components:**
1. **math-orchestrator** -- thin workflow coordinator; maintains state machine in STATE.md; spawns agents; handles iteration loops; validates state transitions
2. **problem-intake (inline)** -- captures problem statement, domain, known constraints; writes PROBLEM.md with structured frontmatter
3. **literature-agent** -- searches arXiv API + Semantic Scholar API via WebFetch; writes LITERATURE.md with verified references and extracted theorems
4. **proof-agent** -- develops proof strategy, structures arguments, identifies gaps; reads PROBLEM.md + LITERATURE.md + COMPUTATION.md; writes PROOF.md with status flags
5. **computation-agent** -- executes Python/SageMath code via Bash; interprets results; writes COMPUTATION.md with reproducible scripts and output
6. **latex-agent** -- generates publication-quality LaTeX from all artifacts; reads everything, writes OUTPUT.tex + OUTPUT.pdf

**Key architectural principles:**
- **Agents never call other agents** -- only orchestrator spawns; prevents spaghetti dependencies
- **Communicate only through files** -- no in-memory state sharing; each agent gets fresh 200k context
- **Artifact-driven routing** -- orchestrator reads YAML frontmatter (`status: complete`, `computation_needed: true`), never parses mathematical content
- **Mathematical research is not linear** -- workflow supports forward flow AND backward loops (proof needs lemma → requests more literature → resumes proof)
- **Thin glue, fat agents** -- intelligence lives in agent prompts, not library code; no Python wrapper scripts to maintain

**Directory structure:** `.planning/math/` contains STATE.md (orchestrator state machine), PROBLEM.md, LITERATURE.md, PROOF.md, COMPUTATION.md, OUTPUT.tex. When user starts new problem, current artifacts archive to `sessions/session-NNN/` preserving history.

**State machine:** INTAKE → LITERATURE_SEARCH → PROOF_DEVELOPMENT ⇄ COMPUTATION (bidirectional) → LATEX_OUTPUT. Proof agent flags trigger transitions; orchestrator reads frontmatter, never evaluates mathematical correctness.

### Critical Pitfalls

1. **Confident hallucination of false mathematical statements** -- LLMs cite theorems that don't exist, apply results outside their hypotheses, invert inequality directions, skip hard steps with "it follows that" -- all with confident tone. **Prevention:** tiered confidence system (VERIFIED/SUGGESTED/SPECULATIVE) in every output; provenance markers for all claims; every theorem citation includes source and exact statement; computational spot-checking wherever possible. Address in Phase 1 (proof agent design) -- retrofitting provenance is extremely painful.

2. **The illusion of verification (circular confidence)** -- LLM "checking" its own proof almost always says yes; linguistic plausibility ≠ logical validity. **Prevention:** never claim "verified" for LLM-checked proofs; use "reviewed" or "no issues detected"; ground verification in computation (numerical checks, identity verification); use structural checks (variables introduced before use?) not semantic ones. Address in Phase 1 (proof agent) + Phase 2 (computation integration).

3. **Computation results without reproducibility** -- code that produced result is lost or not saved in runnable format; numerical results lack precision statements. **Prevention:** every computation saves persistent script to `.math/computations/` with metadata (timestamp, purpose, library versions); prefer symbolic over numerical; state method and limitations ("computed with scipy.integrate.quad, tolerance 1e-10"); version-pin environments. Address in Phase 2 (computation agent design) -- this is architecture, not a feature.

4. **Destroying mathematical nuance through oversimplification** -- "convergence" without specifying type (pointwise/uniform/in measure), "continuous" without topology, "differentiable" without specifying sense. **Prevention:** prompt engineering for precision; domain-specific checklists for common failures; structured output for mathematical objects with explicit attributes; require agent to state "which version" of overloaded terms. Address in Phase 1 (proof agent system prompt) -- impossible to retrofit.

5. **Literature search that confabulates references** -- LLMs fabricate plausible paper titles/authors/arXiv IDs that don't exist. **Prevention:** every reference MUST be verified against API (arXiv, Semantic Scholar); include DOI/arXiv ID and verify resolves correctly; separate "suggested references" (unverified) from "confirmed references" (API-verified); never mix categories. Address in Phase 1 (literature agent) -- API integration is not optional, it's the core value proposition.

6. **Notation inconsistency across agent boundaries** -- different agents use different notation for same objects (f'(x) vs df/dx, 0-indexed vs 1-indexed, R^n vs RR^n). **Prevention:** per-project notation registry in `.math/notation.md` loaded into every agent's context; notation translation layer between computation environments and mathematical writing; require agents to register new notation before using. Address in Phase 1 (project setup/problem intake) -- affects all downstream agents.

7. **Conflating LLM pattern matching with mathematical proof** -- users accepting proof agent output as actual proof rather than heuristic sketch. **Prevention:** frame as "proof collaborator," never "prover"; mark every proof step with justification type (AXIOM/CITED/COMPUTED/DEDUCED/SUGGESTED); make SUGGESTED steps visually prominent; include explicit "gap markers" where agent cannot complete step; track proof status (any SUGGESTED steps = "in progress"). Address in Phase 1 (proof agent UX) -- this is the most important design decision for the entire project.

8. **Unsafe code execution from LLM-generated computation** -- generated Python can execute system commands, read/write files, enter infinite loops. **Prevention:** sandbox execution (no network, no file system outside project, resource limits); whitelist safe imports (sympy, numpy, scipy only; block os, subprocess, socket); timeouts (60s), memory caps; log all executed code. Address in Phase 2 (computation agent) -- sandbox before any auto-execution.

9. **Losing research state across long investigations** -- mathematical research takes weeks/months; without persistent state, agents re-explore failed approaches, lose track of proven lemmas, introduce contradictions. **Prevention:** structured "research journal" tracking problem, attempts, outcomes, current proof state, abandoned approaches with reasons; agents read/update at every session; contradiction detection; journal must fit in 200k context. Address in Phase 1 (research state management) -- foundational architecture.

10. **LaTeX output that looks right but is mathematically wrong** -- formatting pass alters content (operator precedence errors, subscript/superscript confusion, `\subset` vs `\subseteq` changes). **Prevention:** strict separation of content from formatting; diff mathematical content between input and output; use LaTeX macros for mathematical objects; never let LaTeX agent "simplify" expressions; compilation check. Address in Phase 3 (LaTeX agent) but intermediate representation must be designed in Phase 1.

## Implications for Roadmap

Based on research findings, the roadmap should build the **document infrastructure and proof agent first** (where hallucination risk is highest), then add **computation and literature as supporting capabilities**, finally **LaTeX output as the presentation layer**. The orchestrator should be built incrementally alongside agents for integration testing, finalized last.

### Phase 1: Foundation & Problem Intake
**Rationale:** Everything downstream depends on structured problem definition and artifact formats. The confidence tier system and notation registry are architectural decisions that affect all agents -- must be designed before any mathematical work begins.

**Delivers:**
- PROBLEM.md schema with structured frontmatter (domain, type, constraints)
- STATE.md format and transition logic for orchestrator
- Notation registry (`.math/notation.md`) loaded into every agent context
- Research journal format for tracking approaches/outcomes
- Math-specific extensions to gsd-tools.js (state transitions, artifact validation)
- Confidence tier system design (VERIFIED/SUGGESTED/SPECULATIVE)

**Features from FEATURES.md:**
- Structured problem intake (must-have)
- Session state and continuity (must-have)
- Notation profiles (should-have)

**Avoids from PITFALLS.md:**
- Pitfall 4 (destroying nuance) -- notation registry prevents inconsistency
- Pitfall 7 (notation inconsistency) -- explicitly addressed
- Pitfall 10 (losing research state) -- research journal format defined

**Research flag:** STANDARD PATTERNS -- state management is proven in GSD; math-specific schema needs design but no external research required.

### Phase 2: Literature Search Agent
**Rationale:** Most independent agent; can be built and tested without proof or computation agents existing. Exercises external API integration patterns that will inform computation agent. Addressing the reference hallucination pitfall early builds trust foundation.

**Delivers:**
- Literature agent definition (markdown agent file)
- arXiv API integration via WebFetch (HTTP GET to `export.arxiv.org/api/query`)
- Semantic Scholar API integration via WebFetch (`api.semanticscholar.org/graph/v1`)
- LITERATURE.md artifact format with verified references
- Reference verification system (every citation must have DOI/arXiv ID that resolves)
- Separation of "confirmed" vs "suggested" references

**Features from FEATURES.md:**
- Literature search (arXiv + Semantic Scholar) -- must-have
- Structured reasoning chains (must-have, applies to literature summaries)

**Avoids from PITFALLS.md:**
- Pitfall 6 (confabulated references) -- API verification is core functionality
- Integration gotcha: arXiv is preprints only, use Semantic Scholar for broader coverage

**Research flag:** NEEDS RESEARCH -- API endpoints, rate limits, response formats need verification (training data from mid-2025). Use `/gsd:research-phase "Literature API Integration"` during phase planning to verify current API state.

### Phase 3: Proof Collaboration Agent
**Rationale:** Core value proposition; most complex agent; must read LITERATURE.md format (Phase 2 dependency). This is where hallucination risk is highest -- the confidence system and provenance tracking designed in Phase 1 get implemented here.

**Delivers:**
- Proof agent definition with rigorous prompt engineering for precision
- PROOF.md artifact format with status flags, justification types, gap markers
- Confidence tier implementation (VERIFIED/SUGGESTED/SPECULATIVE marking)
- Proof strategy suggestion (direct/contradiction/induction/contrapositive)
- Gap identification and computation request system
- Domain-specific precision checklists (analysis, algebra, topology)
- Structured proof output (claim → strategy → numbered steps with justifications → QED)

**Features from FEATURES.md:**
- Proof collaboration (must-have) -- THE feature
- Structured reasoning chains (must-have)
- Orchestrated research pipeline (should-have) -- first agent in pipeline

**Avoids from PITFALLS.md:**
- Pitfall 1 (confident hallucination) -- confidence tiers and provenance markers
- Pitfall 2 (circular confidence) -- never claim "verified," use structural checks
- Pitfall 4 (destroying nuance) -- precision prompting + checklists
- Pitfall 8 (conflating LLM with proof) -- framed as "collaborator," justification types make uncertainty visible

**Research flag:** STANDARD PATTERNS -- mathematical proof structure is well-documented; prompt engineering for precision is the challenge, not external research. Consider validation with sample proofs during testing.

### Phase 4: Computation Agent & Integration
**Rationale:** Enables verification of proof claims; proof agent can flag `computation_needed: true` once PROOF.md format exists (Phase 3 dependency). Sandbox design is safety-critical -- must be architected before any auto-execution.

**Delivers:**
- Computation agent definition
- COMPUTATION.md artifact format with reproducible scripts
- Sandbox execution environment (import whitelist, resource limits, no network/file access)
- Python/SymPy integration via Bash tool
- SageMath detection and optional backend support
- Computation artifact storage (`.math/computations/` with metadata)
- Version pinning and precision statements for results
- Computational verification loop (proof agent ← computation results → proof agent revises)

**Features from FEATURES.md:**
- Computation integration (must-have)
- Conjecture testing (should-have) -- auto-generate test cases, find counterexamples
- Computation-proof bridge (should-have) -- defer to v1.1; needs proof + computation maturity first

**Avoids from PITFALLS.md:**
- Pitfall 3 (computation without reproducibility) -- persistent artifacts with metadata
- Pitfall 9 (unsafe code execution) -- sandboxing is non-negotiable
- Integration gotcha: SymPy `simplify()` uses heuristics; verify with `equals()` or numerical evaluation

**Research flag:** NEEDS RESEARCH -- SymPy current version/capabilities, SageMath installation patterns, sandboxing approaches for Python execution. Use `/gsd:research-phase "Computation Sandbox"` to investigate secure execution patterns.

### Phase 5: Math Orchestrator Workflow
**Rationale:** Wires all agents together; depends on all agent definitions and artifact formats existing (Phases 1-4 complete). A skeleton orchestrator should exist from Phase 2 for integration testing, but full state machine with iteration loops and user interaction happens here.

**Delivers:**
- Math orchestrator agent definition
- State machine implementation (INTAKE → LITERATURE → PROOF ⇄ COMPUTATION → LATEX)
- Artifact-driven routing (read YAML frontmatter, spawn appropriate agents)
- Iteration loop management (cap at 3 loops per pair: proof-literature, proof-computation)
- User interaction points (when proof is stuck, present options)
- Session archival to `sessions/session-NNN/` when starting new problem
- Error handling and recovery strategies

**Features from FEATURES.md:**
- Orchestrated research pipeline (should-have) -- completes the integration
- Multi-agent proof decomposition (should-have) -- orchestrator handles lemma decomposition and agent spawning

**Avoids from PITFALLS.md:**
- Anti-pattern 2 (stateless agent spawning) -- orchestrator loads relevant artifacts via @-references
- Anti-pattern 3 (orchestrator does math) -- only reads status flags, never evaluates mathematical content
- Anti-pattern 4 (unbounded iteration loops) -- iteration caps with user fallback

**Research flag:** STANDARD PATTERNS -- orchestrator-agent routing is proven GSD pattern; math-specific state transitions are straightforward once artifact formats exist.

### Phase 6: LaTeX Output Agent
**Rationale:** Depends on all artifact formats (PROBLEM.md, LITERATURE.md, PROOF.md, COMPUTATION.md). Consumes accumulated state and produces final deliverable. The intermediate representation between proof agent and LaTeX agent (designed in Phase 3) determines what information is preserved.

**Delivers:**
- LaTeX agent definition
- OUTPUT.tex generation with proper document structure (amsmath, amsthm, bibliography)
- Theorem/lemma/proof environment mapping from PROOF.md structure
- SymPy `latex()` integration for expression conversion
- Bibliography generation from LITERATURE.md verified references
- Compilation check (run pdflatex, catch errors)
- Semantic preservation diff (compare mathematical content in PROOF.md vs OUTPUT.tex)

**Features from FEATURES.md:**
- LaTeX output generation (must-have)
- Bidirectional LaTeX workflow (should-have) -- defer input parsing to v1.1; output generation is v1 priority

**Avoids from PITFALLS.md:**
- Pitfall 5 (LaTeX semantic corruption) -- strict content/formatting separation, semantic diff checker
- Anti-pattern 5 (mixing LaTeX with proof development) -- proof agent writes informal markdown, LaTeX agent handles presentation

**Research flag:** STANDARD PATTERNS -- LaTeX generation is Claude's native strength; no external research needed.

### Phase 7: Approach Journal & Enhancements
**Rationale:** Add after core pipeline works and is validated with real mathematicians. These features enhance existing capabilities rather than adding new ones.

**Delivers:**
- Approach journal implementation (track attempts, outcomes, reasons for abandonment)
- Literature synthesis enhancement (connect papers: "Theorem A + Lemma B gives key ingredient")
- Bidirectional LaTeX input parsing (paste partial proof, identify gaps)
- Enhanced conjecture testing (beyond basic computation integration from Phase 4)

**Features from FEATURES.md:**
- Approach journal (should-have)
- Literature synthesis (should-have)
- Bidirectional LaTeX (should-have)

**Trigger conditions:** Add when users report:
- Losing track of what they've tried (→ approach journal)
- Wanting tool to connect dots between papers (→ literature synthesis)
- Regularly pasting partial LaTeX proofs (→ bidirectional input)

**Research flag:** STANDARD PATTERNS -- all build on existing agent capabilities.

### Phase Ordering Rationale

**Why Foundation First (Phase 1):**
- Artifact formats are architectural decisions that affect all agents
- Confidence system and notation registry cannot be retrofitted
- State management patterns must be established before any agents spawn

**Why Literature Before Proof (Phase 2 → 3):**
- Literature agent is most independent; can be built/tested in isolation
- Proof agent needs to read LITERATURE.md format
- API integration patterns learned in Phase 2 inform computation agent in Phase 4
- Early focus on reference verification builds trust foundation

**Why Proof Before Computation (Phase 3 → 4):**
- Proof agent defines what computations are needed (PROOF.md computation requests)
- Computation agent exists to support proof development, not standalone
- Verification loop requires PROOF.md format to exist first

**Why Orchestrator After All Agents (Phase 5):**
- Cannot wire agents that don't exist yet
- Skeleton orchestrator built alongside Phase 2-4 for integration testing
- Full state machine needs all artifact formats and agent behaviors defined

**Why LaTeX Last (Phase 6):**
- Depends on all upstream artifact formats
- Presentation layer; no other agents depend on it
- Can be delayed without blocking research workflow (users get proof text)

**Why Enhancements After Validation (Phase 7):**
- Add features based on real mathematician feedback, not speculation
- Core pipeline must work first to validate product-market fit
- Enhancements build on proven patterns

### Research Flags

**Needs deeper research during phase planning:**
- **Phase 2 (Literature):** API endpoints, rate limits, response formats -- training data from mid-2025, verify current state with `/gsd:research-phase`
- **Phase 4 (Computation):** Python sandboxing approaches, SymPy current version/capabilities, SageMath installation patterns -- security-critical, needs thorough research

**Standard patterns (skip research-phase):**
- **Phase 1 (Foundation):** State management proven in GSD; math-specific schema design, not research
- **Phase 3 (Proof):** Mathematical proof structure well-documented; prompt engineering challenge
- **Phase 5 (Orchestrator):** GSD orchestrator-agent routing is established pattern
- **Phase 6 (LaTeX):** Claude's native strength, no external research needed
- **Phase 7 (Enhancements):** Build on existing capabilities

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| **Stack** | MEDIUM | Core recommendations HIGH confidence (SymPy/arXiv/Semantic Scholar are stable ecosystems); specific version numbers MEDIUM (training data mid-2025, need verification); architecture approach HIGH (thin glue, fat agents matches proven GSD pattern) |
| **Features** | MEDIUM-HIGH | Table stakes and differentiators HIGH confidence (mathematical research workflow is well-understood; competitor analysis clear); MVP definition solid; v2 deferrals sensible; anti-features correctly identified |
| **Architecture** | HIGH | Orchestrator-agent-document pattern proven in GSD; mathematical workflow maps naturally to component boundaries; artifact-driven routing is sound; data flow well-defined; pitfall prevention mechanisms identified |
| **Pitfalls** | HIGH | Core failure modes well-documented in ML literature (hallucination, reference confabulation, oversimplification); computation safety patterns established; mathematical precision requirements clear; recovery strategies practical |

**Overall confidence:** MEDIUM-HIGH

- **HIGH confidence areas:** Architecture patterns, pitfall identification, feature categorization, anti-patterns
- **MEDIUM confidence areas:** Specific library versions, API endpoint details, integration specifics
- **Known limitations:** Web verification unavailable; all external API details from training data (mid-2025 cutoff)

### Gaps to Address

**Version verification needed (HIGH priority):**
- Python library versions: SymPy (stated >= 1.13), NumPy (>= 1.26), SciPy (>= 1.13), SageMath (>= 10.3)
- Verify with: `pip install <package> && python -c "import <package>; print(<package>.__version__)"`
- Impact: Requirements specification in README, installation instructions

**API endpoint verification (HIGH priority):**
- arXiv API: Confirm `export.arxiv.org/api/query` endpoint still current, rate limits, Atom XML response format
- Semantic Scholar API: Verify `api.semanticscholar.org/graph/v1` endpoint, rate limits (stated 100/sec unauthenticated), available fields
- Handle during Phase 2 planning with `/gsd:research-phase "Literature API Integration"`

**New competitor check (MEDIUM priority):**
- Tools like Julius AI, Minerva, DeepMind AlphaProof may have matured since mid-2025 cutoff
- Check during MVP validation; doesn't affect initial architecture

**Claude math capabilities (LOW priority):**
- Claude Opus 4.6 mathematical reasoning may have improved since training cutoff
- Test during Phase 3 (proof agent) with sample research-level problems
- Affects prompt engineering tuning, not architecture

**Integration validation (ongoing):**
- SymPy `simplify()` behavior, SageMath installation patterns, LaTeX compilation requirements
- Validate incrementally during implementation phases
- Known issues documented in PITFALLS.md integration gotchas section

### Validation Strategy

**Before Phase 1:**
- Verify Python library versions and update requirements
- Confirm arXiv + Semantic Scholar API endpoints (use `/gsd:research-phase` if needed)

**During Phase 3:**
- Test proof agent with sample research problems to calibrate Claude's actual mathematical reasoning capability
- Validate confidence tier system with mathematician users

**During Phase 4:**
- Thoroughly test sandboxing approach with adversarial computation code
- Verify SymPy and SageMath integration patterns

**After Phase 6 (pre-launch):**
- End-to-end workflow validation with real mathematicians
- Competitor feature comparison refresh
- Documentation completeness check

## Sources

### Primary (HIGH confidence)
- **GSD codebase analysis** (`.planning/codebase/`) -- orchestrator-agent-document architecture, state management patterns, command structure, agent spawning, document-driven state
- **ML research literature** (training data) -- LLM mathematical reasoning failures (GSM8K, MATH benchmark, MiniF2F), hallucination patterns, verification limitations
- **Mathematical computation ecosystem** (training data) -- SymPy/SageMath/Mathematica capabilities, symbolic vs numerical computation tradeoffs, CAS limitations
- **LaTeX typesetting best practices** (training data) -- theorem environments, mathematical notation, TeX StackExchange community knowledge

### Secondary (MEDIUM confidence)
- **arXiv API documentation** (training data, likely stable) -- endpoint structure, Atom XML format, rate limits; verify against current docs during Phase 2
- **Semantic Scholar API** (training data, widely used) -- endpoint structure, JSON response format, rate limits; verify during Phase 2
- **Python library documentation** (training data) -- SymPy, NumPy, SciPy, mpmath capabilities; version numbers need verification
- **Academic search patterns** (domain knowledge) -- how mathematicians search literature, what metadata matters, citation practices

### Tertiary (LOW confidence, needs validation)
- **SageMath current version** (stated >= 10.3 from training data) -- verify installation methods, feature availability, verify during Phase 4 if users report issues
- **zbMATH Open API availability** (training data) -- stated as "verify API availability" in STACK.md; optional, low priority
- **Google Scholar access patterns** (training data) -- scraping prohibited, use Semantic Scholar API instead; confirmed in PITFALLS.md
- **Specific library version numbers** (training data mid-2025) -- all flagged for verification in STACK.md "Version Verification Needed" section

---
*Research completed: 2026-02-08*
*Ready for roadmap: yes*
