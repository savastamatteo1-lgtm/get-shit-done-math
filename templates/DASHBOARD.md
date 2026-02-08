---
generated: "{ISO_TIMESTAMP}"
problem: "{PROBLEM_SLUG}"
state: "{CURRENT_STATE}"
session_number: 0
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
