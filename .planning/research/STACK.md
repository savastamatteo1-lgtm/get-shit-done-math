# Technology Stack

**Project:** get-shit-done-math
**Researched:** 2026-02-08
**Overall Confidence:** MEDIUM (web verification tools unavailable; recommendations based on training data through mid-2025 and project constraints; versions flagged for validation)

## Critical Context: This Is a Claude Code Plugin

This project is **not** a standalone application. It is a Claude Code plugin distributed via npx. This fundamentally constrains the stack:

- **No server, no database, no bundler** -- agents run inside Claude Code's sandbox
- **Computation happens via Bash tool** -- agents spawn Python/SageMath/Mathematica as subprocesses
- **Literature search happens via WebFetch/WebSearch** -- agents call APIs through Claude Code's built-in tools
- **The "stack" is mostly agent prompts (markdown) + a thin Node.js utility layer** -- matching the existing GSD architecture
- **Python libraries are user-installed prerequisites**, not npm dependencies

The real technology decisions are: which APIs to call, which Python libraries to require, and how to structure agent prompts to use them effectively.

---

## Recommended Stack

### Core Runtime (Inherited from GSD)

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Node.js | >= 16.7.0 | Plugin installer, gsd-tools.js utilities | Inherited from GSD; no reason to change | HIGH |
| esbuild | ^0.24.0 | Hook bundling (dev only) | Inherited from GSD | HIGH |
| Markdown | -- | Agent definitions, commands, workflows | Core GSD paradigm: document-as-interface | HIGH |

**No new npm dependencies needed.** The existing zero-production-dependency philosophy is correct for a Claude Code plugin. All computation and API calls are delegated to agents via Bash/WebFetch tools.

### Literature Search APIs

| Technology | Access Method | Purpose | Why | Confidence |
|------------|---------------|---------|-----|------------|
| arXiv API (Atom/RSS) | WebFetch to `export.arxiv.org/api/query` | Search preprints by subject, author, keyword | Free, no auth required, covers all math preprints. The arXiv API returns Atom XML with full metadata (title, abstract, authors, categories, PDF link). Rate limit: ~1 request per 3 seconds. | HIGH |
| Semantic Scholar API | WebFetch to `api.semanticscholar.org/graph/v1` | Citation graphs, related papers, abstract search | Free tier (100 req/sec unauthenticated, more with API key). Returns structured JSON with citation counts, references, TLDRs. Better for discovering related work than arXiv's keyword search. | HIGH |
| Google Scholar | WebSearch tool (Claude Code built-in) | Broad academic search including books, theses | No official API, but Claude Code's WebSearch covers Google Scholar results. Use as fallback when arXiv/Semantic Scholar miss older or non-preprint sources. | MEDIUM |
| ZBMATH Open API | WebFetch to `zbmath.org/api/v1` | Search reviewed mathematics literature | Free, open API. Covers reviewed publications (not preprints). Provides MSC classification codes, which are valuable for categorizing mathematical subfields. Less well-known but highly relevant for pure math. | LOW -- verify API availability and current status |
| MathSciNet | Not usable | Reviewed math literature | Requires institutional subscription. Cannot be accessed programmatically from a local plugin without credentials. Exclude from scope. | HIGH (exclusion confidence) |

**Recommendation:** Use arXiv API as primary (preprints, free, comprehensive), Semantic Scholar as secondary (citations, related work, structured data), and Claude Code's WebSearch as tertiary fallback. Do NOT attempt to scrape Google Scholar -- it blocks automated access and violates ToS.

**arXiv API Query Format (for agent prompts):**
```
GET http://export.arxiv.org/api/query?search_query=all:{query}&start=0&max_results=10
```
The response is Atom XML. Agents parse this via Claude's native text processing -- no XML library needed since Claude can read the structured text directly.

**Semantic Scholar API Query Format:**
```
GET https://api.semanticscholar.org/graph/v1/paper/search?query={query}&fields=title,abstract,authors,citationCount,references&limit=10
```
Returns JSON. Consider requesting an API key for higher rate limits (registered keys get 1 req/sec sustained vs 100/5min burst for anonymous).

### Computation: Python Libraries (User Prerequisites)

These are **not npm dependencies**. They run on the user's machine, invoked by agents via the Bash tool.

| Library | Version | Purpose | Why | Confidence |
|---------|---------|---------|-----|------------|
| Python | >= 3.10 | Runtime for all computation | Universal. 3.10+ for match statements and modern typing. | HIGH |
| SymPy | >= 1.13 | Symbolic algebra, calculus, number theory, combinatorics | The standard Python CAS. Handles symbolic differentiation, integration, series, limits, polynomial manipulation, linear algebra, number theory. Pure Python, no compilation needed. | MEDIUM -- verify latest version |
| NumPy | >= 1.26 | Numerical computation, matrix operations | Standard numerical library. Required by many other packages. Essential for numerical verification of symbolic results. | MEDIUM -- verify latest version |
| SciPy | >= 1.13 | Numerical methods (optimization, integration, ODE solvers) | Builds on NumPy for applied math: numerical integration, optimization, interpolation, FFT, linear algebra routines not in NumPy. | MEDIUM -- verify latest version |
| Matplotlib | >= 3.9 | Plotting and visualization | Agents may need to generate plots to visualize results. Standard choice, works everywhere. | MEDIUM -- verify latest version |
| mpmath | >= 1.3 | Arbitrary precision arithmetic | Ships with SymPy but can be used standalone. Essential for verifying results to hundreds of digits, computing special functions, high-precision numerical integration. | MEDIUM -- verify latest version |

**Optional (power users):**

| Library | Version | Purpose | When to Use | Confidence |
|---------|---------|---------|-------------|------------|
| SageMath | >= 10.3 | Full CAS with algebra, number theory, algebraic geometry | When SymPy is insufficient: algebraic number fields, elliptic curves, modular forms, group theory. SageMath wraps PARI/GP, GAP, Singular, Maxima. Significant installation burden (multi-GB). | LOW -- verify current version |
| Mathematica / Wolfram Engine | latest | Proprietary CAS | When user has a license. Superior for some symbolic computations, pattern matching, and special functions. Agents invoke via `wolframscript -code`. | HIGH (for approach, not version) |
| NetworkX | >= 3.3 | Graph theory computations | Graph algorithms: connectivity, coloring, planarity, spectral graph theory. Only needed for graph theory problems. | LOW -- verify version |

**Why SymPy over SageMath as default:** SymPy is pure Python, pip-installable, lightweight (~30MB), and covers 80% of computational needs. SageMath is a 2-8GB install requiring conda or system packages. For a Claude Code plugin, the barrier to entry must be low. Support SageMath as an optional backend for users who already have it installed.

### LaTeX Generation and Manipulation

| Technology | Access Method | Purpose | Why | Confidence |
|------------|---------------|---------|-----|------------|
| Claude's native LaTeX | Agent prompts | Generate LaTeX from scratch | Claude is already excellent at writing LaTeX. The agent's prompt engineering is the "technology" here. No library needed for generation. | HIGH |
| SymPy `latex()` | Python via Bash | Convert symbolic expressions to LaTeX | `sympy.latex(expr)` produces publication-quality LaTeX from any SymPy expression. Use when computation agent produces results that need to go into the LaTeX document. | HIGH |
| Python `textwrap` / string ops | Python via Bash | Template-based LaTeX assembly | For structured documents (theorems, proofs, sections), Python string formatting with LaTeX templates is simpler than any library. | HIGH |

**What NOT to use for LaTeX:**

| Anti-choice | Why Not |
|-------------|---------|
| KaTeX / MathJax (npm) | These are rendering engines for browsers. We are generating LaTeX source, not rendering it. Wrong tool entirely. |
| LaTeX.js (npm) | LaTeX-to-HTML compiler. Again, we want to produce .tex files, not render them. |
| Pandoc | Heavyweight document conversion. Our output is native LaTeX, not converted from another format. |
| pylatex (Python) | Overly abstracted. Mathematicians want to see and control the LaTeX directly, not work through a Python abstraction layer. Claude can write LaTeX directly -- adding a library in between adds complexity without value. |

**Key Insight:** LaTeX generation is a prompt engineering problem, not a library problem. The computation agent should produce SymPy expressions, call `sympy.latex()` for those, and the writing agent should compose the full document. Claude writes better LaTeX than any template library.

### Mathematical Reasoning with LLMs

| Technology | Access Method | Purpose | Why | Confidence |
|------------|---------------|---------|-----|------------|
| Claude (via agent system) | GSD orchestrator-agent pattern | Primary reasoning engine | Claude is the LLM. The plugin exists within Claude Code. The "mathematical reasoning" capability is Claude itself, guided by carefully engineered agent prompts. No external LLM API needed. | HIGH |
| Chain-of-thought prompting | Agent prompt design | Structured proof reasoning | Mathematical reasoning improves dramatically with explicit step-by-step reasoning chains. Agent prompts must enforce: state assumptions, show each deduction, verify intermediate results. | HIGH |
| Computation-verification loop | Agent workflow design | Self-checking via computation | Agent proposes symbolic result -> computation agent verifies numerically -> if mismatch, reasoning agent revises. This is the core quality assurance pattern. | HIGH |

**What NOT to use:**

| Anti-choice | Why Not |
|-------------|---------|
| OpenAI API / external LLM | We are inside Claude Code. Adding another LLM adds latency, cost, API key management, and no clear benefit. Claude is the reasoning engine. |
| LangChain / LlamaIndex | Orchestration frameworks for LLM apps. GSD already IS the orchestration framework. Adding LangChain would duplicate the orchestrator layer and add massive dependency weight for no gain. |
| AutoGPT / CrewAI patterns | Multi-agent frameworks. GSD's orchestrator-agent pattern already handles this better for our use case (fresh context, document-driven state, background execution). |
| Lean/Coq/Isabelle | Formal proof assistants. Explicitly out of scope (PROJECT.md). High complexity, steep learning curve, and would require users to install heavy toolchains. Consider for v2. |

### Infrastructure / Distribution

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| npm / npx | latest | Plugin distribution | Inherited from GSD. `npx get-shit-done-math` installs to `~/.claude/` | HIGH |
| Git | >= 2.30 | Version control, atomic commits per task | Inherited from GSD. Research state committed to `.planning/` | HIGH |
| Claude Code | latest | Host runtime | The plugin runs inside Claude Code. Agents use Read, Write, Bash, WebFetch, WebSearch, Task tools. | HIGH |

---

## Alternatives Considered

### Literature Search

| Category | Recommended | Alternative | Why Not Alternative |
|----------|-------------|-------------|---------------------|
| Preprint search | arXiv API (direct HTTP) | `arxiv` Python package | Adding a Python dependency for what is a single HTTP GET call adds installation burden. WebFetch can call the API directly. The `arxiv` package is a thin wrapper we do not need. |
| Citation graph | Semantic Scholar API | OpenAlex API | OpenAlex is newer and fully open, but Semantic Scholar has better math coverage, TLDRs, and more mature API. Could add OpenAlex as fallback later. |
| Broad search | Claude's WebSearch | SerpAPI / Google Scholar scraping | WebSearch is built into Claude Code. SerpAPI requires API key and payment. Scraping violates Google ToS. |

### Computation

| Category | Recommended | Alternative | Why Not Alternative |
|----------|-------------|-------------|---------------------|
| Symbolic math | SymPy (default) + SageMath (optional) | Maxima / PARI/GP directly | SymPy has Python-native API. Maxima/PARI require separate installation and non-Python syntax. SageMath wraps both anyway for users who need them. |
| Numerical | NumPy + SciPy | Julia / MATLAB | Python is the lingua franca. Julia is faster but less installed. MATLAB requires license. Our agents already speak Python. |
| Arbitrary precision | mpmath (ships with SymPy) | flint / arb | mpmath is pure Python and sufficient. C libraries like flint are faster but require compilation and are overkill for verification tasks. |

### LLM Reasoning

| Category | Recommended | Alternative | Why Not Alternative |
|----------|-------------|-------------|---------------------|
| Reasoning engine | Claude (native) | GPT-4 / Gemini via API | We are inside Claude Code. Adding external LLMs adds complexity, latency, and cost without demonstrated benefit for math. |
| Orchestration | GSD agent pattern | LangChain / CrewAI | GSD already provides orchestrator-agent-document architecture. LangChain would be a dependency that duplicates existing functionality. |

---

## Installation

### Plugin Installation (for users)
```bash
# Install the Claude Code plugin
npx get-shit-done-math
```

### Python Prerequisites (user must install separately)
```bash
# Core computation stack (required)
pip install sympy numpy scipy matplotlib

# Optional: SageMath (for advanced algebra, number theory)
# Via conda (recommended for SageMath):
conda install -c conda-forge sage

# Optional: Wolfram Engine (free for personal use)
# Download from https://www.wolfram.com/engine/
```

### No npm production dependencies
The plugin has zero npm production dependencies, matching the GSD philosophy. All computation is delegated to the user's Python environment via the Bash tool.

---

## Architecture Implications for Stack

### How Agents Use the Stack

```
User asks math question
  |
  v
Orchestrator Agent (reads problem, routes to phases)
  |
  +--> Literature Agent
  |      Uses: WebFetch -> arXiv API, Semantic Scholar API
  |      Uses: WebSearch -> Google Scholar (fallback)
  |      Output: .planning/research/LITERATURE.md
  |
  +--> Proof Agent
  |      Uses: Claude reasoning (prompt-engineered)
  |      Uses: Bash -> python3 -c "from sympy import *; ..." (verification)
  |      Output: .planning/research/PROOF.md
  |
  +--> Computation Agent
  |      Uses: Bash -> python3 scripts in .planning/computation/
  |      Uses: Bash -> sage (if available)
  |      Uses: Bash -> wolframscript (if available)
  |      Output: .planning/computation/RESULTS.md + generated plots
  |
  +--> Writing Agent
       Uses: Claude LaTeX generation (prompt-engineered)
       Uses: Bash -> python3 -c "from sympy import latex; ..." (expr -> LaTeX)
       Output: output/{document}.tex
```

### Key Design Principle: Thin Glue, Fat Agents

The technology stack is intentionally thin. The "intelligence" lives in agent prompts, not in library code. This means:

1. **No Python wrapper scripts to maintain** -- agents write Python inline via Bash
2. **No API client libraries** -- agents call HTTP endpoints via WebFetch
3. **No LaTeX template engines** -- agents write LaTeX directly
4. **No orchestration framework** -- GSD is the orchestration framework

The only "new code" this project adds to GSD is:
- Math-specific agent prompts (markdown files)
- Math-specific workflow definitions (markdown files)
- Math-specific document templates (markdown files)
- Possibly a few utility functions in gsd-tools.js (for computation environment detection)

---

## Version Verification Needed

**IMPORTANT: The following versions are from training data (mid-2025 cutoff) and should be verified before finalizing the roadmap.**

| Package | Stated Version | Verify How | Priority |
|---------|---------------|------------|----------|
| SymPy | >= 1.13 | `pip install sympy && python -c "import sympy; print(sympy.__version__)"` | HIGH |
| NumPy | >= 1.26 | `pip install numpy && python -c "import numpy; print(numpy.__version__)"` | HIGH |
| SciPy | >= 1.13 | `pip install scipy && python -c "import scipy; print(scipy.__version__)"` | MEDIUM |
| SageMath | >= 10.3 | `sage --version` or check sagemath.org | LOW (optional) |
| mpmath | >= 1.3 | `pip install mpmath && python -c "import mpmath; print(mpmath.__version__)"` | LOW |
| Matplotlib | >= 3.9 | `pip install matplotlib && python -c "import matplotlib; print(matplotlib.__version__)"` | LOW |

---

## Sources

- arXiv API documentation: https://info.arxiv.org/help/api/index.html (HIGH confidence -- stable API, unlikely to have changed fundamentally)
- Semantic Scholar API: https://api.semanticscholar.org/api-docs/ (HIGH confidence -- well-documented, widely used)
- SymPy documentation: https://docs.sympy.org/ (HIGH confidence for capabilities, MEDIUM for exact version)
- SageMath: https://www.sagemath.org/ (MEDIUM confidence -- verify current version)
- Wolfram Engine: https://www.wolfram.com/engine/ (HIGH confidence -- stable product)
- GSD codebase analysis: `.planning/codebase/` (HIGH confidence -- read directly from source)

**Confidence note:** WebSearch and WebFetch tools were unavailable during this research session. All version numbers and API details are from training data (cutoff mid-2025). Core recommendations (which tools to use and why) are HIGH confidence because the mathematical computing ecosystem is stable. Specific version numbers are MEDIUM confidence and flagged for validation.

---

*Stack research: 2026-02-08*
