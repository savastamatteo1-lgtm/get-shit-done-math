# Feature Research

**Domain:** AI-powered mathematical research assistant (Claude Code plugin)
**Researched:** 2026-02-08
**Confidence:** MEDIUM (based on training data through early 2025; web verification unavailable)

## Feature Landscape

### Table Stakes (Users Expect These)

Features mathematicians assume exist. Missing these means the tool is not worth using over raw Claude Code.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| **Problem intake with mathematical context** | Mathematicians need to specify domain (algebra, topology, analysis...), known results, notation conventions, and what "done" means (proof, computation, survey). Without structured intake, the LLM guesses wrong context. | MEDIUM | Must capture: problem statement (ideally LaTeX), mathematical domain/subdomain, known related results, desired output type (proof, computation, conjecture exploration, survey). Parse LaTeX in problem statement. |
| **LaTeX output generation** | LaTeX is the universal language of mathematical writing. Any tool that outputs plain text or Markdown for final deliverables is immediately dismissed. Mathematicians live in LaTeX. | MEDIUM | Must produce compilable LaTeX, not just LaTeX fragments. Support common packages: amsmath, amssymb, amsthm, tikz-cd (commutative diagrams), algorithm2e. Theorem/lemma/proof environments are mandatory. |
| **Computation integration (Python/SymPy/SageMath)** | Mathematicians routinely need to verify conjectures numerically, compute examples, check edge cases. Wolfram Alpha and SageMath are the baseline expectation. Running computation locally via Bash is already possible in Claude Code -- the tool must orchestrate it well. | MEDIUM | Execute Python/SymPy/SageMath code via Bash tool. Parse and present results meaningfully. Handle: symbolic computation, numeric verification, polynomial algebra, linear algebra, group theory computations, number theory (primality, factorization), combinatorics. |
| **Literature search (arXiv, Google Scholar)** | "What's already known?" is the first question in any research problem. Mathematicians expect the tool to find relevant papers, theorems, and known results. Without this, it's just a chatbot. | HIGH | arXiv API is free and well-documented. Google Scholar has no official API (scraping is fragile). Semantic Scholar API is the best alternative -- free, structured, good math coverage. Must return: title, authors, year, abstract, arXiv ID/DOI, relevance explanation. |
| **Proof collaboration (suggest approaches, fill gaps)** | The core value proposition. LLMs can suggest proof strategies, identify which known theorems apply, fill routine gaps, and check logical structure. This is why a mathematician would use an AI assistant at all. | HIGH | Must support: suggesting proof strategies (direct, contradiction, induction, contrapositive), identifying applicable theorems, filling routine computational gaps, checking logical flow. Must NOT claim to have "verified" proofs -- LLMs hallucinate math. |
| **Structured reasoning chains** | Mathematical proofs require step-by-step logical structure. The tool must present reasoning in numbered steps with clear justifications, not stream-of-consciousness paragraphs. | LOW | This is mostly prompt engineering. Structure output as: Claim -> Strategy -> Step 1 (justification) -> Step 2 (justification) -> ... -> QED. Map to LaTeX proof environments. |
| **Session state and continuity** | Mathematical research spans days/weeks. A mathematician working on a proof needs to resume where they left off: what's been tried, what worked, what didn't, what's pending. | MEDIUM | Already supported by GSD's STATE.md and .planning/ paradigm. Math-specific state: proof progress (what lemmas are proven, what gaps remain), computation results, literature found, approaches tried and abandoned (with reasons). |
| **Mathematical notation rendering in intermediate output** | While final output is LaTeX, intermediate steps displayed in the terminal should use recognizable mathematical notation where possible, or at minimum well-formatted LaTeX source that a mathematician can read fluently. | LOW | Claude Code terminal supports basic formatting. Use LaTeX source notation in intermediate outputs (mathematicians read LaTeX source fluently). Don't try to render -- just use clean LaTeX source strings. |

### Differentiators (Competitive Advantage)

Features that set get-shit-done-math apart from using raw Claude Code, ChatGPT, or Wolfram Alpha individually. These are the reasons to adopt this tool specifically.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| **Orchestrated research pipeline (search -> prove -> compute -> write)** | No existing tool connects literature search, proof strategy, computation, and LaTeX writing in a single orchestrated flow. Mathematicians currently alt-tab between arXiv, Wolfram Alpha, a LaTeX editor, and ChatGPT. This tool replaces that fragmented workflow with a coordinated pipeline. | HIGH | This IS the product. Each stage feeds the next: literature search informs proof strategy, proof strategy identifies what needs computation, computation results feed back into proof, everything flows into LaTeX output. The orchestrator-agent pattern from GSD maps perfectly here. |
| **Multi-agent proof decomposition** | Complex proofs have subproblems. Decompose a proof into lemmas, farm each lemma to a specialized agent (one does the algebraic manipulation, another handles the topological argument, another verifies edge cases computationally), then synthesize. No other tool does this. | HIGH | Leverage GSD's parallel agent spawning. Each sub-problem gets fresh 200k context. Critical insight: mathematical proofs are naturally decomposable into lemmas -- this maps perfectly to the task-plan-execute pattern. |
| **Literature synthesis (not just search)** | Existing tools find papers. This tool should CONNECT them: "Theorem A from [Smith 2019] combined with Lemma 3 from [Jones 2021] gives you the key ingredient for your proof." This is what a knowledgeable colleague does. | HIGH | Requires: (1) finding relevant papers, (2) extracting key results, (3) identifying connections between them and the user's problem. This is where LLM capability shines -- pattern matching across abstractions. Agent reads multiple paper abstracts and synthesizes. |
| **Approach journal (what was tried, what failed, why)** | Research is as much about knowing what DOESN'T work as what does. The tool should maintain a persistent record of attempted approaches, why they failed, and what was learned. This is invaluable when returning to a problem after days away. | MEDIUM | Extend GSD's state management. After each proof attempt: record approach, outcome, reason for failure/success, what it suggests about alternative approaches. This becomes a "research diary" that prevents repeating dead ends. |
| **Conjecture testing via computation** | Before attempting a full proof, test the conjecture on examples. Automatically generate test cases, run computations, and report: "Conjecture holds for all tested cases" or "Counterexample found: n=17." This saves hours of attempting to prove false statements. | MEDIUM | Generate test cases based on problem domain (small groups, low-dimensional spaces, first N integers, etc.). Run via SageMath/SymPy. Report results with confidence: "Tested 10,000 cases, no counterexample" vs "Counterexample at n=17: here's why." |
| **Bidirectional LaTeX workflow** | Not just output LaTeX, but INPUT existing LaTeX. A mathematician should paste their partial proof in LaTeX, and the tool understands it, identifies gaps, suggests completions. Most LLM tools treat LaTeX as output-only. | MEDIUM | Parse LaTeX input to understand theorem statements, proof structure, where gaps are marked (e.g., "TODO" or "..."). Claude already handles LaTeX well. The key is structured parsing to identify what's proven vs what's claimed vs what's missing. |
| **Computation-proof bridge** | When a computation reveals something (e.g., "this polynomial always factors over Q"), automatically suggest why this might be true and what proof technique could establish it. Bridge the gap between numerical evidence and formal proof. | HIGH | This is the "insight" feature. Requires: (1) interpreting computation results, (2) recognizing patterns, (3) connecting patterns to known theorems or proof techniques. Example: computation shows a sequence is always divisible by 6 -> suggest proof by showing divisibility by 2 and 3 separately via modular arithmetic. |
| **Customizable notation and convention profiles** | Different mathematical subfields have different notational conventions (algebraists write groups multiplicatively, some write additively; analysts use epsilon-delta, some use sequences; topologists have specific conventions for homology). Let users configure their conventions once. | LOW | Store in .planning/ config: notation preferences, theorem numbering style, bibliography style (alpha vs numeric), preferred LaTeX packages, department/journal style requirements. Load into every agent context. |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem valuable but would hurt the tool if built. Deliberately exclude these.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| **Formal proof verification (Lean/Coq/Isabelle integration)** | "Verify proofs are correct with a proof assistant." Seems like the obvious next step. | Formal verification requires translating informal math into formal syntax -- a massive, unsolved problem. The translation itself introduces more errors than it catches for working mathematicians. Lean's Mathlib covers ~50% of undergraduate math; research-level math is mostly unformalized. Building this would consume 10x the effort of everything else combined and deliver value to <1% of users. The proof assistant community is its own specialized world. | Instead: flag logical gaps and questionable steps explicitly. Say "I couldn't verify step 3 -- the claim that f is continuous needs justification" rather than claiming formal verification. Be honest about uncertainty. |
| **Real-time collaboration / multiplayer** | "Two mathematicians working on the same proof simultaneously." | Mathematical research is deeply individual cognitive work. Real-time sync adds massive infrastructure complexity (CRDT, WebSocket, conflict resolution) for a workflow where people naturally work asynchronously. Mathematicians already collaborate via email, shared Overleaf documents, and in-person discussion. | Instead: export-friendly output. Generate LaTeX that drops into Overleaf. Generate summaries shareable via email. Support "hand off to collaborator" by producing a self-contained research state document. |
| **Automated paper submission** | "Submit directly to arXiv/journals." | Submission involves nuanced decisions: which journal, cover letter, formatting to specific style guides, response to referees. Automating this removes human judgment at the most consequential moment. Also, arXiv submission has specific requirements (endorsement, category selection) that change. | Instead: produce submission-ready LaTeX with correct formatting. Let the mathematician handle the actual submission and editorial decisions. |
| **Teaching/tutoring mode** | "Explain concepts step by step for students." | This is a fundamentally different product. Teaching requires pedagogical scaffolding, exercises, assessment, curriculum structure. Building it dilutes the research focus and attracts a completely different user base with different needs. Khan Academy, Brilliant, and many LLM tutors already exist. | Instead: stay focused on research. The "explain this theorem" capability naturally exists within the proof collaboration feature -- but framed as "help me understand this result I need for my proof" rather than "teach me calculus." |
| **Visual/graphical proof construction** | "Drag-and-drop proof trees, visual theorem maps." | Requires a full GUI, which contradicts the CLI-first Claude Code plugin model. Mathematical reasoning doesn't naturally map to visual drag-and-drop (unlike, say, circuit design). The overhead of building and maintaining a GUI is enormous. | Instead: produce well-structured text output with clear logical flow. For diagrams (commutative diagrams, graphs), generate TikZ code that compiles to visual output in the LaTeX document. |
| **General-purpose AI assistant features** | "Also help me write emails, manage my calendar, summarize meetings." | Scope creep. Every feature that isn't mathematical research dilutes agent prompt quality and adds maintenance burden. Claude Code already does general tasks. | Instead: do one thing exceptionally well. If someone wants general AI assistance, they use Claude Code directly. This tool activates specifically for mathematical research workflows. |
| **Plagiarism/novelty checking** | "Check if my result is truly new." | Novelty is domain-specific and nuanced. No automated system can reliably determine if a result is novel -- it requires deep understanding of the field's state of the art. False negatives ("your result is new!" when it isn't) would be embarrassing; false positives ("this is already known" when it isn't) would be discouraging. | Instead: thorough literature search. Present what IS known and let the mathematician determine novelty. Frame it as "here's what I found that's related" rather than "your result is/isn't novel." |
| **Wolfram Alpha wrapper** | "Just call Wolfram Alpha for computations." | Wolfram Alpha's API is paid, rate-limited, and returns results in Wolfram's proprietary format. It's also a black box -- you get an answer but not the method. Local SageMath/SymPy gives full control, is free, and shows its work. | Instead: use Python/SymPy/SageMath locally via Bash tool. This is free, inspectable, customizable, and runs offline. Users who want Wolfram Alpha can use it separately. |

## Feature Dependencies

```
[Problem Intake]
    |
    +--requires--> [Session State & Continuity]
    |
    +--feeds-----> [Literature Search]
    |                  |
    |                  +--feeds-----> [Literature Synthesis]
    |                  |                  |
    |                  +--feeds-----> [Proof Collaboration]
    |                                     |
    +--feeds-----> [Proof Collaboration]  |
    |                  |                  |
    |                  +--feeds-----> [Computation Integration]
    |                  |                  |
    |                  +--feeds-----> [LaTeX Output]
    |                                     |
    +--feeds-----> [Computation Integration]
    |                  |
    |                  +--feeds-----> [Conjecture Testing]
    |                  |
    |                  +--feeds-----> [Computation-Proof Bridge]
    |                  |
    |                  +--feeds-----> [LaTeX Output]
    |
    +--feeds-----> [LaTeX Output]

[Approach Journal] --enhances--> [Proof Collaboration]
[Approach Journal] --enhances--> [Session State & Continuity]

[Bidirectional LaTeX] --enhances--> [Proof Collaboration]
[Bidirectional LaTeX] --enhances--> [LaTeX Output]

[Notation Profiles] --enhances--> [LaTeX Output]
[Notation Profiles] --enhances--> [Proof Collaboration]

[Structured Reasoning] --enhances--> [Proof Collaboration]
[Structured Reasoning] --enhances--> [LaTeX Output]

[Multi-Agent Proof Decomposition] --requires--> [Proof Collaboration]
[Multi-Agent Proof Decomposition] --requires--> [Orchestrated Pipeline]

[Computation-Proof Bridge] --requires--> [Computation Integration]
[Computation-Proof Bridge] --requires--> [Proof Collaboration]

[Conjecture Testing] --requires--> [Computation Integration]

[Literature Synthesis] --requires--> [Literature Search]
```

### Dependency Notes

- **Problem Intake is the root dependency:** Everything else requires a well-structured problem statement. Build this first.
- **Literature Search requires API integration:** arXiv API + Semantic Scholar API. These are external dependencies that need error handling, rate limiting, and fallback behavior.
- **Proof Collaboration is the core value:** Most other features feed into or enhance this. It depends on Problem Intake and benefits enormously from Literature Search results.
- **Computation Integration is independent of Literature:** Can be built in parallel with literature search. Both feed into Proof Collaboration.
- **LaTeX Output depends on everything upstream:** It consumes proof content, computation results, and literature references to produce the final document.
- **Multi-Agent Proof Decomposition requires the orchestrated pipeline:** Can't decompose proofs across agents without the pipeline infrastructure being in place.
- **Approach Journal enhances but doesn't block:** Can be added after core workflow exists. Enhances Proof Collaboration and Session State.
- **Conjecture Testing and Computation-Proof Bridge require Computation Integration:** Build computation first, then layer these analytical features on top.

## MVP Definition

### Launch With (v1)

Minimum viable product -- what's needed to validate that orchestrated mathematical research is valuable.

- [ ] **Problem Intake** -- Structured capture of mathematical problem with domain, known results, desired output. This is the entry point; without it, everything downstream gets wrong context.
- [ ] **Literature Search (arXiv + Semantic Scholar)** -- Find relevant papers and known results. Mathematicians won't trust a tool that doesn't know what's already published.
- [ ] **Proof Collaboration** -- Suggest proof strategies, fill gaps, check logical flow. This is the primary reason mathematicians would use the tool.
- [ ] **Computation Integration (Python/SymPy)** -- Execute computations to verify conjectures and check examples. SageMath as optional upgrade for users who have it installed.
- [ ] **LaTeX Output** -- Produce compilable LaTeX documents. Non-negotiable for mathematicians.
- [ ] **Orchestrated Pipeline** -- Connect search -> prove -> compute -> write in a coordinated flow. This is what differentiates the tool from raw Claude Code.
- [ ] **Session State** -- Track progress across sessions (what's proven, what's pending, what was tried).

### Add After Validation (v1.x)

Features to add once core pipeline is working and validated with real mathematicians.

- [ ] **Literature Synthesis** -- When users report that search results are useful but they want the tool to connect dots between papers. Trigger: users manually synthesizing search results.
- [ ] **Approach Journal** -- When users report losing track of what they've tried. Trigger: users asking "didn't we already try X?"
- [ ] **Conjecture Testing** -- When users manually writing test code. Trigger: users pasting Python code to test before proving.
- [ ] **Bidirectional LaTeX** -- When users pasting partial proofs and wanting gap analysis. Trigger: users copy-pasting LaTeX into the tool regularly.
- [ ] **Notation Profiles** -- When users repeatedly correcting notation. Trigger: users saying "I told you, I use additive notation for groups."
- [ ] **Multi-Agent Proof Decomposition** -- When proofs are complex enough that single-agent collaboration isn't sufficient. Trigger: context window filling up on complex proofs.

### Future Consideration (v2+)

Features to defer until product-market fit is established.

- [ ] **Computation-Proof Bridge** -- Requires deep integration of computation and proof engines. Defer until both are mature.
- [ ] **Formal verification integration (Lean 4)** -- Only after core workflow is excellent. HIGH complexity, niche audience. Consider as opt-in module.
- [ ] **Custom CAS backend support** -- Let users plug in Mathematica, Maple, or other CAS. Defer until SymPy/SageMath proves insufficient.
- [ ] **Reference management integration (BibTeX/Zotero)** -- Nice to have for paper writing. Defer until LaTeX output is mature.
- [ ] **Collaborative research state export** -- Export full research state for sharing with collaborators. Defer until single-user workflow is solid.

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Problem Intake | HIGH | LOW | P1 |
| Literature Search (arXiv + Semantic Scholar) | HIGH | MEDIUM | P1 |
| Proof Collaboration | HIGH | HIGH | P1 |
| Computation Integration (Python/SymPy) | HIGH | MEDIUM | P1 |
| LaTeX Output | HIGH | MEDIUM | P1 |
| Orchestrated Pipeline | HIGH | HIGH | P1 |
| Session State (math-specific) | HIGH | LOW | P1 |
| Structured Reasoning Chains | MEDIUM | LOW | P1 |
| Literature Synthesis | HIGH | HIGH | P2 |
| Approach Journal | MEDIUM | LOW | P2 |
| Conjecture Testing | MEDIUM | MEDIUM | P2 |
| Bidirectional LaTeX | MEDIUM | MEDIUM | P2 |
| Notation Profiles | LOW | LOW | P2 |
| Multi-Agent Proof Decomposition | HIGH | HIGH | P2 |
| Computation-Proof Bridge | MEDIUM | HIGH | P3 |
| Formal Verification (Lean 4) | LOW | VERY HIGH | P3 |
| Custom CAS Backends | LOW | MEDIUM | P3 |
| BibTeX/Zotero Integration | LOW | LOW | P3 |

**Priority key:**
- P1: Must have for launch -- the tool is unusable without these
- P2: Should have, add when possible -- significant value, build after core works
- P3: Nice to have, future consideration -- only after product-market fit

## Competitor Feature Analysis

| Feature | Wolfram Alpha / Mathematica | Lean 4 / Mathlib | ChatGPT / Claude (raw) | Elicit / Semantic Scholar | Our Approach |
|---------|---------------------------|-------------------|------------------------|--------------------------|--------------|
| **Computation** | Best-in-class symbolic & numeric computation. Gold standard CAS. | No computation -- pure logic. | Can write code but doesn't execute reliably. No CAS integration. | None. | Execute SymPy/SageMath locally via Bash. Not as powerful as Mathematica but free, inspectable, and integrated into workflow. |
| **Proof assistance** | Step-by-step solutions for textbook problems. Weak on research-level proofs. | Formal proof verification. Extremely rigorous but requires learning Lean syntax. | Can suggest strategies but hallucinates steps, no verification, no persistence. | None. | LLM-powered proof collaboration with honest uncertainty. Don't claim verification -- flag gaps explicitly. Persist proof state across sessions. |
| **Literature search** | None. | None. | Can discuss papers from training data but can't search live. Hallucinates citations. | Best-in-class academic search. Semantic Scholar API is excellent. | Use Semantic Scholar API + arXiv API for live search. LLM synthesizes results in context of user's problem. |
| **LaTeX output** | Exports to LaTeX but in Wolfram's style. | Lean syntax, not LaTeX. | Generates LaTeX snippets but not full documents. | Exports citations in BibTeX. | Full compilable LaTeX documents with proper theorem environments, bibliography, custom notation. |
| **Orchestrated workflow** | No workflow -- single query/response. | Build proofs incrementally but no research workflow. | No workflow -- conversation only. | Search workflow only. | Full pipeline: intake -> search -> prove -> compute -> write. Each stage feeds the next. This is the differentiator. |
| **Persistence** | Notebooks persist but no research state. | Lean files persist proofs. | Conversation history only. Fragile, context-limited. | Saved searches and collections. | Full research state: problem, literature found, approaches tried, proofs in progress, computations run, LaTeX drafts. |
| **Mathematical domain knowledge** | Encyclopedic for computable math. | Formalized math in Mathlib (~700k LOC). | Broad but shallow. Hallucinates. | Paper metadata, not mathematical content. | LLM knowledge enhanced by live literature search and computation verification. Honest about limitations. |
| **Target user** | Students, engineers, applied mathematicians. | Formal verification researchers, advanced proof engineers. | Everyone (not specialized). | Researchers across all fields. | Working mathematicians (pure and applied) doing research. |

### Key Competitive Insights

1. **No existing tool orchestrates the full mathematical research workflow.** Mathematicians currently use 4-5 separate tools (arXiv, CAS, LaTeX editor, LLM chatbot, paper manager). The orchestrated pipeline is genuinely novel.

2. **Raw LLM chat is the real competitor.** Most mathematicians experimenting with AI are using ChatGPT or Claude directly. The value proposition is: structured workflow > ad-hoc conversation. This needs to be demonstrably true, not just theoretically.

3. **Wolfram Alpha owns computation.** Don't compete on CAS power. Compete on integration -- SymPy computation as part of a workflow is more valuable than Mathematica computation in isolation, even though Mathematica is more powerful.

4. **Lean owns formal verification.** Don't compete here at all. Different user base, different problem, different skillset. Acknowledge Lean exists, respect its role, and stay in a different lane.

5. **Semantic Scholar + arXiv own search.** Don't rebuild search. USE their APIs and add value through synthesis and contextualization.

6. **Honest uncertainty is a differentiator.** Every LLM math tool has a hallucination problem. Being explicitly honest about what's verified vs uncertain vs speculative builds trust with mathematicians who are trained to demand rigor. This is cultural alignment, not just a feature.

## Sources

- Training data knowledge of: Wolfram Alpha/Mathematica features, Lean 4/Mathlib capabilities, arXiv API, Semantic Scholar API, SymPy/SageMath capabilities, LaTeX ecosystem, ChatGPT/Claude mathematical capabilities, Elicit features
- Project context from: PROJECT.md, ARCHITECTURE.md, README.md (existing GSD system)
- Confidence: MEDIUM -- all findings based on training data (cutoff early 2025). Web verification was unavailable. Key claims to verify: Semantic Scholar API current status, arXiv API rate limits, SymPy current feature set, any new math-AI tools launched in 2025-2026.

### Verification Gaps

- **Semantic Scholar API**: Verify current rate limits, endpoint availability, and math-specific coverage. Training data suggests it's free with API key and has good coverage, but this should be confirmed.
- **arXiv API**: Verify current endpoint (arxiv.org/api/query), rate limits, and any changes to search capabilities.
- **New competitors**: Tools like Julius AI, Minerva, DeepMind's AlphaProof (announced mid-2024) may have matured. Check for new entrants in the AI-math space.
- **Claude's math capabilities**: Claude Opus 4's mathematical reasoning may have improved significantly since training cutoff. Test actual performance on research-level problems.
- **SageMath current version**: Verify compatibility with current Python versions and feature availability for advanced algebraic computations.

---
*Feature research for: AI-powered mathematical research assistant*
*Researched: 2026-02-08*
