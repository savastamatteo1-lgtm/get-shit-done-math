# Architecture Patterns

**Domain:** AI-powered mathematical research assistant
**Researched:** 2026-02-08
**Overall confidence:** MEDIUM (web verification unavailable; architecture derived from domain knowledge + existing GSD patterns)

## Recommended Architecture

### Overview

The system extends GSD's orchestrator-agent-document architecture into the mathematical research domain. Five specialized agents operate under a thin math orchestrator, communicating exclusively through markdown files in `.planning/math/`. The orchestrator never does math -- it routes, validates state transitions, and spawns agents with fresh context.

```
User
  |
  v
/math:research  (command entry point)
  |
  v
math-orchestrator  (thin workflow coordinator)
  |
  +---> [1] problem-intake      writes PROBLEM.md
  |
  +---> [2] literature-agent    reads PROBLEM.md, writes LITERATURE.md
  |
  +---> [3] proof-agent         reads PROBLEM.md + LITERATURE.md, writes PROOF.md
  |         ^                    (may loop back for more literature)
  |         |
  +---> [4] computation-agent   reads PROOF.md, writes COMPUTATION.md
  |         |                    (returns results to proof-agent if needed)
  |         v
  +---> [5] latex-agent         reads all artifacts, writes OUTPUT.tex + OUTPUT.pdf
```

### Why This Shape

Mathematical research is not a linear pipeline. It is an iterative loop with backtracking:

1. You state a problem.
2. You search for prior work.
3. You attempt a proof, realize you need a lemma.
4. You compute examples to build intuition.
5. You revise the proof with computational evidence.
6. You search for one more reference.
7. You write it up.

The architecture must support **forward flow** (the happy path) and **backward loops** (agent A requests agent B re-run with new constraints). The orchestrator handles this by maintaining a state machine in `STATE.md` with explicit transition rules.

## Component Boundaries

| Component | Responsibility | Reads | Writes | Spawns |
|-----------|---------------|-------|--------|--------|
| **math-orchestrator** | Route workflow, validate state, spawn agents, handle loops | STATE.md, config.json | STATE.md (transitions) | All agents |
| **problem-intake** (inline) | Capture problem statement, domain, known constraints | User input | PROBLEM.md | None |
| **literature-agent** | Search arXiv/Scholar, retrieve abstracts, identify relevant theorems | PROBLEM.md, PROOF.md (if re-query) | LITERATURE.md | None |
| **proof-agent** | Develop proof strategy, structure arguments, identify gaps | PROBLEM.md, LITERATURE.md, COMPUTATION.md | PROOF.md | None |
| **computation-agent** | Execute symbolic/numerical computation via Python/SageMath | PROOF.md (computation requests), PROBLEM.md | COMPUTATION.md | None |
| **latex-agent** | Generate publication-quality LaTeX from all artifacts | PROBLEM.md, LITERATURE.md, PROOF.md, COMPUTATION.md | OUTPUT.tex, OUTPUT.pdf | None |

### Key Boundary Rules

1. **Agents never call other agents.** Only the orchestrator spawns agents.
2. **Agents communicate only through files.** No in-memory state sharing.
3. **Each agent gets fresh 200k context.** Load only the files it needs via @-references.
4. **The orchestrator never does math.** It reads status fields from artifact files and routes.

## Data Flow

### Primary Flow: Problem to LaTeX

```
PROBLEM.md ──────────────────────────────────────────────────────> latex-agent
    |                                                                  ^
    v                                                                  |
literature-agent ──> LITERATURE.md ──────────────────────────────> latex-agent
                         |                                             ^
                         v                                             |
                     proof-agent ──> PROOF.md ───────────────────> latex-agent
                         |               |                             ^
                         v               v                             |
                     (needs computation?) ──> computation-agent        |
                                                  |                    |
                                                  v                    |
                                          COMPUTATION.md ──────────────+
```

### Artifact Format and Schema

Each artifact file follows a structured markdown format with machine-readable YAML frontmatter that the orchestrator uses for routing decisions.

#### PROBLEM.md

```markdown
---
status: defined
domain: [algebra|analysis|topology|number-theory|combinatorics|applied|...]
type: [proof|computation|exploration|conjecture-testing]
tags: [list of mathematical keywords]
---

# Problem Statement

[Precise mathematical statement in LaTeX-compatible notation]

## Known Results

[What the user already knows about this problem]

## Constraints

[Time bounds, computation limits, output preferences]

## Goals

- [ ] Find relevant prior work
- [ ] Develop proof/solution strategy
- [ ] Verify computationally (if applicable)
- [ ] Write formal LaTeX output
```

#### LITERATURE.md

```markdown
---
status: [searching|complete|needs-refinement]
query_count: [number of searches performed]
paper_count: [number of relevant papers found]
last_updated: [timestamp]
---

# Literature Review

## Search Queries

| Query | Source | Results | Relevant |
|-------|--------|---------|----------|
| [query text] | arXiv/Scholar | [count] | [count] |

## Key References

### [Author et al., Year] - [Short Title]
- **arXiv ID:** [if applicable]
- **Relevance:** [why this matters to the problem]
- **Key Result:** [theorem/lemma statement]
- **Technique:** [proof method used]
- **Applicability:** [how it connects to our problem]

## Relevant Theorems

### Theorem: [Name/Label]
**Source:** [Reference]
**Statement:** [Mathematical statement]
**Usefulness:** [How this helps the proof]

## Gaps Identified

- [Areas where no prior work was found]
- [Open questions adjacent to our problem]
```

#### PROOF.md

```markdown
---
status: [strategy|in-progress|gaps-identified|complete|stuck]
approach: [direct|contradiction|induction|construction|...]
confidence: [high|medium|low]
computation_needed: [true|false]
literature_needed: [true|false]
---

# Proof Development

## Strategy

[High-level approach and why it was chosen]

## Proof Structure

### Step 1: [Description]
**Status:** [complete|in-progress|blocked]
**Depends on:** [other steps, literature, computation]

[Mathematical argument]

### Step 2: [Description]
...

## Gaps

### Gap 1: [Description]
**Type:** [lemma-needed|computation-needed|reference-needed|insight-needed]
**Blocking:** [which proof steps]
**Suggested resolution:** [what to try]

## Computation Requests

### Request 1: [Description]
**Purpose:** [why this computation helps]
**Input:** [mathematical objects, parameters]
**Expected output:** [what we need to see]
**Tool:** [sympy|sage|mathematica|numpy]
```

#### COMPUTATION.md

```markdown
---
status: [pending|running|complete|error]
runtime: [python|sage|mathematica]
last_run: [timestamp]
---

# Computation Results

## Environment

- Python [version] with SymPy [version]
- SageMath [version] (if used)

## Computation 1: [Description]

**Request from:** PROOF.md, Gap/Request [N]
**Purpose:** [why]

### Code

```python
[executable code]
```

### Output

```
[actual output from execution]
```

### Interpretation

[What this result means for the proof]

## Summary for Proof Agent

| Computation | Result | Implication |
|-------------|--------|-------------|
| [desc] | [result] | [what it means] |
```

#### OUTPUT.tex (managed by latex-agent)

Standard LaTeX document with `\documentclass{article}`, proper theorem environments, bibliography from LITERATURE.md references, proofs from PROOF.md, computation appendices from COMPUTATION.md.

### State Machine

The orchestrator maintains a state machine. Each state maps to which agent to spawn next.

```
INTAKE ──> LITERATURE_SEARCH ──> PROOF_DEVELOPMENT ──> LATEX_OUTPUT
               ^       |              |        ^            |
               |       v              v        |            v
               +-- (needs more    COMPUTATION  |          DONE
                    references)       |        |
                                      +--------+
                                   (results ready)
```

**State transitions in STATE.md:**

```yaml
current_state: PROOF_DEVELOPMENT
history:
  - INTAKE -> LITERATURE_SEARCH (problem defined)
  - LITERATURE_SEARCH -> PROOF_DEVELOPMENT (12 papers found)
transitions_available:
  - COMPUTATION (proof-agent flagged computation_needed: true)
  - LITERATURE_SEARCH (proof-agent flagged literature_needed: true)
  - LATEX_OUTPUT (proof-agent set status: complete)
```

The orchestrator reads `PROOF.md` frontmatter after proof-agent completes. If `computation_needed: true`, it spawns computation-agent. If `literature_needed: true`, it spawns literature-agent with the specific queries from PROOF.md's Gaps section. If `status: complete`, it proceeds to latex-agent.

## Patterns to Follow

### Pattern 1: Artifact-Driven Routing

**What:** The orchestrator reads YAML frontmatter from artifact files to decide the next step. Agents set status flags; orchestrators read them.

**When:** Every state transition.

**Why:** Keeps the orchestrator thin. The orchestrator does not parse mathematical content. It reads machine-parseable metadata that agents write.

**Example:**

```bash
# Orchestrator reads proof status
PROOF_STATUS=$(grep '^status:' .planning/math/PROOF.md | cut -d' ' -f2)
NEEDS_COMPUTATION=$(grep '^computation_needed:' .planning/math/PROOF.md | cut -d' ' -f2)
NEEDS_LITERATURE=$(grep '^literature_needed:' .planning/math/PROOF.md | cut -d' ' -f2)

if [ "$NEEDS_COMPUTATION" = "true" ]; then
  # Spawn computation-agent
elif [ "$NEEDS_LITERATURE" = "true" ]; then
  # Spawn literature-agent with refinement context
elif [ "$PROOF_STATUS" = "complete" ]; then
  # Spawn latex-agent
elif [ "$PROOF_STATUS" = "stuck" ]; then
  # Present to user: proof is stuck, offer options
fi
```

### Pattern 2: Incremental Artifact Append

**What:** When an agent re-runs (e.g., literature-agent called a second time for a refinement query), it appends to the existing artifact rather than overwriting. Each run is a numbered section.

**When:** Any agent invoked more than once in a session.

**Why:** Mathematical research is cumulative. The second literature search builds on the first. The proof-agent needs to see all prior computation results, not just the latest.

**Example:**

```markdown
# Literature Review

## Search Round 1 (initial)
[original results]

## Search Round 2 (refinement: requested by proof-agent)
**Query context:** Proof needs lemma about bounded operators on L^2 spaces
[new results]
```

### Pattern 3: Computation Sandbox

**What:** The computation-agent executes code in a sandboxed Bash environment. All code runs locally. Results are captured as text output and written to COMPUTATION.md.

**When:** proof-agent flags `computation_needed: true` with specific requests.

**Why:** Mathematical computation must be reproducible. The code and its output are both artifacts. The proof-agent can reference specific computations.

**Example execution flow:**

```bash
# computation-agent writes a Python script
cat > /tmp/math_computation_1.py << 'PYEOF'
from sympy import *
x = Symbol('x')
result = integrate(exp(-x**2), (x, -oo, oo))
print(f"Result: {result}")
print(f"Simplified: {simplify(result)}")
PYEOF

python3 /tmp/math_computation_1.py
```

The computation-agent captures stdout, interprets the result mathematically, and writes both code and interpretation to COMPUTATION.md.

### Pattern 4: Problem Decomposition for Large Proofs

**What:** For complex proofs, the proof-agent decomposes into lemmas. Each lemma gets its own section in PROOF.md with independent status tracking. The orchestrator can re-run the proof-agent with a focus directive (e.g., "work on Lemma 3").

**When:** Proof complexity exceeds what a single agent pass can handle.

**Why:** A 200k context window is large but not infinite. A proof of a major theorem may require multiple focused passes rather than one monolithic attempt.

**Example:**

```markdown
## Proof Structure

### Lemma 1: Boundedness of T on L^p
**Status:** complete
**Proved in:** Pass 1

### Lemma 2: Density argument
**Status:** complete
**Proved in:** Pass 1

### Lemma 3: Extension to general domains
**Status:** in-progress
**Depends on:** Lemma 1, Lemma 2, Computation 2
**Notes:** Need computation results before completing

### Main Theorem
**Status:** blocked
**Depends on:** Lemma 1, 2, 3
```

### Pattern 5: Literature Search via Bash Tools

**What:** The literature-agent searches arXiv using the arXiv API directly via `curl` or via the `arxiv` Python package. Google Scholar is searched via web scraping or the Scholarly Python package.

**When:** Literature search phase.

**Why:** No MCP server exists for arXiv. The agent must use Bash to execute HTTP requests or Python scripts. This is consistent with the existing GSD pattern where agents use Bash for external operations.

**Example:**

```bash
# arXiv API search
curl -s "http://export.arxiv.org/api/query?search_query=all:banach+space+operator+theory&max_results=20" \
  | python3 -c "
import sys, xml.etree.ElementTree as ET
tree = ET.parse(sys.stdin)
ns = {'atom': 'http://www.w3.org/2005/Atom'}
for entry in tree.findall('.//atom:entry', ns):
    title = entry.find('atom:title', ns).text.strip()
    arxiv_id = entry.find('atom:id', ns).text.strip()
    summary = entry.find('atom:summary', ns).text.strip()[:200]
    print(f'---')
    print(f'Title: {title}')
    print(f'ID: {arxiv_id}')
    print(f'Abstract: {summary}...')
"
```

## Anti-Patterns to Avoid

### Anti-Pattern 1: Monolithic Proof Agent

**What:** Putting all mathematical reasoning (literature search, proof, computation, LaTeX) into a single agent.

**Why bad:** A single agent trying to search arXiv, develop a proof, run computations, and write LaTeX will exhaust its context window rapidly. Each capability requires different tools, different prompt engineering, and different error handling.

**Instead:** Separate agents with clear boundaries. The proof-agent only does proof development. It flags when it needs help (literature, computation) and the orchestrator dispatches.

### Anti-Pattern 2: Stateless Agent Spawning

**What:** Spawning agents without loading their prior work into context.

**Why bad:** The proof-agent on its second pass must see what it proved on the first pass. Without PROOF.md loaded via @-reference, the agent starts from scratch and contradicts its own prior work.

**Instead:** Always load the current state of all relevant artifacts via @-references in the agent spawn prompt. The orchestrator's job is to construct the right context package for each agent invocation.

### Anti-Pattern 3: Orchestrator Does Math

**What:** The orchestrator parsing mathematical content to make routing decisions.

**Why bad:** The orchestrator cannot reliably evaluate whether a proof step is correct or whether a computation result supports a theorem. It does not have mathematical reasoning capability in workflow-coordination mode.

**Instead:** Agents write machine-readable status flags in YAML frontmatter. The orchestrator reads flags (`status: complete`, `computation_needed: true`), never mathematical content. Mathematical judgment stays in agents.

### Anti-Pattern 4: Unbounded Iteration Loops

**What:** Allowing proof-agent and literature-agent to loop indefinitely (proof needs literature, literature reveals new approach, proof restarts, needs more literature...).

**Why bad:** Infinite loops burn tokens and context. Mathematical research can genuinely loop forever.

**Instead:** Cap iterations at 3 loops per pair (proof-literature, proof-computation). After the cap, present the current state to the user with options: continue, redirect, or accept partial results. Track loop count in STATE.md.

### Anti-Pattern 5: Mixing LaTeX Generation with Proof Development

**What:** Having the proof-agent write LaTeX as it develops the proof.

**Why bad:** Proof development is exploratory and messy. LaTeX output should be clean, formal, and complete. Mixing the two means the proof-agent wastes context on formatting, and the LaTeX output contains the mess of exploration.

**Instead:** The proof-agent writes informal but structured markdown. The latex-agent translates to formal LaTeX. This separation means the proof-agent can be informal, use shorthand, and focus on mathematics. The latex-agent can focus on presentation, theorem environments, and bibliography.

## Scalability Considerations

| Concern | Simple Problem | Multi-lemma Proof | Research Paper |
|---------|---------------|-------------------|----------------|
| **Context budget** | Single pass per agent | Multiple focused passes on lemma subsets | Multiple passes + chunked literature |
| **Literature volume** | 5-10 papers | 20-50 papers | 50-100+ papers, needs summary agent |
| **Computation complexity** | One-off calculation | Multiple interdependent computations | Extensive numerical experiments |
| **LaTeX complexity** | 2-3 page note | 10-15 page proof | 20-40 page paper with appendices |
| **Agent invocations** | 4-5 total | 8-15 total | 15-30+ total |
| **State file size** | Trivial | Moderate (all fit in context) | May need chunking/summarization |

### Handling Large Problems

For research-paper-scale work, add a **summarization step** between passes:

1. After literature-agent finds 50+ papers, a summary pass distills them into a `LITERATURE-SUMMARY.md` (key theorems only, 5-10 pages). The proof-agent loads the summary, not the full review.
2. After proof-agent completes 5+ lemmas, a summary pass creates `PROOF-OUTLINE.md` (structure + key results, no detailed steps). Subsequent proof passes load the outline plus the specific lemma under development.
3. COMPUTATION.md results are summarized into a table at the top. Detailed code/output lives in appendix sections that agents load only when needed.

## Session Directory Structure

```
.planning/
  math/
    STATE.md              # Orchestrator state machine
    PROBLEM.md            # Problem definition (user input)
    LITERATURE.md         # Literature search results (cumulative)
    PROOF.md              # Proof development (cumulative)
    COMPUTATION.md        # Computation results (cumulative)
    OUTPUT.tex            # Final LaTeX output
    OUTPUT.pdf            # Compiled PDF (if pdflatex available)
    sessions/
      session-001/        # Archive of a completed research session
        PROBLEM.md
        LITERATURE.md
        PROOF.md
        COMPUTATION.md
        OUTPUT.tex
```

**Why `.planning/math/` not `.planning/phases/`:** Mathematical research is not software engineering phases. The workflow is iterative and non-linear. Phases imply sequential completion; math research loops. A flat directory with cumulative artifacts better matches the domain.

**Why `sessions/`:** When the user starts a new problem, the current artifacts archive into a numbered session directory. This preserves history without cluttering the active workspace.

## Build Order (Dependencies)

Components should be built in this order based on dependency chains:

### Wave 1: Foundation (no dependencies)
1. **Problem intake workflow** -- Captures problem statement from user, writes PROBLEM.md. This is a simple orchestrator step (interactive questions), not a full agent. Build first because everything downstream depends on PROBLEM.md existing.
2. **State management** -- STATE.md schema, transition logic in orchestrator, session archival. Build early because the orchestrator needs this for all routing decisions.
3. **Math-tools utility** -- Extensions to gsd-tools.js for math workflows (state transitions, artifact validation, session management).

### Wave 2: Core Agents (depend on Wave 1)
4. **Literature agent** -- Depends on PROBLEM.md format. The most independent agent -- searches external sources, writes structured results. Build second because it is self-contained and exercisable without proof/computation.
5. **Computation agent** -- Depends on PROBLEM.md format. Also relatively independent -- executes Python/SageMath scripts, captures output. Can be tested standalone with sample computation requests.

### Wave 3: Integration Agent (depends on Wave 2)
6. **Proof agent** -- Depends on LITERATURE.md and COMPUTATION.md formats. The most complex agent. Reads prior work, develops mathematical arguments, identifies gaps, requests computation. Build after literature and computation agents so it can request their outputs.

### Wave 4: Output (depends on Wave 3)
7. **LaTeX agent** -- Depends on all artifact formats. Translates the accumulated state into formal LaTeX. Build last because it consumes all other artifacts.

### Wave 5: Orchestration (depends on all agents)
8. **Math orchestrator workflow** -- Wires all agents together with state machine routing, iteration caps, user interaction points. Build last because it needs all agents to exist. (Though a skeleton orchestrator that calls agents in sequence should be built alongside Wave 2 for integration testing.)

### Build Order Rationale

```
Wave 1: Foundation
  PROBLEM.md schema + STATE.md + math-tools
     |
Wave 2: Independent Agents (parallel)
  literature-agent    computation-agent
     |                     |
Wave 3: Core Agent
  proof-agent (needs lit + comp formats)
     |
Wave 4: Output Agent
  latex-agent (needs all formats)
     |
Wave 5: Full Orchestration
  math-orchestrator (wires everything together)
```

This order means each wave is testable in isolation before integration. The literature agent can be tested with real arXiv queries against a sample PROBLEM.md without needing the proof or computation agents to exist.

## How Mathematical State Flows Between Agents

### Information Flow Model

Agents do not share memory. They share files. The critical design question is: what does each agent need to read, and what does it produce?

```
problem-intake
  writes: PROBLEM.md (problem statement, domain, known results, goals)
  reads:  user input only

literature-agent (pass 1)
  reads:  PROBLEM.md (to know what to search for)
  writes: LITERATURE.md (papers, theorems, techniques found)

literature-agent (pass N, refinement)
  reads:  PROBLEM.md + PROOF.md (to find Gap section with specific queries)
  writes: LITERATURE.md (appends new search round)

proof-agent (pass 1)
  reads:  PROBLEM.md + LITERATURE.md
  writes: PROOF.md (strategy, proof structure, gaps, computation requests)

proof-agent (pass N)
  reads:  PROBLEM.md + LITERATURE.md + COMPUTATION.md + PROOF.md (prior state)
  writes: PROOF.md (updates statuses, fills gaps, adds new steps)

computation-agent
  reads:  PROOF.md (computation requests section) + PROBLEM.md (domain context)
  writes: COMPUTATION.md (code, output, interpretation)

latex-agent
  reads:  PROBLEM.md + LITERATURE.md + PROOF.md + COMPUTATION.md
  writes: OUTPUT.tex + OUTPUT.pdf
```

### What Crosses Agent Boundaries

| Information Type | Where Produced | Where Consumed | Format |
|-----------------|----------------|----------------|--------|
| Problem statement | PROBLEM.md | All agents | Natural language + LaTeX math notation |
| Domain classification | PROBLEM.md frontmatter | Orchestrator (routing), literature-agent (query construction) | YAML enum |
| Search results | LITERATURE.md | proof-agent, latex-agent | Structured markdown with theorem statements |
| Proof structure | PROOF.md | computation-agent (requests), latex-agent (formalization) | Hierarchical markdown with status flags |
| Computation requests | PROOF.md Gaps section | computation-agent | Structured request blocks |
| Computation results | COMPUTATION.md | proof-agent (to fill gaps), latex-agent (appendices) | Code blocks + interpretation text |
| Status flags | All artifact frontmatter | Orchestrator (routing decisions) | YAML boolean/enum |
| Loop counters | STATE.md | Orchestrator (iteration caps) | YAML integers |

### Mathematical Notation Convention

All agents use LaTeX notation inline within markdown. This serves two purposes:
1. **Human readability:** Mathematicians can read `$\int_0^1 f(x)\,dx$` naturally.
2. **LaTeX agent compatibility:** The latex-agent can extract mathematical expressions without translation.

Agents should write mathematical expressions using standard LaTeX: `$...$` for inline, `$$...$$` or `\[ ... \]` for display. Theorem environments are written as markdown headers in PROOF.md and converted to `\begin{theorem}...\end{theorem}` by the latex-agent.

## Sources

- Existing GSD architecture (`.planning/codebase/ARCHITECTURE.md`) -- HIGH confidence
- GSD orchestrator-agent patterns (workflows, agents, commands) -- HIGH confidence
- arXiv API structure -- MEDIUM confidence (from training data, needs verification against current docs)
- SymPy/SageMath capabilities -- MEDIUM confidence (from training data, stable libraries)
- Mathematical research workflow patterns -- MEDIUM confidence (domain knowledge, not from specific tools)
- Scholarly/Google Scholar access patterns -- LOW confidence (API availability and terms may have changed)

**Note:** WebSearch and WebFetch were unavailable during research. arXiv API details, Python library versions, and Google Scholar access patterns should be verified during implementation phases. The architectural patterns (component boundaries, data flow, state machine) are high confidence because they derive from the proven GSD orchestrator-agent paradigm applied to a well-understood domain workflow.

---

*Architecture research: 2026-02-08*
