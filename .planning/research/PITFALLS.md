# Pitfalls Research

**Domain:** AI-powered mathematical research tools (LLM-based proof collaboration, computation orchestration, literature search, LaTeX output)
**Researched:** 2026-02-08
**Confidence:** HIGH (core failure modes well-documented in ML research literature; computation integration pitfalls from established CAS ecosystem)

## Critical Pitfalls

### Pitfall 1: Confident Hallucination of False Mathematical Statements

**What goes wrong:**
LLMs generate mathematically false statements with the same confident tone as true ones. Unlike code (which fails visibly at runtime), a wrong lemma citation, an incorrect inequality direction, or a fabricated theorem name can silently poison an entire proof chain. The downstream mathematician trusts the statement because it "sounds right" and is formatted correctly, then builds further reasoning on a false foundation.

Specific failure modes:
- Citing theorems that do not exist ("by the Matsumura-Katz theorem...")
- Stating a theorem correctly but applying it outside its hypotheses (e.g., applying a result for compact spaces to a non-compact space)
- Inverting inequality directions or quantifier order (for-all vs. there-exists swaps)
- Fabricating plausible-sounding lemma names that match the style of a subfield
- Generating "proofs" that skip the hard step, papering over the gap with phrases like "it follows that" or "by a standard argument"

**Why it happens:**
LLMs are trained on pattern completion, not logical deduction. Mathematical text follows predictable linguistic patterns ("by Theorem X, we have..."), and the model completes these patterns based on co-occurrence statistics rather than truth verification. The model has no internal proof checker. Its confidence calibration is based on token probability, not mathematical validity.

**How to avoid:**
1. **Never present LLM mathematical claims without provenance markers.** Every theorem cited must include: source paper/textbook, exact statement, and whether the citation was verified against a database (arXiv, MathSciNet) or is LLM-generated.
2. **Implement a "confidence tier" system in proof collaboration output:**
   - VERIFIED: Statement checked against computation or known database
   - SUGGESTED: LLM-generated, plausible but unverified
   - SPECULATIVE: LLM exploring, may be wrong
3. **Require the proof agent to separately state each hypothesis being used** before applying a theorem, so the mathematician can check applicability.
4. **Build a "claim extraction" post-processor** that pulls out every factual mathematical assertion and presents them as a checkable list.

**Warning signs:**
- Proof steps that use "clearly" or "it is easy to see" to skip justification
- Theorem names that return no results on MathSciNet/zbMATH
- Proofs that are suspiciously short for known-hard results
- The model claiming a result "is well-known" without providing a reference

**Phase to address:**
Phase 1 (proof collaboration agent design). The confidence tier system must be baked into the agent's output format from day one. Retrofitting provenance tracking is extremely painful.

---

### Pitfall 2: The Illusion of Verification (Circular Confidence)

**What goes wrong:**
When an LLM "checks" its own proof, it tends to agree with itself. If the proof collaboration agent generates a proof and then a separate LLM call is asked "is this proof correct?", the verifier will almost always say yes -- because the same linguistic patterns that made the proof sound convincing to generate also make it sound convincing to verify. This creates a dangerous illusion: the system reports "proof verified" when no actual verification occurred.

This is especially insidious in multi-agent architectures (like get-shit-done-math) where spawning a "verifier agent" feels like genuine separation of concerns, but both agents share the same fundamental inability to do rigorous logical verification.

**Why it happens:**
LLM-based verification is essentially asking "does this text look like a correct proof?" rather than "is this proof logically valid?" The model evaluates linguistic plausibility, not logical soundness. Two LLM calls hitting the same blind spot will both miss the same error. This is a fundamental limitation, not an engineering problem.

**How to avoid:**
1. **Never claim "verified" for LLM-checked proofs.** Use terminology like "reviewed" or "no issues detected" with explicit disclaimer that LLM review is not formal verification.
2. **Ground verification in computation wherever possible.** If a proof claims a particular identity, compute both sides numerically for specific values. If it claims a bound, check the bound computationally for small cases.
3. **Use structural checks, not semantic ones.** Instead of asking "is this proof correct?", check: Are all variables introduced before use? Does each step cite a justification? Are the hypotheses of cited theorems satisfied? These structural checks are more reliable than asking an LLM to evaluate truth.
4. **Log which claims are LLM-only vs. computationally grounded.** The mathematician must know what has been actually checked vs. what has only been LLM-reviewed.
5. **Make formal verification (Lean/Coq) a future integration point.** Design the proof output format so it could eventually feed into a formal proof assistant, even if that integration is out of scope for v1.

**Warning signs:**
- A "verification" step that never finds errors
- Verification agent that rubber-stamps proofs in under 2 seconds
- No computational spot-checks in the verification pipeline
- Users developing false confidence in unverified proofs

**Phase to address:**
Phase 1 (proof collaboration agent) and Phase 2 (computation integration). The proof agent must clearly distinguish its output tiers from the start. Computation integration provides the first real verification mechanism.

---

### Pitfall 3: Computation Results Presented Without Reproducibility

**What goes wrong:**
The computation agent runs Python/SageMath or Mathematica code, gets a result, and reports it to the user. But the code that produced the result is either lost, buried in a log, or not saved in a reproducible format. When the mathematician later needs to verify the computation, modify parameters, or include it in a paper, they cannot reproduce what the agent did.

Worse: numerical computations may be sensitive to floating-point precision, library versions, or random seeds. An answer of "the integral evaluates to 3.14159..." means nothing if the computation used insufficient precision or an unstable numerical method.

**Why it happens:**
AI coding agents are optimized for "get the answer" rather than "create reproducible science." The natural flow is: write throwaway code, run it, report the result, move on. This is antithetical to mathematical research where reproducibility and exact computation are requirements, not nice-to-haves.

**How to avoid:**
1. **Every computation must produce a persistent, runnable artifact.** Save all computation scripts to a designated directory (e.g., `.math/computations/`) with metadata (timestamp, purpose, dependencies, exact library versions).
2. **Prefer symbolic computation over numerical whenever possible.** SageMath/SymPy symbolic results are exact and verifiable. Numerical results should include error bounds or precision statements.
3. **Require the computation agent to state its method and limitations.** "Computed numerically using scipy.integrate.quad with tolerance 1e-10" is useful. "The integral equals 2.718..." is not.
4. **Version-pin computation environments.** Document the exact SymPy/SageMath/Mathematica version used, since results can differ across versions (especially for symbolic simplification and integration).
5. **Separate "exploration computations" from "publication computations."** Exploration can be fast and loose. Anything going into a proof or paper needs full reproducibility metadata.

**Warning signs:**
- Computation results appear in proof text with no linked source code
- Numerical results stated without precision/tolerance information
- No computation artifacts directory in the project
- Agent using `eval()` or inline computation rather than saved scripts

**Phase to address:**
Phase 2 (computation agent design). The artifact storage pattern must be designed before any computations run. This is an architecture decision, not a feature.

---

### Pitfall 4: Destroying Mathematical Nuance Through Oversimplification

**What goes wrong:**
LLMs flatten the rich semantic structure of mathematics into natural language summaries that lose critical distinctions. Examples:
- "convergence" without specifying pointwise vs. uniform vs. in measure vs. almost sure
- "continuous" without specifying the topology
- "differentiable" without specifying how many times, or in what sense (classical, weak, distributional)
- "the space X" without tracking whether it is a topological space, metric space, normed space, or Banach space -- and which specific structure is being used
- Eliding the difference between a function and its equivalence class in L^p

These are not pedantic distinctions. They are the substance of mathematical reasoning. A proof that works for uniform convergence may completely fail for pointwise convergence.

**Why it happens:**
LLMs learn from a corpus that includes textbooks (precise), lecture notes (somewhat precise), blog posts (imprecise), and forum answers (often sloppy). The model averages over these precision levels. It also optimizes for readability, and precise mathematical language is verbose and dense. The model naturally "simplifies" toward more readable (but less precise) formulations.

**How to avoid:**
1. **Prompt engineering for precision.** The proof collaboration agent's system prompt must explicitly require: "Always specify the type of convergence. Always state the topology. Always distinguish between strict and non-strict inequalities."
2. **Build domain-specific precision checklists.** For each mathematical subfield (analysis, algebra, topology, probability), maintain a checklist of common precision failures. The agent should self-check against these.
3. **Use structured output for mathematical objects.** Instead of free-text descriptions, represent mathematical objects with explicit attributes: `{type: "function", domain: "R^n", codomain: "R", regularity: "C^2", additional: "compactly supported"}`.
4. **Require the agent to explicitly state "which version" when using overloaded terms.** "We use continuity in the sense of sequential continuity with respect to the weak topology" not just "f is continuous."

**Warning signs:**
- Proof text that uses bare terms like "converges," "continuous," "bounded" without qualification
- Contradictions that arise from ambiguous terminology (a step that works for one interpretation but not another)
- The agent producing correct-looking proofs for false statements (often caused by equivocating on definitions)

**Phase to address:**
Phase 1 (proof collaboration agent). The precision requirements must be embedded in the agent's system prompt and output format. This is extremely difficult to retrofit because imprecise intermediate results propagate.

---

### Pitfall 5: LaTeX Output That Looks Right But Is Mathematically Wrong

**What goes wrong:**
The LaTeX output agent produces beautifully typeset documents where the mathematical content has been subtly altered during the formatting pass. Common failures:
- Operator precedence errors: `$a + b \cdot c$` rendered as `$(a + b) \cdot c$` by adding parentheses "for clarity"
- Subscript/superscript confusion: `$x_i^2$` (x-sub-i squared) vs. `$x_{i^2}$` (x-sub-i-squared) vs. `$x^{2_i}$` (nonsensical)
- Missing or extra negation signs in long expressions
- Changing `\subset` (proper subset) to `\subseteq` (subset or equal) or vice versa
- Using `\implies` where `\iff` was meant
- Flattening display math with multiple alignment points into a single unaligned equation, losing the correspondence between steps
- Dropping conditions from theorem statements during reformatting ("for all x > 0" becoming "for all x")

**Why it happens:**
The LaTeX generation agent treats the content as text to be formatted rather than semantics to be preserved. LLMs are good at producing valid LaTeX syntax but lack the mathematical understanding to verify that the formatted output preserves the exact mathematical meaning of the input. Any "cleanup" or "improvement" pass risks changing meaning.

**How to avoid:**
1. **Separate content from formatting strictly.** The proof agent should produce mathematical content in a structured intermediate representation. The LaTeX agent should only handle formatting, never modifying mathematical content.
2. **Implement a diff-and-review step.** After LaTeX generation, automatically diff the mathematical content (not the formatting) between input and output. Flag any semantic changes for human review.
3. **Use LaTeX macros for mathematical objects** rather than raw symbols. Define `\converges`, `\implies`, `\propsubset` etc. as macros. This prevents the formatting agent from "improving" symbol choices.
4. **Never let the LaTeX agent "simplify" or "clarify" mathematical expressions.** Its job is typesetting, not mathematics.
5. **Include a compilation check.** Run `pdflatex` on the output and catch LaTeX errors. This catches syntax issues but not semantic ones -- still valuable.

**Warning signs:**
- LaTeX output that compiles but contains different mathematical statements than the proof agent produced
- The LaTeX agent adding "helpful" parentheses or rearranging expressions
- Symbol substitutions (e.g., `\le` to `<` or `\in` to `\subset`)
- Theorem statements in the formatted paper that differ from the proof collaboration output

**Phase to address:**
Phase 3 (LaTeX output agent). But the intermediate representation between proof agent and LaTeX agent must be designed in Phase 1, because it determines what information is preserved.

---

### Pitfall 6: Literature Search That Confabulates References

**What goes wrong:**
The literature search agent fabricates plausible-sounding paper references that do not exist. This is one of the most well-documented LLM failure modes. The agent generates:
- Papers with real author names but fake titles (combining known authors with plausible topics)
- Papers with real titles but wrong authors, years, or journal names
- Papers that exist but whose results are misstated
- ArXiv IDs that are syntactically valid but point to unrelated papers or do not exist
- Textbook theorem references with wrong theorem numbers or wrong editions

**Why it happens:**
LLMs have memorized many paper titles, author names, and result descriptions, but not the exact mappings between them. When asked for a reference, the model pattern-matches to generate a plausible-looking citation rather than retrieving an exact record. The model cannot distinguish between "I remember this paper" and "I am generating a plausible paper."

**How to avoid:**
1. **Every reference must be verified against a real database.** The literature agent must query arXiv API, Semantic Scholar API, or Google Scholar and confirm that the paper exists with the stated title and authors before including it.
2. **Include DOI or arXiv ID for every citation** and verify these resolve correctly.
3. **Separate "suggested references" (LLM-generated, unverified) from "confirmed references" (API-verified).** Never mix these categories.
4. **When citing a specific result from a paper, include the exact theorem/proposition number** and flag it as "LLM-recalled, verify against source" unless the actual paper text was retrieved and parsed.
5. **Build an API integration layer** for arXiv, Semantic Scholar, and optionally MathSciNet/zbMATH. Do not rely on the LLM's training data for bibliographic information.

**Warning signs:**
- References with no DOI or arXiv ID
- ArXiv IDs that do not match the claimed paper when checked
- Author names that are well-known but not in the claimed combination
- Year of publication that seems wrong for the topic (e.g., a deep learning paper attributed to 1998)
- The agent providing references without being asked (often hallucinated gap-filling)

**Phase to address:**
Phase 1 (literature search agent). API integration for reference verification is not optional -- it is the core value proposition. An LLM-only literature search that confabulates references is worse than useless; it actively harms the researcher.

---

### Pitfall 7: Notation Inconsistency Across Agent Boundaries

**What goes wrong:**
Different agents in the pipeline use different notation conventions for the same mathematical objects, creating confusion and errors when results are combined:
- The proof agent writes `\mathbb{R}^n` but the computation agent outputs `R^n` or `RR^n` (SageMath notation)
- One agent uses `f'(x)` (Lagrange notation) while another uses `\frac{df}{dx}` (Leibniz notation) for the same derivative
- The literature agent reports a theorem using the paper's notation (`\alpha`-Holder continuous) while the proof agent has been using a different variable name (`\beta`-Holder continuous)
- Index conventions differ: 0-indexed (Python/computation) vs. 1-indexed (most mathematical writing)
- Matrix conventions: row-major vs. column-major, or `A_{ij}` vs. `a_{ij}` vs. `A(i,j)`
- The same symbol used for different things across agents (e.g., `\sigma` meaning a permutation in one context and a standard deviation in another)

**Why it happens:**
Each agent operates in a fresh 200k context window (per the GSD architecture). There is no shared state for notation conventions. Each agent inherits notation from its own prompt, the papers it reads, or the computation environment it uses. Without explicit coordination, notation diverges.

**How to avoid:**
1. **Create a per-project notation registry** stored in `.math/notation.md` or `.planning/notation.md`. This file defines the canonical notation for all mathematical objects in the current research project.
2. **Include the notation registry in every agent's context.** Each agent's prompt should reference this file and be instructed to use only the registered notation.
3. **Build a notation translation layer** between computation environments and mathematical writing. SageMath's `RR` maps to `\mathbb{R}`, Python's 0-indexing maps to 1-indexing with explicit offset, etc.
4. **When an agent introduces new notation, require it to register it** rather than using it ad hoc.
5. **Use consistent variable naming conventions** specified in the project setup: "We use lowercase Greek for scalars, uppercase Latin for matrices, lowercase Latin for vectors."

**Warning signs:**
- The same mathematical object referred to differently in proof text vs. computation output
- Index-off-by-one errors when computation results are inserted into proofs
- LaTeX output containing inconsistent notation for the same concept
- The notation registry (if it exists) not being loaded into agent contexts

**Phase to address:**
Phase 1 (project setup / problem intake). The notation registry must be created when the research problem is defined. Every subsequent agent must receive it. This is an architecture decision that affects all agents.

---

### Pitfall 8: Conflating LLM Pattern Matching with Mathematical Proof

**What goes wrong:**
The system (and its users) begin to treat the proof collaboration agent's output as actual mathematical proof rather than as a heuristic sketch that requires human verification. This is the most philosophically dangerous pitfall. Specific manifestations:
- Publishing results where the "proof" was primarily generated by the LLM without sufficient human verification
- Trusting multi-step reasoning chains that "look right" but contain subtle logical gaps
- Accepting proof strategies that work for easy cases but fail for the general case the LLM was asked about
- Confusing "the LLM couldn't find a counterexample" with "we have proved the statement"

**Why it happens:**
LLMs produce output that mimics the structure and language of mathematical proof. For simple results, the output is often correct, building user trust. Users then extrapolate this trust to harder results where the LLM is far less reliable. The fluent, confident prose style of LLM output masks uncertainty. There is no visual distinction between a step the model is confident about and one it is guessing at.

**How to avoid:**
1. **Frame the proof agent as a "proof collaborator" or "proof sketch generator," never as a "prover."** Naming matters. The UX should constantly reinforce that the human mathematician is the one doing the proving.
2. **Mark every proof step with a justification type:**
   - AXIOM/DEFINITION: Starting point, no justification needed
   - CITED: References a specific published result
   - COMPUTED: Verified by computation
   - DEDUCED: Follows from previous steps by a named logical rule
   - SUGGESTED: LLM's suggestion, needs human verification
3. **Design the UX to make the "SUGGESTED" steps visually prominent** (not hidden). The hard part of a proof is exactly where the LLM is least reliable.
4. **Include explicit "gap markers"** where the agent cannot complete a step. "I believe this step requires [X], but I cannot rigorously justify it" is vastly more useful than a fabricated justification.
5. **Track proof status at the project level.** A proof with any SUGGESTED steps is "in progress," not "complete."

**Warning signs:**
- Users accepting proof agent output without reading it carefully
- No SUGGESTED or gap markers in proof output (indicates the agent is overconfident, not that the proof is complete)
- The agent never saying "I don't know how to prove this step"
- Proof output that lacks explicit justification types

**Phase to address:**
Phase 1 (proof collaboration agent). This is the single most important design decision for the entire project. The proof agent's output format and confidence system define what the tool is.

---

### Pitfall 9: Unsafe Code Execution from LLM-Generated Computation

**What goes wrong:**
The computation agent generates Python/SageMath code that the system executes via Bash. LLM-generated code can:
- Execute arbitrary system commands (os.system, subprocess)
- Read/write arbitrary files
- Make network requests (exfiltrating data or downloading malicious payloads)
- Enter infinite loops consuming resources
- Consume excessive memory (e.g., computing factorial of 10^9)
- Import libraries that have side effects
- Accidentally delete files or overwrite important data

In the context of a math tool, most of these are accidental (the LLM generates bad code, not malicious code), but the execution environment must still be safe.

**Why it happens:**
The LLM generates code based on patterns from its training data, which includes code with side effects, file I/O, and network access. It does not have a concept of "this code will be auto-executed in the user's environment." The convenience of auto-execution (run computation, get result) creates pressure to minimize friction, which conflicts with safety.

**How to avoid:**
1. **Sandbox computation execution.** Run generated Python code in a restricted environment: no network access, no file system access outside the project directory, resource limits (CPU time, memory).
2. **Whitelist safe imports.** Only allow known-safe mathematical libraries: sympy, sage, numpy, scipy, matplotlib, mpmath. Block os, subprocess, socket, requests, etc.
3. **Implement resource limits.** Set timeouts (e.g., 60 seconds), memory caps, and output size limits for all computations.
4. **Require user approval for first-time execution patterns.** If the computation agent tries to do something it has not done before (new import, file write), pause for confirmation.
5. **Separate computation into "sandboxed exploration" and "user-approved scripts."** Quick computations run in sandbox. Anything saved or published requires explicit approval.
6. **Log all executed code.** Every computation run should be logged with input, output, and the full code that was executed.

**Warning signs:**
- Computation code importing os, subprocess, or socket
- Computations that take more than 60 seconds
- Computation output that includes file paths or network addresses
- The agent running code without showing it to the user first

**Phase to address:**
Phase 2 (computation agent). The sandbox must be designed before any auto-execution is enabled. This is a safety-critical architecture decision.

---

### Pitfall 10: Losing Research State Across Long Investigations

**What goes wrong:**
Mathematical research is inherently long-running: a single problem may take weeks or months. The research state (what has been tried, what worked, what failed, which approaches were abandoned and why) is critical context that must persist across many sessions. In the GSD architecture, each agent gets a fresh 200k context window. If the research state is not carefully maintained in documents, critical context is lost:
- Re-exploring approaches that were already tried and failed
- Losing track of which lemmas have been proven vs. which are still conjectured
- Forgetting why a particular approach was abandoned (and re-attempting it)
- Contradictory assumptions accumulating across sessions without detection
- The proof "evolving" across sessions in ways that introduce inconsistencies

**Why it happens:**
The fresh-context-per-agent design (a strength for software engineering tasks) becomes a liability for long-running mathematical research. In software, each task is relatively independent. In mathematics, each proof step depends on everything before it. The document-as-interface paradigm must capture not just results but reasoning history.

**How to avoid:**
1. **Design a structured "research journal" document** that tracks: problem statement, known results, attempted approaches (with outcomes), current proof state (which steps are complete, which are in progress), open questions, and abandoned approaches (with reasons).
2. **Keep the proof state as a structured document** (not free-form notes). Each lemma/proposition should have a status: PROVEN, IN_PROGRESS, CONJECTURED, ABANDONED(reason).
3. **Require agents to read and update the research journal** at the start and end of every session. This is the memory of the project.
4. **Implement contradiction detection.** When new results are added, check them against existing results for logical consistency (at minimum, check for direct contradictions in stated hypotheses or conclusions).
5. **Design the document format to fit within agent context windows.** The research journal must be comprehensive but not so large that it exceeds the 200k token limit when combined with the agent's instructions and other context.

**Warning signs:**
- An agent proposing an approach that was already tried and abandoned
- Contradictory claims in different parts of the proof state
- The research journal growing too large to fit in a single agent's context
- Agents producing output that contradicts previous session results

**Phase to address:**
Phase 1 (research state management / problem intake). The research journal format is foundational architecture. It must be designed before any proof work begins, because it defines how knowledge persists across the agent boundary.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Skipping reference verification against APIs | Faster literature search results | Hallucinated references pollute the research, erode user trust | Never in production; acceptable only in offline/demo mode with clear "UNVERIFIED" labels |
| Free-form proof text instead of structured output | Faster agent development, more natural output | Cannot reliably extract claims, track proof status, or verify consistency; impossible to integrate with formal verification later | MVP proof-of-concept only; must migrate to structured output before any real use |
| Running computation without sandboxing | Simpler architecture, faster iteration | Security risk, resource exhaustion, potential data loss from runaway code | Local development only; must sandbox before any user-facing release |
| Single notation convention hardcoded in prompts | Works for one research project | Breaks when the user works in a different subfield or notation tradition | Never; build the notation registry even if it starts small |
| Storing proof state as unstructured markdown | Easy to write, human-readable | Cannot programmatically check consistency, track status, or detect contradictions | Phase 1 only; structured format needed by Phase 2 |
| LLM-only "verification" without computational checks | Provides some error detection cheaply | Creates false confidence; users trust "verified" label when no real verification occurred | Acceptable if clearly labeled as "LLM review" not "verification" |

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| arXiv API | Treating arXiv search as comprehensive (it only covers preprints, not all published math) | Use arXiv for preprints + Semantic Scholar for broader coverage + MathSciNet for authoritative metadata; clearly indicate source for each reference |
| SageMath | Assuming SageMath is installed system-wide; using `sage` command without checking availability | Check for SageMath at startup; provide fallback to SymPy (pure Python, always available); document the capability difference |
| Mathematica | Assuming free availability; generating Wolfram Language code without checking license | Make Mathematica optional; detect `wolframscript` availability; never require Mathematica for core functionality; provide SymPy equivalents |
| SymPy symbolic computation | Trusting `simplify()` to always produce the simplest form (it uses heuristics, not guarantees) | Use domain-specific simplification (`trigsimp`, `radsimp`, `factor`); always verify with `equals()` or numerical evaluation; never trust that two different-looking expressions are unequal just because `simplify` did not reduce them to the same form |
| LaTeX compilation | Assuming the user has a full TeX distribution (texlive-full is 5GB+) | Generate LaTeX that works with minimal distributions (texlive-base); list required packages in document preamble; provide compilation instructions |
| Google Scholar | Scraping Google Scholar directly (violates ToS, gets IP-blocked quickly) | Use Semantic Scholar API (free, documented, rate-limited) or SerpAPI (paid) for programmatic access; never scrape Google Scholar |
| MathSciNet / zbMATH | Assuming free access (both require institutional subscriptions) | Use for verification only when available; fall back to DOI resolution (CrossRef API, free) for basic metadata verification |

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Loading entire research journal into every agent context | Slow agent startup, hitting context limits, truncated instructions | Summarize older entries; only load current proof state + relevant history; implement a "context budget" that allocates tokens | When research journal exceeds ~50k tokens (typically after 2-3 weeks of active research) |
| Running symbolic computation synchronously in proof agent | Agent hangs waiting for computation; timeout kills the agent | Run computation asynchronously; set timeouts; provide partial results; never block the proof agent on computation | Complex symbolic integrals, series expansions, or Grobner basis computations (can take minutes to hours) |
| Storing all computation artifacts without cleanup | Disk usage grows; project directory becomes cluttered; git history bloated | Separate ephemeral exploration computations from persistent results; implement cleanup for exploration artifacts; keep only final/significant computations | After ~100 computation runs (typically a few days of active use) |
| Re-running literature search for every session | Slow startup; redundant API calls; hitting rate limits | Cache search results with TTL; only re-search when problem statement changes or user explicitly requests refresh | First session of each day if not cached; immediately on first use |
| Full LaTeX compilation for every edit | Multi-second delay per change; disrupts iterative workflow | Use incremental compilation or preview-only mode for editing; full compilation only for final output | When document exceeds ~20 pages or uses complex packages (tikz, pgfplots) |

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Auto-executing LLM-generated Python without sandboxing | Arbitrary code execution on user's machine; data exfiltration; file deletion | Whitelist imports; sandbox execution environment; resource limits; require user approval for dangerous operations |
| Storing API keys (arXiv, Semantic Scholar, etc.) in project files | Credential exposure via git commit | Use environment variables or system keychain; add API key patterns to .gitignore; never store credentials in .planning/ |
| Including user's unpublished mathematical results in LLM context | Potential exposure of unpublished research to model training (if using API-based LLM) | Clearly document data handling policy; note that Claude Code operates locally but API calls send data to Anthropic; let users opt out of specific content being sent |
| Fetching and executing remote computation scripts | Supply chain attack via compromised dependency or malicious computation template | Never fetch executable code from remote sources; all computation templates must be local; verify any external mathematical libraries before use |
| Parsing LaTeX from untrusted sources (downloaded papers) | LaTeX can contain `\write18` commands that execute shell code | Disable `\write18` / shell-escape in LaTeX compilation; parse LaTeX content as text, never compile untrusted LaTeX directly |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Presenting LLM-generated proofs as "verified" or "complete" | Mathematician publishes or builds on false results; career and scientific damage | Always show confidence tier (VERIFIED/SUGGESTED/SPECULATIVE); make unverified steps visually distinct; require explicit human sign-off before marking any proof as complete |
| Mixing computation output notation with mathematical notation | Confusion when SageMath's `sqrt(2)` appears next to LaTeX's `\sqrt{2}`; user must mentally translate | Auto-translate computation output to the project's notation convention; present both forms (computation and typeset) side by side |
| Overwhelming the mathematician with too many suggestions | Decision fatigue; the tool becomes a distraction rather than an aid | Limit proof suggestions to top 2-3 approaches with clear rationale; let user request more if needed; quality over quantity |
| Hiding the reasoning behind proof suggestions | Mathematician cannot evaluate whether a suggestion is worth pursuing | Always show why the agent suggests an approach: "This approach may work because [X] is similar to [known result Y], where [technique Z] was used" |
| Not supporting the mathematician's existing workflow | Tool requires abandoning familiar tools (personal LaTeX templates, notation conventions, bibliography style) | Import existing notation conventions; support custom LaTeX preambles and templates; work with the user's BibTeX/BibLaTeX database rather than generating a new one |
| Printing long proofs as a single block of text | Impossible to navigate; user cannot find the step they need to verify | Structure output with clear section headers, numbered steps, and collapsible sections; provide a "proof outline" view alongside the detailed proof |

## "Looks Done But Isn't" Checklist

- [ ] **Literature search:** Often missing verification that cited papers actually exist -- verify each reference against arXiv/Semantic Scholar API
- [ ] **Proof collaboration:** Often missing explicit hypotheses for cited theorems -- verify every "by Theorem X" includes the theorem's conditions and confirms they apply
- [ ] **Proof collaboration:** Often missing justification for "obvious" steps -- verify no step relies solely on "clearly" / "it follows that" / "by a standard argument" without a citable justification or computation
- [ ] **Computation:** Often missing reproducibility metadata -- verify every computation output links to a saved script with version-pinned dependencies
- [ ] **Computation:** Often missing precision statements -- verify numerical results include tolerance/precision information; symbolic results include simplification method used
- [ ] **LaTeX output:** Often missing semantic preservation -- verify the mathematical content in the LaTeX output matches the proof agent's output exactly (diff the claims, not the formatting)
- [ ] **LaTeX output:** Often missing compilation test -- verify the LaTeX actually compiles without errors on a standard distribution
- [ ] **Notation:** Often missing consistency check -- verify the same mathematical object uses the same notation throughout all outputs (proof, computation, LaTeX)
- [ ] **Research state:** Often missing abandoned-approach logging -- verify the research journal records why approaches were abandoned, not just that they were
- [ ] **Overall:** Often missing the "mathematician has reviewed this" step -- verify no proof or result is marked as final without explicit human sign-off

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Hallucinated references published in a paper | HIGH | Retract or issue erratum; manually verify all references in published work; rebuild reference database from verified sources only |
| False proof accepted as correct | HIGH | Identify the false step; determine what downstream results depend on it; re-prove or retract affected results; add the failure case to the project's "known pitfalls" log |
| Notation inconsistency across proof document | MEDIUM | Create notation registry retroactively; do a full pass to normalize notation; may require re-verifying proof steps where notation change alters meaning |
| Lost computation reproducibility | MEDIUM | Re-run computations from memory/notes; implement artifact storage going forward; accept that some historical computations may not be reproducible |
| Oversimplified mathematical statement in proof | MEDIUM | Identify which precision was lost; determine if the proof still holds with the correct (more precise) statement; if not, redo the affected proof steps |
| Research state lost between sessions | LOW-MEDIUM | Reconstruct from git history of .planning/ files; interview the mathematician to recapture abandoned approaches; implement the research journal going forward |
| Unsafe code execution from computation agent | LOW-HIGH (depends on damage) | Assess what ran; check for file modifications, data exfiltration; implement sandboxing immediately; review git history for any committed changes from the rogue execution |
| LaTeX semantic errors in published paper | MEDIUM | Issue erratum with corrected mathematics; implement the semantic diff checker; re-generate LaTeX from the proof agent's structured output |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Confident hallucination of false statements | Phase 1: Proof agent design | Proof output includes confidence tiers; no mathematical claim lacks provenance marking |
| Circular confidence / illusory verification | Phase 1: Proof agent + Phase 2: Computation | LLM "verification" is labeled as "review"; computational spot-checks exist for all numerical/algebraic claims |
| Computation without reproducibility | Phase 2: Computation agent | Every computation output links to a saved, runnable script with version metadata |
| Destroying mathematical nuance | Phase 1: Proof agent prompts | Spot-check proof output for bare terms; precision checklist passes |
| LaTeX semantic corruption | Phase 3: LaTeX agent + Phase 1: intermediate format | Semantic diff between proof output and LaTeX output shows no mathematical changes |
| Confabulated references | Phase 1: Literature agent | Every reference has a verified DOI or arXiv ID; zero references fail API verification |
| Notation inconsistency | Phase 1: Problem intake / setup | Notation registry exists and is loaded into every agent context |
| Conflating LLM output with proof | Phase 1: Proof agent UX | Proof output uses justification types; SUGGESTED steps are visually prominent |
| Unsafe code execution | Phase 2: Computation agent | Import whitelist enforced; execution sandboxed; resource limits in place |
| Lost research state | Phase 1: Research state management | Research journal format defined; agents read/update it; contradiction detection exists |

## Sources

- Extensive ML research literature on LLM mathematical reasoning failures (GSM8K benchmark analysis, MATH benchmark, MiniF2F, etc.) -- HIGH confidence from training data
- Known issues with LLM reference hallucination (widely documented across legal, medical, and academic domains) -- HIGH confidence
- SageMath/SymPy documentation on symbolic computation limitations (simplify heuristics, precision handling) -- HIGH confidence from training data
- arXiv API documentation and Semantic Scholar API patterns -- MEDIUM confidence (API specifics should be verified against current docs)
- Formal verification community discussions on the gap between informal and formal proof -- HIGH confidence from training data
- LaTeX-specific issues from TeX StackExchange and mathematical typesetting best practices -- HIGH confidence from training data
- Security considerations for LLM-generated code execution (established best practices from the AI coding tool ecosystem) -- HIGH confidence

**Confidence note:** Web search was unavailable during this research. All findings are based on training data (cutoff: May 2025). The core failure modes of LLM mathematical reasoning are well-established in the research literature and unlikely to have changed fundamentally. However, specific API details (arXiv, Semantic Scholar) and tool versions should be verified against current documentation during implementation phases.

---
*Pitfalls research for: AI-powered mathematical research tools*
*Researched: 2026-02-08*
