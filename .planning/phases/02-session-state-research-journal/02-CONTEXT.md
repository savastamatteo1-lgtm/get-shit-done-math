# Phase 2: Session State & Research Journal - Context

**Gathered:** 2026-02-08
**Status:** Ready for planning

<domain>
## Phase Boundary

Persistent research sessions and approach tracking across time. User can leave a research session, return days later, and resume with full context restored — including what approaches were tried and why they succeeded or failed. This phase delivers the state management and journaling infrastructure that all downstream agents will write to and read from.

</domain>

<decisions>
## Implementation Decisions

### Session Restoration
- Summary + on-demand loading: compact summary auto-loads on resume, detailed sections (full proof state, all literature) load only when user or agent needs them
- Resume summary is a **status dashboard** (structured overview): problem statement, current proof state, open gaps, last approach tried, key literature found
- **Provenance tracking**: each artifact section in the dashboard shows which agent created it and when (e.g., "Literature: 5 papers found by search agent")
- Dashboard ends with a **suggested next action** based on current state (e.g., "Based on current state, you might want to search for more literature on X"). User can follow or ignore

### Approach Journal Structure
- **Significant action level** granularity: new journal entry whenever something meaningful happens (literature found, computation run, proof step attempted, gap identified) — not just per proof strategy
- **Full context metadata** per entry: what was tried, outcome (succeeded/failed/partial), reasoning for abandoning or continuing, which agent ran it, artifacts produced, links to relevant files, timestamp
- **Categorized sections** in a single file: sections for proof attempts, literature searches, computations, etc. Each section chronological within itself
- **Required insight/takeaway field** on every entry, including successes: even successful approaches capture what was learned (e.g., "Contradiction approach fails because X is not bounded, suggesting we need a bound first")

### Dead-End Detection
- **Gentle nudge** intervention: system mentions prior similar approach in passing ("Note: a similar strategy was tried before — see journal entry #X"). Doesn't block, just informs
- **Allow re-attempts with acknowledgment**: show prior attempt info, ask "Proceed anyway?" to make it a conscious choice. Creates a journal entry noting the intentional re-attempt
- **Strategy-level matching**: match on proof strategy type (e.g., "induction on n" matches "induction on n", even with different details). Broad but catches most repeats
- **Highlight opportunities from changed conditions**: when conditions have changed since a failed attempt, proactively suggest retrying (e.g., "Previous induction attempt failed due to missing bound. You now have Lemma 2 providing that bound — worth retrying?")

### Session Lifecycle
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

</decisions>

<specifics>
## Specific Ideas

- Dashboard should feel like a project status page — structured, scannable, not a wall of text
- Journal insight field is always required because even successes teach something — this is core to the research journal philosophy
- The "highlight opportunities" feature for dead-end detection is key: the system should connect new developments to previously blocked paths, essentially acting as a research memory that notices when obstacles have been removed

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 02-session-state-research-journal*
*Context gathered: 2026-02-08*
