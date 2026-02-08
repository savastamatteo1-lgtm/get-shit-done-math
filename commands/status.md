---
description: View the research state dashboard showing problem status, notation config, and next steps
allowed-tools:
  - Read
  - Glob
---

# /math:status -- Research State Dashboard

Display a summary of the current math research project state with contextual next steps.

## Path Resolution

1. Read `.math/config.json`
2. Get `current_problem` slug
3. Resolve all file paths as `.math/problems/{current_problem}/FILENAME.md`
4. If `config.json` has no `current_problem` OR `.math/PROBLEM.md` exists at root (Phase 1 layout):
   - Inform user: "Detected legacy layout. Run `/math:init` to upgrade to multi-problem format."

## Process

### Step 1: Validate project exists

Check if `.math/` directory exists.

- **If `.math/` does not exist:** Tell the user: "No math project found. Run `/math:init` first to set up your project." Stop here.
- **If `.math/` exists:** Proceed to Step 2.

### Step 2: Resolve current problem

Read `.math/config.json` and extract `current_problem`.

- **If `current_problem` is empty or missing:** Check if `.math/PROBLEM.md` exists at root (Phase 1 legacy layout). If yes, inform user: "Detected legacy layout. Run `/math:init` to upgrade to multi-problem format." Stop here. If no legacy files either, tell user: "No active problem. Run `/math:init` or `/math:switch` to set one." Stop here.
- **If `current_problem` is set:** Resolve the problem directory as `.math/problems/{current_problem}/`. Proceed to Step 3.

### Step 3: Read project state

Read the following files from the resolved problem directory and extract frontmatter fields:

1. **`.math/problems/{current_problem}/STATE.md`:** Extract `current_state`, `problem_defined`, `notation_configured`, `session_count`
2. **`.math/problems/{current_problem}/PROBLEM.md`:** Extract `status`, `domain`, `type`, `output_type` (only if problem is defined)
3. **`.math/problems/{current_problem}/NOTATION.md`:** Extract `domain`, `based_on_preset`, `modified`
4. **`.math/config.json`:** Extract `current_problem`, `problems` array length, `archived` array length
5. **`.math/problems/{current_problem}/LITERATURE.md`:** Extract `total_papers`, `confirmed_count`, `unconfirmed_count`, `last_search`, `sources_queried` (if file exists)

### Step 4: Display dashboard

Format and display:

```
Math Research Status
====================

Problem:  {current_problem} ({domain})
Location: .math/problems/{current_problem}/
State:    {current_state}
Defined:  {if problem_defined: "Yes -- {domain}, {output_type}" else: "Not defined"}
Notation: {if notation_configured: "{domain} preset{if modified: ' (customized)'}" else: "Not configured"}

Active problems: {problems array length}  |  Archived: {archived array length}

Problem Files:
  .math/problems/{current_problem}/PROBLEM.md      {status}
  .math/problems/{current_problem}/NOTATION.md      {domain or "empty"}
  .math/problems/{current_problem}/STATE.md         {current_state}
  .math/problems/{current_problem}/JOURNAL.md       {total_entries} entries
  .math/problems/{current_problem}/LITERATURE.md    {confirmed_count} confirmed, {unconfirmed_count} unconfirmed
```

If LITERATURE.md exists, also display:

```
Literature:
  Papers found: {total_papers} ({confirmed_count} confirmed, {unconfirmed_count} unconfirmed)
  Last search: {last_search}
  Sources: {sources_queried joined with ", "}
```

If LITERATURE.md does not exist:

```
Literature: No searches yet. Run /math:search to find relevant papers.
```

### Step 5: Show contextual next steps

Based on current state, show relevant suggestions:

**If no problem defined:**
```
Next Steps:
  -> Run /math:problem to submit a mathematical problem
  -> Run /math:notation to customize your notation profile
  -> Run /math:switch to change active problem
```

**If problem defined:**
```
Next Steps:
  -> Run /math:search to find relevant mathematical literature
  -> Run /math:notation to review or customize notation
  -> Run /math:switch to work on a different problem
  -> Run /math:help to see all available and upcoming commands
```

**If notation not configured:**
Include: `-> Run /math:notation to set up domain-specific notation`

### Step 6: Show recent history

Read the History table from `.math/problems/{current_problem}/STATE.md` and display the last 5 entries (if any exist):

```
Recent Activity:
  {timestamp} -- {event}: {details}
```

If no history entries, skip this section.

## References

- `.math/config.json` -- Project configuration with current_problem pointer
- `.math/problems/{current_problem}/STATE.md` -- Research state with history log
- `.math/problems/{current_problem}/PROBLEM.md` -- Problem definition and metadata
- `.math/problems/{current_problem}/NOTATION.md` -- Notation profile and domain
- `.math/problems/{current_problem}/JOURNAL.md` -- Research journal
- `.math/problems/{current_problem}/LITERATURE.md` -- Literature search results and references
