---
description: Switch between active math research problems
allowed-tools:
  - Read
  - Write
---

# /math:switch -- Switch Active Problem

Switch the active problem workspace to a different problem in the project.

## Process

### Step 1: Validate project

Check if `.math/config.json` exists.

- **If not:** Tell the user: "No math project found. Run `/math:init` first." Stop here.

### Step 2: Read config

Read `.math/config.json`. Extract `problems` array and `current_problem`.

### Step 3: Check problem count

- **If `problems` array is empty:** "No active problems. Run `/math:init` to create one." Stop here.
- **If only 1 problem:** "Only one active problem: {slug}. No switching needed." Stop here.
- **If 2+ problems:** Proceed to Step 4.

### Step 4: Display active problems

Show a numbered list of all active problems with current marker:

```
Active Problems:
  1. [*] neumann-series (analysis) -- PROOF_IN_PROGRESS -- last active: 2h ago
  2. [ ] spectral-gap (algebra) -- LITERATURE_SEARCH -- last active: 3d ago
  3. [ ] fixed-point (topology) -- INITIALIZED -- last active: 1w ago

[*] = current problem
```

For each problem entry, display:
- Index number
- `[*]` if it matches `current_problem`, `[ ]` otherwise
- Slug
- Domain (in parentheses)
- State (from the problem entry)
- Relative `last_active` time formatted as:
  - Under 1 minute: "just now"
  - Under 1 hour: "{N}min ago"
  - Under 24 hours: "{N}h ago"
  - Under 7 days: "{N}d ago"
  - Otherwise: "{N}w ago"

### Step 5: Wait for user selection

Ask the user to pick a problem:

> Select a problem (number or slug):

Accept either a number (1-indexed) or a slug string. Validate the selection exists in the `problems` array.

### Step 6: Switch active problem

Update `.math/config.json`:
- Set `current_problem` to the selected problem's slug
- Update the selected problem's `last_active` to current ISO timestamp
- Write `config.json` back

### Step 7: Confirm and auto-resume

Display confirmation:

```
Switched to: {slug} ({domain})
```

Then read the problem's key artifacts for a brief status summary:
1. Read `.math/problems/{slug}/PROBLEM.md` -- extract problem title or first line of problem statement
2. Read `.math/problems/{slug}/STATE.md` -- extract `current_state`
3. Read `.math/problems/{slug}/JOURNAL.md` -- extract `total_entries` from frontmatter

Display:

```
  State: {current_state}
  Journal entries: {total_entries}
```

If `current_state` is not `INITIALIZED` (i.e., work has been done):

```
  Run /math:status for full dashboard.
```

## References

- `.math/config.json` -- Project configuration with current_problem pointer and problems array
- `.math/problems/` -- Directory containing all active problem workspaces
- `.math/problems/{slug}/PROBLEM.md` -- Problem definition
- `.math/problems/{slug}/STATE.md` -- Research state
- `.math/problems/{slug}/JOURNAL.md` -- Research journal
