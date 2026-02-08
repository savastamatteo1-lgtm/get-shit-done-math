---
description: Search arXiv and Semantic Scholar for mathematical literature relevant to your problem
agent: literature-search
allowed-tools:
  - Read
  - Write
  - WebFetch
  - Bash
  - Glob
  - Task
---

# /math:search -- Search Mathematical Literature

Search arXiv and Semantic Scholar for mathematical papers relevant to your active research problem. Results are verified against source APIs, synthesized with connections to your problem, and written to LITERATURE.md.

## Usage

- `/math:search` -- Run a broad literature search based on your problem definition
- `/math:search "Neumann series convergence"` -- Search with a specific query focus
- `/math:search "spectral radius bounded operator"` -- Target a specific topic within your domain

## Path Resolution

1. Read `.math/config.json`
2. Get `current_problem` slug
3. Resolve problem directory as `.math/problems/{current_problem}/`
4. Pass resolved directory path to the literature-search agent

## Process

### Step 0: Resolve current problem

Read `.math/config.json` and extract `current_problem`.

- **If `current_problem` is empty or missing:** Check if `.math/PROBLEM.md` exists at root (Phase 1 legacy layout). If yes, inform user: "Detected legacy layout. Run `/math:init` to upgrade to multi-problem format." Stop here. If no legacy files, tell user: "No active problem. Run `/math:init` first." Stop here.
- **If `current_problem` is set:** Resolve the problem directory as `.math/problems/{current_problem}/`. Proceed to Step 1.

### Step 1: Validate project exists

Check if `.math/` directory and `.math/problems/{current_problem}/` directory exist.

- **If `.math/` does not exist:** Tell the user: "No math project found. Run `/math:init` first to set up your project." Stop here.
- **If problem directory does not exist:** Tell the user: "Problem directory not found for '{current_problem}'. Run `/math:init` or `/math:switch`." Stop here.
- **If both exist:** Proceed to Step 2.

### Step 2: Validate problem is defined

Read `.math/problems/{current_problem}/PROBLEM.md` frontmatter.

- **If PROBLEM.md does not exist:** Tell the user: "No problem defined for '{current_problem}'. Run `/math:problem` first to submit your research problem." Stop here.
- **If `status` is `draft`:** Tell the user: "Problem is still in draft for '{current_problem}'. Complete `/math:problem` first before searching literature." Stop here.
- **If `status` is `defined` or any other non-draft value:** Proceed to Step 3.

### Step 3: Spawn literature search agent

Use the Task tool to spawn the `agents/literature-search.md` agent. The agent handles the full search-verify-synthesize workflow and writes results to LITERATURE.md.

Pass context to the agent:
- The problem directory path: `.math/problems/{current_problem}/`
- The problem slug: `{current_problem}`
- The user's search query (if any argument was provided to `/math:search`)
- Protocol references: `@protocols/literature-protocol.md`, `@protocols/confidence-tiers.md`, `@protocols/journal-protocol.md`

The agent will:
1. Load problem context from PROBLEM.md and NOTATION.md
2. Construct and display search queries for user review
3. Search arXiv and Semantic Scholar (Phase A: Discovery)
4. Verify all candidate papers against source APIs (Phase B: Verification)
5. Synthesize connections between confirmed papers and the user's problem (Phase C: Synthesis)
6. Write results to `.math/problems/{current_problem}/LITERATURE.md`
7. Log a journal entry via the session-manager agent
8. Report a summary to the user with top papers and key findings

## References

- `@agents/literature-search.md` -- Literature search agent with full search-verify-synthesize workflow
- `@protocols/literature-protocol.md` -- API endpoints, schemas, anti-hallucination rules
- `@protocols/confidence-tiers.md` -- [V]/[S]/[~] confidence markers
- `.math/config.json` -- Project configuration with current_problem pointer
