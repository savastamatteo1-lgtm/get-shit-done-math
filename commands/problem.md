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

## Process

### Step 1: Validate project exists

Check if `.math/` directory exists.

- **If `.math/` does not exist:** Tell the user: "No math project found. Run `/math:init` first to set up your project." Stop here.
- **If `.math/` exists:** Proceed to Step 2.

### Step 2: Check for existing problem

Read `.math/PROBLEM.md` frontmatter. If `status` is not `draft` (i.e., a problem has already been submitted):

- Inform the user: "A problem is already defined. Submitting a new problem will overwrite it."
- Ask: "Continue? (yes/no)"
- If no, abort.

### Step 3: Spawn intake agent

Use the Task tool to spawn the `agents/math-intake.md` agent. The agent handles all interaction with the user, collecting problem information step-by-step and writing the structured `.math/PROBLEM.md`.

Pass context to the agent:
- The `.math/` directory path
- Current notation profile domain (from `.math/config.json`)

The agent will:
1. Collect problem statement (with LaTeX confirmation)
2. Collect mathematical domain
3. Collect desired output type
4. Optionally collect known results
5. Check notation preferences
6. Optionally collect references
7. Write structured `.math/PROBLEM.md`
8. Update `.math/STATE.md`

## References

- `@agents/math-intake.md` -- Problem intake agent with guided wizard logic
- `@templates/PROBLEM.md` -- Problem document structure reference
