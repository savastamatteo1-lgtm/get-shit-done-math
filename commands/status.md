---
description: View the research state dashboard showing problem status, notation config, and next steps
allowed-tools:
  - Read
  - Glob
---

# /math:status -- Research State Dashboard

Display a summary of the current math research project state with contextual next steps.

## Process

### Step 1: Validate project exists

Check if `.math/` directory exists.

- **If `.math/` does not exist:** Tell the user: "No math project found. Run `/math:init` first to set up your project." Stop here.
- **If `.math/` exists:** Proceed to Step 2.

### Step 2: Read project state

Read the following files and extract frontmatter fields:

1. **`.math/STATE.md`:** Extract `current_state`, `problem_defined`, `notation_configured`, `session_count`
2. **`.math/PROBLEM.md`:** Extract `status`, `domain`, `type`, `output_type` (only if problem is defined)
3. **`.math/NOTATION.md`:** Extract `domain`, `based_on_preset`, `modified`
4. **`.math/config.json`:** Extract `domain`, `notation_preset`

### Step 3: Display dashboard

Format and display:

```
Math Research Status
====================

State:    {current_state}
Problem:  {if problem_defined: "Defined -- {domain}, {output_type}" else: "Not defined"}
Notation: {if notation_configured: "{domain} preset{if modified: ' (customized)'}" else: "Not configured"}

Project Files:
  .math/PROBLEM.md   {status}
  .math/NOTATION.md   {domain or "empty"}
  .math/STATE.md      {current_state}
  .math/config.json   {domain or "no domain"}
```

### Step 4: Show contextual next steps

Based on current state, show relevant suggestions:

**If no problem defined:**
```
Next Steps:
  -> Run /math:problem to submit a mathematical problem
  -> Run /math:notation to customize your notation profile
```

**If problem defined but early state:**
```
Next Steps:
  -> Problem captured. Literature search, proof collaboration,
     and computation agents coming in future phases.
  -> Run /math:notation to review or customize notation
  -> Run /math:help to see all available and upcoming commands
```

**If notation not configured:**
Include: `-> Run /math:notation to set up domain-specific notation`

### Step 5: Show recent history

Read the History table from `.math/STATE.md` and display the last 5 entries (if any exist):

```
Recent Activity:
  {timestamp} -- {event}: {details}
```

If no history entries, skip this section.

## References

- `.math/STATE.md` -- Research state with history log
- `.math/PROBLEM.md` -- Problem definition and metadata
- `.math/NOTATION.md` -- Notation profile and domain
- `.math/config.json` -- Project configuration
