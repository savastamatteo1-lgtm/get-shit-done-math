---
description: Archive a completed math research problem
allowed-tools:
  - Read
  - Write
  - Bash
---

# /math:archive -- Archive a Completed Problem

Move a completed (or abandoned) problem out of the active workspace into the archive. All files are preserved and accessible but the problem no longer appears in active problem lists.

## Process

### Step 1: Validate project

Check if `.math/config.json` exists.

- **If not:** Tell the user: "No math project found. Run `/math:init` first." Stop here.

### Step 2: Determine which problem to archive

Read `.math/config.json`. Extract `problems` array and `current_problem`.

- **If user provided a slug argument:** Validate it exists in the `problems` array. If not found, list active problems and ask the user to pick.
- **If no argument provided:** Show the active problems list (same format as `/math:switch` Step 4) and ask which to archive:

```
Active Problems:
  1. [*] neumann-series (analysis) -- PROOF_IN_PROGRESS -- last active: 2h ago
  2. [ ] spectral-gap (algebra) -- LITERATURE_SEARCH -- last active: 3d ago

Which problem would you like to archive? (number or slug):
```

### Step 3: Confirm with user

Display confirmation prompt:

```
Archive '{problem_title}' ({slug})?

This moves it out of active view but keeps all files accessible in .math/archive/.
Type 'yes' to confirm:
```

If the user does not confirm, abort.

### Step 4: Execute archive

Perform the following operations:

1. **Create archive directory** if it doesn't exist: `.math/archive/`

2. **Move problem directory:** `.math/problems/{slug}/` to `.math/archive/{slug}/`
   - Use Bash: `mv .math/problems/{slug} .math/archive/{slug}`

3. **Update `.math/config.json`:**
   - Remove the problem entry from the `problems` array
   - Add to the `archived` array:
     ```json
     {
       "slug": "{slug}",
       "title": "{title from problem entry}",
       "domain": "{domain from problem entry}",
       "archived": "{current ISO timestamp}"
     }
     ```
   - If this was `current_problem`:
     - If other active problems remain: set `current_problem` to the first remaining problem's slug, update that problem's `last_active`
     - If no active problems remain: set `current_problem` to `""`

4. **Add final journal entry** to `.math/archive/{slug}/JOURNAL.md`:
   - Append under the appropriate section (General Notes):
     ```markdown
     ### Entry {N+1}: Problem archived

     **Date:** {ISO timestamp}
     **Category:** NOTE
     **Outcome:** SUCCEEDED

     **Insight/Takeaway:** Problem archived. Final state: {state from problem entry}. All artifacts preserved in .math/archive/{slug}/.
     ```
   - Update JOURNAL.md frontmatter: increment `total_entries`, update `last_entry`

### Step 5: Confirm

Display result:

```
Archived '{slug}'. {N} active problems remain.
```

If a new `current_problem` was set:
```
Current problem is now: {new_current_problem} ({domain})
```

If no active problems remain:
```
No active problems. Run /math:init to start a new one.
```

## References

- `.math/config.json` -- Project configuration with problems and archived arrays
- `.math/problems/` -- Active problem workspaces
- `.math/archive/` -- Archived problem workspaces (preserved but inactive)
