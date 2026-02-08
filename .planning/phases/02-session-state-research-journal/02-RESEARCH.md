# Phase 2: Session State & Research Journal - Research

**Researched:** 2026-02-08
**Domain:** Document-driven session persistence, approach journaling, dead-end detection, multi-problem workspace management
**Confidence:** HIGH

## Summary

Phase 2 builds the persistence and journaling infrastructure that lets a mathematician leave a research session and return days later with full context restored. The technical domain is entirely document architecture -- there are zero external library dependencies. Everything is markdown files with YAML frontmatter, read and written by Claude Code agents using the Read/Write tools established in Phase 1.

The core challenge is designing a document schema that serves two audiences simultaneously: (1) agents that need machine-parseable metadata for routing and dead-end detection, and (2) humans who need scannable, structured summaries for session restoration. Phase 1 established the pattern -- YAML frontmatter for machines, markdown body for humans -- and Phase 2 extends it to three new document types: a session dashboard (DASHBOARD.md), a research journal (JOURNAL.md), and a problem registry for multi-problem switching.

The hardest design problem is dead-end detection with strategy-level matching. The user decided on "strategy-level matching" (e.g., "induction on n" matches "induction on n" even with different details) with "gentle nudge" intervention. This requires a lightweight taxonomy of proof strategy types stored as structured tags in journal entries, combined with a matching algorithm that an agent can execute by scanning YAML frontmatter. The matching does not need to be algorithmic -- a Claude agent reading journal entries with strategy tags can identify matches using its own reasoning. The key is making the journal entries machine-scannable enough that the matching agent has the data it needs.

**Primary recommendation:** Extend the `.math/` directory with three new artifacts (DASHBOARD.md, JOURNAL.md, problem registry in config.json), a `/math:resume` command that generates the dashboard, a `/math:switch` command for problem switching, and a session-management agent that handles journal writing and dead-end detection. All document schemas follow the YAML-frontmatter + markdown-body pattern established in Phase 1.

<user_constraints>

## User Constraints (from CONTEXT.md)

### Locked Decisions

#### Session Restoration
- Summary + on-demand loading: compact summary auto-loads on resume, detailed sections (full proof state, all literature) load only when user or agent needs them
- Resume summary is a **status dashboard** (structured overview): problem statement, current proof state, open gaps, last approach tried, key literature found
- **Provenance tracking**: each artifact section in the dashboard shows which agent created it and when (e.g., "Literature: 5 papers found by search agent")
- Dashboard ends with a **suggested next action** based on current state (e.g., "Based on current state, you might want to search for more literature on X"). User can follow or ignore

#### Approach Journal Structure
- **Significant action level** granularity: new journal entry whenever something meaningful happens (literature found, computation run, proof step attempted, gap identified) -- not just per proof strategy
- **Full context metadata** per entry: what was tried, outcome (succeeded/failed/partial), reasoning for abandoning or continuing, which agent ran it, artifacts produced, links to relevant files, timestamp
- **Categorized sections** in a single file: sections for proof attempts, literature searches, computations, etc. Each section chronological within itself
- **Required insight/takeaway field** on every entry, including successes: even successful approaches capture what was learned (e.g., "Contradiction approach fails because X is not bounded, suggesting we need a bound first")

#### Dead-End Detection
- **Gentle nudge** intervention: system mentions prior similar approach in passing ("Note: a similar strategy was tried before -- see journal entry #X"). Doesn't block, just informs
- **Allow re-attempts with acknowledgment**: show prior attempt info, ask "Proceed anyway?" to make it a conscious choice. Creates a journal entry noting the intentional re-attempt
- **Strategy-level matching**: match on proof strategy type (e.g., "induction on n" matches "induction on n", even with different details). Broad but catches most repeats
- **Highlight opportunities from changed conditions**: when conditions have changed since a failed attempt, proactively suggest retrying (e.g., "Previous induction attempt failed due to missing bound. You now have Lemma 2 providing that bound -- worth retrying?")

#### Session Lifecycle
- **1:1 relationship** between sessions and problems: each problem has exactly one research session
- **Multiple problems can be active simultaneously**: each has its own directory tree, switch between them freely
- **Explicit archiving**: user marks a problem as complete, session artifacts move to an archive folder. Still accessible but out of active view
- **Slash command for switching**: `/math:switch` command lists active problems and lets you pick. Clean, explicit workspace switching

### Claude's Discretion
- Exact dashboard layout and formatting
- How on-demand section loading is triggered (by agent request vs explicit user command)
- Journal file naming and internal structure details
- Strategy-level matching implementation (tags, keywords, or pattern matching)
- Archive folder structure and naming

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope

</user_constraints>

## Standard Stack

### Core

| Library/Tool | Version | Purpose | Why Standard |
|-------------|---------|---------|-------------|
| Markdown + YAML frontmatter | -- | All session state documents (DASHBOARD.md, JOURNAL.md) | Established in Phase 1; document-as-interface paradigm for agent communication |
| Claude Code built-in tools | latest | Read, Write, Bash, Glob, Grep for agent operations | Native tools; agents use Read to load state, Write to persist state, Grep to scan journal entries |
| Claude Code Plugin System | latest | Command registration (/math:resume, /math:switch, /math:archive) | Established in Phase 1; auto-namespacing as /math:* |

### Supporting

| Library/Tool | Version | Purpose | When to Use |
|-------------|---------|---------|-------------|
| Node.js date utilities | built-in | ISO 8601 timestamps in journal entries | Every journal entry needs a timestamp; use `new Date().toISOString()` via Bash |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Single JOURNAL.md per problem | Multiple files per category (proof-journal.md, lit-journal.md) | Single file is simpler, matches user decision. Multiple files reduce merge conflicts but add complexity for agents scanning across categories. Use single file per user decision. |
| YAML frontmatter for strategy tags | Inline markdown tags (#induction, #contradiction) | YAML frontmatter is machine-parseable with grep; inline tags require parsing body text. YAML is more reliable for dead-end detection. Use YAML. |
| Directory-per-problem under .math/ | Flat files with problem ID prefix | Directory-per-problem is cleaner for multi-problem support, isolates all artifacts. Matches 1:1 session-problem relationship. Use directories. |

**Installation:**
```bash
# No new dependencies. Phase 2 is pure document architecture.
```

## Architecture Patterns

### Recommended Directory Structure

Phase 2 extends the `.math/` directory established in Phase 1:

```
.math/
├── config.json                    # Extended: problem registry, active problem pointer
├── active -> problems/problem-1/  # Symlink OR pointer in config.json to active problem
├── problems/                      # One directory per problem
│   ├── problem-1/                 # Slug derived from problem statement
│   │   ├── PROBLEM.md            # From Phase 1 (moved here from .math/ root)
│   │   ├── NOTATION.md           # From Phase 1 (moved here from .math/ root)
│   │   ├── STATE.md              # Extended: richer state machine
│   │   ├── DASHBOARD.md          # NEW: session restoration summary
│   │   ├── JOURNAL.md            # NEW: approach journal with categorized sections
│   │   ├── LITERATURE.md         # Future: Phase 3 writes here
│   │   ├── PROOF.md              # Future: Phase 4 writes here
│   │   └── COMPUTATION.md        # Future: Phase 5 writes here
│   └── problem-2/
│       ├── PROBLEM.md
│       ├── NOTATION.md
│       ├── STATE.md
│       ├── DASHBOARD.md
│       └── JOURNAL.md
├── archive/                       # Completed problems moved here
│   └── problem-old/
│       ├── PROBLEM.md
│       ├── JOURNAL.md
│       └── ...
└── templates/                     # Shared templates (optional, or use plugin templates/)
```

**Alternative (simpler, recommended):** Rather than symlinks (which can be fragile across platforms), use a `current_problem` field in `.math/config.json` that stores the active problem slug. Commands resolve the path as `.math/problems/{current_problem}/`. This is more portable and explicit.

```
.math/
├── config.json                    # { "current_problem": "neumann-series", "problems": [...] }
├── problems/
│   ├── neumann-series/
│   │   ├── PROBLEM.md
│   │   ├── NOTATION.md
│   │   ├── STATE.md
│   │   ├── DASHBOARD.md
│   │   └── JOURNAL.md
│   └── spectral-gap/
│       └── ...
└── archive/
    └── ...
```

### Migration from Phase 1 Layout

Phase 1 created files at `.math/` root (PROBLEM.md, NOTATION.md, STATE.md, config.json). Phase 2 introduces per-problem directories. The migration path:

1. `/math:init` is updated to create `problems/{slug}/` subdirectory
2. Existing `.math/PROBLEM.md`, `.math/NOTATION.md`, `.math/STATE.md` are moved into `problems/{slug}/` during the first `/math:resume` or `/math:problem` invocation
3. Backward compatibility: if files exist at `.math/` root, treat them as a single problem and auto-migrate

### Pattern 1: Session Dashboard as Generated Artifact

**What:** DASHBOARD.md is not manually maintained -- it is regenerated every time `/math:resume` runs. It reads the current state of all artifact files and produces a structured summary.

**When to use:** Every session restoration.

**Why:** Avoids stale dashboards. The dashboard is always current because it reads source-of-truth artifacts (PROBLEM.md, STATE.md, JOURNAL.md, LITERATURE.md, PROOF.md) each time.

**Schema:**

```markdown
---
generated: 2026-02-08T14:30:00Z
problem: neumann-series
state: PROOF_IN_PROGRESS
session_number: 4
---

# Research Dashboard: Neumann Series Convergence

## Problem
Let $T: X \to X$ be a bounded linear operator on a Banach space $X$...
(Truncated to first 3 lines of PROBLEM.md body)

## Current State: PROOF_IN_PROGRESS

### Proof Progress
- Strategy: Direct proof via Neumann series convergence
- Steps completed: 3 of 5
- Open gaps: 1 (uniform convergence of partial sums)
- Confidence: 2 [V], 1 [S], 0 [~]
- _Source: proof-agent, 2026-02-07T16:20:00Z_

### Literature Found
- Papers: 3 relevant (Rudin 1991, Conway 1990, Kato 1966)
- Key theorem: Banach contraction mapping (Rudin, Thm 3.3)
- _Source: literature-agent, 2026-02-06T10:15:00Z_

### Computations
- None yet
- _Source: (no computation agent runs)_

### Last Approach Tried
- **Approach:** Direct construction of partial sums $S_n = \sum_{k=0}^n T^k$
- **Outcome:** Partial -- showed convergence of $S_n$ but gap in showing limit is $(I-T)^{-1}$
- **Journal entry:** #PROOF-003

## Suggested Next Action

Based on current state: The proof has a gap at Step 4 (showing the limit equals the inverse). The journal shows the partial sums converge (PROOF-003). Consider verifying the identity $(I-T)S_n = I - T^{n+1}$ and taking limits. This is a direct calculation that could close the gap.

> Run `/math:prove` to continue proof development (available Phase 4)
```

**Key design points:**
- Frontmatter is minimal (generated timestamp, problem slug, state, session count)
- Each section has provenance: which agent produced the data, when
- Dashboard is scannable -- structured, not a wall of text
- Suggested next action is specific and actionable, not generic
- On-demand loading: the dashboard shows summaries; full artifacts load only when needed

### Pattern 2: Research Journal with Categorized Sections

**What:** JOURNAL.md is an append-only log of significant research actions, organized into categorized sections within a single file.

**When to use:** Every significant action by any agent.

**Schema:**

```markdown
---
problem: neumann-series
total_entries: 7
last_entry: 2026-02-08T14:30:00Z
strategies_tried:
  - id: PROOF-001
    strategy: contradiction
    status: abandoned
    tags: [contradiction, operator-norm, convergence]
  - id: PROOF-002
    strategy: direct-construction
    status: in-progress
    tags: [direct, neumann-series, partial-sums]
---

# Research Journal: Neumann Series Convergence

## Proof Attempts

### PROOF-001: Contradiction via non-invertibility
- **Timestamp:** 2026-02-06T10:00:00Z
- **Agent:** proof-agent
- **Strategy type:** contradiction
- **Tags:** [contradiction, operator-norm, convergence]
- **What was tried:** Assumed $I - T$ not invertible, attempted to derive contradiction with $\|T\| < 1$.
- **Outcome:** ABANDONED
- **Reasoning:** Contradiction approach requires showing kernel is trivial, but the norm bound $\|T\| < 1$ is more naturally suited to constructive argument. The approach was technically viable but unnecessarily indirect.
- **Artifacts produced:** None
- **Insight/Takeaway:** Direct construction is more natural when you have a norm bound. Contradiction works better when you have qualitative conditions (e.g., compactness) rather than quantitative bounds.

### PROOF-002: Direct construction via Neumann series
- **Timestamp:** 2026-02-07T09:00:00Z
- **Agent:** proof-agent
- **Strategy type:** direct-construction
- **Tags:** [direct, neumann-series, partial-sums, operator-norm]
- **What was tried:** Construct $S = \sum_{n=0}^{\infty} T^n$ directly, show convergence and verify $(I-T)S = S(I-T) = I$.
- **Outcome:** IN_PROGRESS
- **Reasoning:** Natural approach given the norm bound. Convergence of partial sums follows from geometric series bound.
- **Artifacts produced:** PROOF.md steps 1-3
- **Related files:** [PROOF.md](PROOF.md)
- **Insight/Takeaway:** The key identity $(I-T)S_n = I - T^{n+1}$ is the backbone. If $\|T\| < 1$, then $\|T^{n+1}\| \to 0$, which closes the argument. Completeness of the Banach space is needed for the limit to exist.

### PROOF-003: Gap identified in Step 4
- **Timestamp:** 2026-02-07T16:20:00Z
- **Agent:** proof-agent
- **Strategy type:** gap-identification
- **Tags:** [gap, limit-exchange, operator-norm]
- **What was tried:** Step 4 of direct proof: showing $\lim_{n \to \infty} S_n = (I-T)^{-1}$
- **Outcome:** PARTIAL
- **Reasoning:** Showed partial sums converge but need to verify the limit satisfies the inverse identity. Requires passing limit through the operator product.
- **Artifacts produced:** PROOF.md gap annotation at Step 4
- **Related files:** [PROOF.md](PROOF.md)
- **Insight/Takeaway:** Need to verify continuity of multiplication in operator algebra to pass the limit. This is standard but must be stated explicitly for rigor.

## Literature Searches

### LIT-001: Initial search for Neumann series results
- **Timestamp:** 2026-02-06T10:15:00Z
- **Agent:** literature-agent
- **Strategy type:** broad-survey
- **Tags:** [neumann-series, banach-space, operator-theory, bounded-operators]
- **What was tried:** arXiv and Semantic Scholar search for "Neumann series bounded operator Banach space"
- **Outcome:** SUCCEEDED
- **Reasoning:** Found standard references confirming the result and its proof strategy
- **Artifacts produced:** LITERATURE.md with 3 papers
- **Related files:** [LITERATURE.md](LITERATURE.md)
- **Insight/Takeaway:** This is a well-known textbook result (Rudin Ch. 10, Conway II.1). The proof strategy is standard. Main value of literature search was confirming there are no subtleties beyond the basic argument.

## Computations

(No computation entries yet)

## General Notes

(No general notes yet)
```

**Key design points:**
- YAML frontmatter includes a `strategies_tried` array -- this is the machine-scannable index that enables dead-end detection without parsing the full body
- Each entry has a unique ID with category prefix (PROOF-001, LIT-001, COMP-001, NOTE-001)
- Category sections: Proof Attempts, Literature Searches, Computations, General Notes
- Within each section, entries are chronological (newest at bottom)
- Every entry has the required insight/takeaway field
- Tags are structured for strategy matching (YAML arrays, not inline text)

### Pattern 3: Dead-End Detection via Strategy Tags

**What:** When an agent is about to attempt a proof strategy, it first scans JOURNAL.md frontmatter `strategies_tried` for entries with matching tags and strategy types. If a match is found, it surfaces the prior attempt.

**When to use:** Before every proof attempt.

**Implementation approach (Claude's discretion -- recommended):**

Strategy matching uses a combination of:
1. **Strategy type match:** Exact match on strategy type field (e.g., "contradiction" matches "contradiction")
2. **Tag overlap:** If 2+ tags match between the proposed approach and a prior entry, flag as potentially similar
3. **Agent judgment:** The matching agent reads the matched entry's full text and decides whether the similarity is meaningful

```
Dead-end detection flow:
1. Agent receives proof strategy request
2. Agent reads JOURNAL.md frontmatter strategies_tried array
3. For each prior entry:
   a. Check strategy type match (exact)
   b. Check tag overlap (>= 2 shared tags)
   c. If match: read full entry body
4. If prior attempt found:
   a. Check if conditions have changed (new literature, new lemmas)
   b. If conditions changed: "Previously tried X (PROOF-001), failed because Y.
      But you now have Z which addresses Y. Worth retrying?"
   c. If conditions unchanged: "Note: similar approach tried before (PROOF-001).
      Outcome: {outcome}. Insight: {insight}. Proceed anyway?"
5. If user proceeds: create new journal entry noting intentional re-attempt
```

**Why tags over free-text matching:** Tags are controlled vocabulary. "induction" always means induction, "contradiction" always means contradiction. Free-text matching would require fuzzy string comparison or embedding similarity, which is unnecessary when the agent can use simple tag intersection.

**Recommended strategy type taxonomy:**

| Strategy Type | Matches When | Example |
|--------------|-------------|---------|
| `direct` | Direct proof / construction | "Construct the inverse directly" |
| `contradiction` | Proof by contradiction | "Assume not invertible, derive contradiction" |
| `induction` | Any form of induction | "Induction on dimension n" |
| `contrapositive` | Proof by contrapositive | "Show if not B then not A" |
| `construction` | Explicit construction | "Construct a counterexample" |
| `estimation` | Bound/estimate argument | "Bound the operator norm" |
| `reduction` | Reduce to known result | "Reduce to finite-dimensional case" |
| `compactness` | Compactness argument | "Extract convergent subsequence" |
| `density` | Density/approximation argument | "Approximate by smooth functions" |
| `duality` | Duality argument | "Pass to dual space" |
| `computation` | Computational verification | "Compute for small cases" |
| `broad-survey` | Literature survey | "Search for related results" |
| `targeted-search` | Targeted literature search | "Find results on specific topic" |
| `gap-identification` | Identifying proof gaps | "Analyze where proof fails" |

### Pattern 4: Multi-Problem Workspace with Config Registry

**What:** `.math/config.json` is extended to maintain a registry of active problems and a pointer to the current one. `/math:switch` reads this registry.

**Schema extension:**

```json
{
  "version": "0.2.0",
  "current_problem": "neumann-series",
  "problems": [
    {
      "slug": "neumann-series",
      "domain": "analysis",
      "created": "2026-02-06T10:00:00Z",
      "last_active": "2026-02-08T14:30:00Z",
      "state": "PROOF_IN_PROGRESS",
      "title": "Neumann Series Convergence for Bounded Operators"
    },
    {
      "slug": "spectral-gap",
      "domain": "algebra",
      "created": "2026-02-05T09:00:00Z",
      "last_active": "2026-02-05T16:00:00Z",
      "state": "LITERATURE_SEARCH",
      "title": "Spectral Gap for Random Matrices"
    }
  ],
  "archived": [
    {
      "slug": "cauchy-schwarz-extension",
      "domain": "analysis",
      "archived": "2026-02-04T12:00:00Z",
      "title": "Extension of Cauchy-Schwarz to Operator Algebras"
    }
  ]
}
```

### Pattern 5: On-Demand Section Loading

**What:** The dashboard shows summaries; full artifact contents load only when needed.

**Recommended trigger (Claude's discretion):** Agent-request-based loading. When an agent needs full proof state, it reads PROOF.md directly via @-reference. When the user asks to see all literature, the agent reads LITERATURE.md. The dashboard never embeds full artifact contents -- it always summarizes.

**Implementation:** This is natural in the Claude Code agent model. Agents receive @-references to specific files. The resume command generates DASHBOARD.md by reading all artifacts and summarizing. Subsequent agent spawns load only the artifacts they need (e.g., proof-agent loads PROBLEM.md + LITERATURE.md + PROOF.md + JOURNAL.md, but NOT DASHBOARD.md).

### Pattern 6: Problem Archiving

**What:** User marks a problem as complete. Artifacts move from `problems/` to `archive/`.

**Recommended implementation (Claude's discretion):**

```
/math:archive flow:
1. Confirm with user: "Archive 'neumann-series'? This moves it out of active view."
2. Move .math/problems/neumann-series/ to .math/archive/neumann-series/
3. Update config.json: remove from problems array, add to archived array
4. If this was the current_problem, set current_problem to next active problem (or null if none)
5. Add final journal entry: "Problem archived. Status: {final_state}"
6. Confirm: "Archived. X active problems remain."
```

**Archive naming:** Use the same slug as the problem directory. The `archive/` directory mirrors `problems/` structure. Archived problems are read-only but fully accessible via direct path.

### Anti-Patterns to Avoid

- **Dashboard as source of truth:** DASHBOARD.md is a generated view, not a primary data store. Never write data to DASHBOARD.md that doesn't exist in another artifact. If the dashboard gets deleted, it can be regenerated from source artifacts with zero data loss.

- **Journal entries without strategy tags:** Every proof-related journal entry MUST have strategy type and tags in the YAML entry of `strategies_tried`. Without tags, dead-end detection cannot work. Make tags a required field in the journal entry protocol, not optional.

- **Coupling session state to Claude Code session:** The math session is a research session, not a Claude Code session. A user might open and close Claude Code 10 times during a single research session. The session boundary is problem-level, not tool-level. STATE.md tracks research state, not Claude Code process state.

- **Over-indexing on dashboard generation speed:** The dashboard is generated at resume time. It reads maybe 5-6 markdown files and produces a summary. This is trivially fast for an agent. Do not pre-compute or cache dashboard state.

- **Blocking on dead-end detection:** The user decided on "gentle nudge." Dead-end detection INFORMS but never BLOCKS. The agent mentions prior attempts and asks "Proceed anyway?" but always allows the user to continue. Never refuse to attempt a strategy because it was tried before.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Timestamp formatting | Custom date format functions | `new Date().toISOString()` via Bash | ISO 8601 is standard, sortable, unambiguous |
| Strategy matching algorithm | Custom embedding/similarity search | Tag intersection + agent judgment | Tags are controlled vocabulary; agent reads matched entries and uses natural language understanding to assess similarity. No algorithm needed. |
| Problem slug generation | Complex slugifier with collision handling | Lowercase, hyphenated, first 3-4 keywords from title | Slugs are human-readable directory names. Simple is better. Agent generates once at problem creation. |
| Session numbering | Auto-incrementing session counter | Count JOURNAL.md entries + dashboard `session_number` field | Session number is metadata for display, not a primary key. Count entries when needed. |
| File-based locking for concurrent access | Lock files, mutexes | Not needed -- Claude Code is single-threaded per session | Only one agent writes to a file at a time within a Claude Code session. No concurrent writes. |

**Key insight:** Phase 2 has zero algorithmic complexity. Every component is a document schema + an agent that reads/writes documents. The "intelligence" is in agent prompt engineering (knowing when to create journal entries, how to match strategies, what to put in the dashboard), not in code.

## Common Pitfalls

### Pitfall 1: Migration Breaking Phase 1 Commands

**What goes wrong:** Phase 1 commands (/math:problem, /math:status, /math:notation) expect files at `.math/PROBLEM.md` but Phase 2 moves them to `.math/problems/{slug}/PROBLEM.md`.
**Why it happens:** Phase 1 was designed for single-problem use. Phase 2 introduces multi-problem directories.
**How to avoid:** All Phase 1 commands must be updated to resolve the active problem path: read `config.json` for `current_problem`, construct path as `.math/problems/{current_problem}/`. Add a helper function or path-resolution pattern that all commands use. Backward compatibility: if files exist at `.math/` root (pre-migration), auto-migrate them into `problems/` on first access.
**Warning signs:** /math:status shows "No problem found" after Phase 2 deployment even though a problem was submitted in Phase 1.

### Pitfall 2: Journal File Growing Too Large for Agent Context

**What goes wrong:** After months of research on a complex problem, JOURNAL.md grows to thousands of lines. Agents loading the full journal consume too much of their 200k context window.
**Why it happens:** Append-only log grows unboundedly.
**How to avoid:** Two-tier loading. The JOURNAL.md frontmatter `strategies_tried` array is always compact (one entry per strategy, ~3 lines each). Agents that need dead-end detection load ONLY the frontmatter + the specific matched entry. The full journal body loads only when explicitly requested. Additionally, consider a "journal summary" approach: after every 20 entries, generate a JOURNAL-SUMMARY.md that captures key insights and strategy outcomes. Agents load the summary; detailed entries load on demand.
**Warning signs:** Agent context warnings or truncation during proof development on long-running problems.

### Pitfall 3: Dashboard Showing Stale Data

**What goes wrong:** User resumes, dashboard generates, then an agent runs and changes state. User re-reads dashboard and sees stale information.
**Why it happens:** Dashboard is a point-in-time snapshot, not a live view.
**How to avoid:** Dashboard is ALWAYS regenerated on resume, never cached. If the user wants current status mid-session, `/math:status` reads source artifacts directly (same as Phase 1 behavior). The dashboard is a session-start artifact, not a live monitor. Document this clearly: "Dashboard reflects state at session start. For current state, use /math:status."
**Warning signs:** User confused about why dashboard disagrees with current proof state.

### Pitfall 4: Dead-End Detection False Positives

**What goes wrong:** Agent flags "similar strategy tried before" when the current approach is actually meaningfully different. User gets fatigued by false nudges and starts ignoring them.
**Why it happens:** Tag overlap is too broad. Two entries sharing "operator-norm" and "convergence" tags might be completely different strategies.
**How to avoid:** The primary match criterion is strategy TYPE, not tags. Tags are secondary confirmation. A "contradiction" entry matches another "contradiction" entry, not a "direct" entry that happens to share tags. Additionally, the agent reads the matched entry's full text before nudging -- it makes a judgment call about whether the similarity is meaningful, not just mechanical tag matching.
**Warning signs:** User repeatedly saying "no that's different" when asked to acknowledge prior attempts.

### Pitfall 5: Problem Slug Collisions

**What goes wrong:** Two problems generate the same slug (e.g., both about "operator theory" get slug "operator-theory").
**Why it happens:** Simple slug generation from domain keywords.
**How to avoid:** Slugs derive from the problem TITLE (first 3-4 distinctive keywords), not the domain. If collision occurs, append a number suffix (operator-theory-2). Check `config.json` problems array before creating. The intake agent generates the slug at problem creation time and checks for uniqueness.
**Warning signs:** Directory already exists when creating new problem.

### Pitfall 6: Archiving Loses Cross-Problem References

**What goes wrong:** Problem A's journal references a technique from Problem B. Problem B gets archived. The reference path breaks.
**Why it happens:** Journal entries use relative links to files that move during archival.
**How to avoid:** Journal entries reference problems by slug, not by path. When reading a reference to another problem, resolve against both `problems/` and `archive/`. Archive is read-only but not invisible. Alternatively, journal entries use absolute-from-.math/ paths: `.math/problems/neumann-series/PROOF.md` and `.math/archive/cauchy-schwarz/PROOF.md`.
**Warning signs:** "File not found" errors when reading journal entries that reference archived problems.

## Code Examples

### config.json Extended Schema

```json
// Phase 2 extension of Phase 1 config.json template
{
  "version": "0.2.0",
  "current_problem": "neumann-series",
  "problems": [
    {
      "slug": "neumann-series",
      "title": "Neumann Series Convergence for Bounded Operators",
      "domain": "analysis",
      "notation_preset": "analysis",
      "created": "2026-02-06T10:00:00Z",
      "last_active": "2026-02-08T14:30:00Z",
      "state": "PROOF_IN_PROGRESS"
    }
  ],
  "archived": [],
  "plugin_version": "0.2.0"
}
```

### DASHBOARD.md Template

```markdown
---
generated: "{ISO_TIMESTAMP}"
problem: "{PROBLEM_SLUG}"
state: "{CURRENT_STATE}"
session_number: {N}
---

# Research Dashboard: {PROBLEM_TITLE}

## Problem

{First 3-5 lines of PROBLEM.md body, truncated}

## Current State: {CURRENT_STATE}

### Proof Progress
- Strategy: {current strategy or "No proof attempted yet"}
- Steps completed: {N of M or "N/A"}
- Open gaps: {count or "none"}
- Confidence: {V count} [V], {S count} [S], {~ count} [~]
- _Source: {agent}, {timestamp}_

### Literature Found
- Papers: {count} relevant ({brief list})
- Key theorem: {most important theorem found}
- _Source: {agent}, {timestamp}_

### Computations
- {summary or "None yet"}
- _Source: {agent}, {timestamp}_

### Last Approach Tried
- **Approach:** {description from latest JOURNAL.md proof entry}
- **Outcome:** {SUCCEEDED/FAILED/PARTIAL/IN_PROGRESS}
- **Journal entry:** #{entry_id}

## Suggested Next Action

{Context-specific suggestion based on current state and journal history.
Examples:
- If stuck on proof gap: suggest specific technique based on available literature
- If no literature searched: suggest /math:search
- If computation needed: suggest /math:compute
- If proof complete: suggest /math:write for LaTeX output
- If nothing done yet: suggest starting with /math:search}
```

### JOURNAL.md Template

```markdown
---
problem: "{PROBLEM_SLUG}"
total_entries: 0
last_entry: ""
strategies_tried: []
---

# Research Journal: {PROBLEM_TITLE}

## Proof Attempts

(No entries yet)

## Literature Searches

(No entries yet)

## Computations

(No entries yet)

## General Notes

(No entries yet)
```

### Journal Entry Template (for agents writing entries)

```markdown
### {CATEGORY}-{NNN}: {Title}
- **Timestamp:** {ISO_TIMESTAMP}
- **Agent:** {agent-name}
- **Strategy type:** {strategy_type from taxonomy}
- **Tags:** [{tag1}, {tag2}, {tag3}]
- **What was tried:** {description of what was attempted}
- **Outcome:** {SUCCEEDED | FAILED | PARTIAL | ABANDONED | IN_PROGRESS}
- **Reasoning:** {why this outcome occurred, why abandoned or continuing}
- **Artifacts produced:** {list of files created/modified, or "None"}
- **Related files:** {links to relevant artifacts}
- **Insight/Takeaway:** {REQUIRED -- what was learned, even from success}
```

### /math:resume Command Skeleton

```markdown
---
description: Resume a math research session with full context restoration
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
---

# /math:resume -- Resume Research Session

## Process

### Step 1: Validate project
Check .math/ exists and config.json is readable.

### Step 2: Resolve active problem
Read config.json, get current_problem. Resolve path: .math/problems/{current_problem}/

### Step 3: Generate dashboard
Read all artifacts in the problem directory:
- PROBLEM.md (problem statement + frontmatter)
- STATE.md (current research state)
- JOURNAL.md (recent entries + strategies_tried from frontmatter)
- LITERATURE.md (if exists -- paper count, key theorem)
- PROOF.md (if exists -- step count, gaps, confidence)
- COMPUTATION.md (if exists -- computation count)

Write DASHBOARD.md from template, filling all sections from artifact data.

### Step 4: Display dashboard
Show the generated DASHBOARD.md content to the user.

### Step 5: Suggest next action
Based on state, show contextual suggestions (from dashboard's suggested next action).
```

### /math:switch Command Skeleton

```markdown
---
description: Switch between active math research problems
allowed-tools:
  - Read
  - Write
---

# /math:switch -- Switch Active Problem

## Process

### Step 1: Read config.json
Load the problems array and current_problem.

### Step 2: Display active problems
Show numbered list:
  1. [*] neumann-series (analysis) -- PROOF_IN_PROGRESS -- last active: 2h ago
  2. [ ] spectral-gap (algebra) -- LITERATURE_SEARCH -- last active: 3d ago

[*] marks the current problem.

### Step 3: User selects
Wait for user to pick a number or slug.

### Step 4: Switch
Update config.json: set current_problem to selected slug.
Update selected problem's last_active timestamp.

### Step 5: Auto-resume
After switching, run the resume flow (generate dashboard, display status).
```

### Dead-End Detection Integration (Agent Protocol)

```markdown
## Dead-End Detection Protocol

Before attempting a new proof strategy, the proof agent MUST:

1. Read JOURNAL.md frontmatter `strategies_tried` array
2. Compare proposed strategy type and tags against prior entries
3. If match found (same strategy type OR 2+ shared tags with any FAILED/ABANDONED entry):
   a. Read the matched entry's full body text
   b. Assess whether the current attempt is meaningfully different
   c. Check if conditions have changed since the prior attempt:
      - New literature found? (check LITERATURE.md dates vs prior attempt date)
      - New lemmas proved? (check PROOF.md for new completed steps)
      - New computations? (check COMPUTATION.md for new results)
   d. If conditions changed:
      > "Previous attempt '{entry_title}' ({entry_id}) used {strategy_type}
      > and was {outcome} because: {reasoning}.
      > However, since then you have: {new_developments}.
      > This may address the original blocker. Worth retrying?"
   e. If conditions unchanged:
      > "Note: A similar approach was tried before -- see {entry_id}.
      > Strategy: {strategy_type}. Outcome: {outcome}.
      > Insight: {insight}.
      > Proceed with the new attempt anyway?"
4. If user proceeds: create journal entry noting intentional re-attempt with reference to prior entry
5. If no match: proceed normally, create journal entry for the new attempt
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| GSD `.continue-here.md` for session pause | Math-specific DASHBOARD.md regenerated from source artifacts | Phase 2 (new) | Dashboard is always fresh; no stale handoff files. Artifacts ARE the state, dashboard is a view. |
| GSD STATE.md as flat text | Math STATE.md with YAML frontmatter state machine + history | Phase 1 established, Phase 2 extends | Machine-parseable state enables orchestrator routing without content understanding |
| No journal -- restart from scratch | Append-only JOURNAL.md with categorized sections | Phase 2 (new) | Research memory survives across sessions; dead-end detection possible |
| Single-problem workspace | Multi-problem directory tree with config.json registry | Phase 2 (new) | Mathematicians often work on multiple problems; clean separation |

## Open Questions

1. **How does /math:init change for multi-problem support?**
   - What we know: Phase 1 /math:init creates files at .math/ root. Phase 2 needs per-problem directories.
   - What's unclear: Should /math:init create the .math/ root structure AND the first problem? Or should problem creation be exclusively through /math:problem?
   - Recommendation: /math:init creates the .math/ root structure (config.json with empty problems array, problems/ and archive/ directories). /math:problem creates problem directories. This separates workspace setup from problem intake. First /math:problem call after /math:init creates the first `problems/{slug}/` directory.

2. **When should journal entries be written by Phase 2 vs future phases?**
   - What we know: Phase 2 builds the journal infrastructure. The agents that write journal entries (proof-agent, literature-agent, computation-agent) are built in Phases 3-5.
   - What's unclear: What journal entries can be written in Phase 2 alone?
   - Recommendation: Phase 2 writes the journal protocol and templates. It also writes journal entries for: problem definition (when /math:problem runs), notation changes (when /math:notation modifies profile), and manual notes (a new /math:note command for user to add free-form journal entries). Phases 3-5 agents are instructed to write journal entries per the protocol.

3. **Should the journal protocol be a separate file like confidence-tiers.md?**
   - What we know: Phase 1 created `protocols/confidence-tiers.md` as a shared reference loaded by all agents.
   - What's unclear: Whether the journal-writing protocol needs the same treatment.
   - Recommendation: Yes. Create `protocols/journal-protocol.md` that defines: entry format, required fields, strategy type taxonomy, how to update frontmatter strategies_tried, and how to perform dead-end detection. All future agents load this via @-reference alongside confidence-tiers.md.

## Sources

### Primary (HIGH confidence)
- Phase 1 deliverables (commands/, agents/, templates/, protocols/) -- direct codebase inspection of established patterns
- Phase 1 RESEARCH.md -- plugin architecture, document schema patterns, YAML frontmatter conventions
- GSD pause/resume workflows (get-shit-done/workflows/pause-work.md, resume-project.md) -- session restoration patterns
- GSD ARCHITECTURE.md -- orchestrator-agent-document paradigm, state management patterns
- Phase 2 CONTEXT.md -- locked user decisions constraining all design choices

### Secondary (MEDIUM confidence)
- GSD STATE.md and CONVENTIONS.md -- state management patterns and naming conventions
- Architecture research (`.planning/research/ARCHITECTURE.md`) -- artifact schemas (LITERATURE.md, PROOF.md, COMPUTATION.md) that journal entries will reference
- Mathematical domain knowledge -- strategy type taxonomy derived from standard proof techniques

### Tertiary (LOW confidence)
- Journal file size projections -- estimated based on typical research session length; actual growth rates depend on user behavior and problem complexity. Flag for monitoring after deployment.
- Strategy matching effectiveness -- tag-based matching with agent judgment is a reasonable approach, but real-world false positive/negative rates are unknown until tested. Flag for tuning.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- zero new dependencies, pure document architecture extending Phase 1 patterns
- Architecture: HIGH -- directory structure, document schemas, and command patterns follow established Phase 1 conventions
- Journal design: HIGH -- structure directly implements locked user decisions with clear format
- Dead-end detection: MEDIUM -- tag-based matching with agent judgment is sound in principle; effectiveness depends on tag taxonomy quality and agent prompt engineering. May need tuning.
- Multi-problem workspace: HIGH -- straightforward directory isolation with config.json registry
- Migration from Phase 1: MEDIUM -- backward compatibility path is clear but must be carefully implemented to avoid breaking existing commands

**Research date:** 2026-02-08
**Valid until:** 90 days (document architecture patterns are stable; no external dependencies to go stale)
