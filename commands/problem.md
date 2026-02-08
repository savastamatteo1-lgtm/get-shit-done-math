---
description: Submit a structured mathematical problem via guided intake wizard
agent: math-intake
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Task
---

# /math:problem -- Submit a Mathematical Problem

Launch the guided problem intake wizard to collect and structure a mathematical problem.

## Path Resolution

1. Read `.math/config.json`
2. Get `current_problem` slug
3. Resolve problem directory as `.math/problems/{current_problem}/`
4. Pass resolved directory path to the math-intake agent

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

### Step 2: Check for existing problem

Read `.math/problems/{current_problem}/PROBLEM.md` frontmatter. If `status` is not `draft` (i.e., a problem has already been submitted):

- Inform the user: "A problem is already defined for '{current_problem}'. Submitting a new problem will overwrite it."
- Ask: "Continue? (yes/no)"
- If no, abort.

### Step 3: Spawn intake agent

Use the Task tool to spawn the `agents/math-intake.md` agent. The agent handles all interaction with the user, collecting problem information step-by-step and writing the structured PROBLEM.md.

Pass context to the agent:
- The problem directory path: `.math/problems/{current_problem}/`
- The problem slug: `{current_problem}`
- Current notation profile domain (from `.math/config.json`)

The agent will:
1. Collect problem statement (with LaTeX confirmation)
2. Collect mathematical domain
3. Collect desired output type
4. Optionally collect known results
5. Check notation preferences
6. Optionally collect references
7. Write structured `.math/problems/{current_problem}/PROBLEM.md`
8. Update `.math/problems/{current_problem}/STATE.md`
9. Update `.math/config.json` problem entry (title, state, last_active)

## References

- `@agents/math-intake.md` -- Problem intake agent with guided wizard logic
- `@templates/PROBLEM.md` -- Problem document structure reference
- `.math/config.json` -- Project configuration with current_problem pointer
