---
description: Initialize a math research project with multi-problem support, notation presets, and document templates
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---

# /math:init -- Initialize Math Research Project

Create or upgrade the `.math/` working directory with multi-problem directory structure, notation profile, problem templates, state tracker, research journal, and configuration.

## Path Resolution

All file operations in this command resolve paths through the multi-problem directory structure:

1. Read `.math/config.json`
2. Get `current_problem` slug
3. Resolve paths as `.math/problems/{current_problem}/FILENAME.md`
4. If `config.json` has no `current_problem` OR `.math/config.json` doesn't exist but `.math/PROBLEM.md` does (Phase 1 layout):
   - Auto-migrate: create `problems/{slug}/` directory, move PROBLEM.md, NOTATION.md, STATE.md into it, update config.json

## Process

### Step 1: Check for existing project

Check if `.math/` directory already exists and determine the layout.

- **If `.math/` does not exist:** Proceed to Step 2 (fresh initialization).
- **If `.math/` exists with Phase 1 flat layout** (`.math/PROBLEM.md` exists at root, no `problems/` directory): Inform the user: "Detected Phase 1 project layout. This will be upgraded to multi-problem format." Proceed to Step 2.
- **If `.math/` exists with Phase 2 layout** (`.math/config.json` exists with `problems` array): Display current problems list and config summary. Ask: "A math research project already exists. Add a new problem or reinitialize? (new/reinitialize/cancel)". If "new", proceed to Step 2 (adding a problem). If "reinitialize", warn about overwriting and proceed. If "cancel", abort.

### Step 2: Select mathematical domain

Ask the user to choose a primary mathematical domain. Present this list:

1. `algebra` -- Groups, rings, fields, modules
2. `analysis` -- Real/complex/functional analysis
3. `topology` -- Point-set and algebraic topology
4. `number-theory` -- Analytic and algebraic number theory
5. `combinatorics` -- Enumerative, extremal, algebraic combinatorics
6. `algebraic-geometry` -- Schemes, sheaves, cohomology
7. `differential-geometry` -- Manifolds, connections, curvature
8. `probability` -- Measure-theoretic probability, stochastic processes
9. `logic` -- Model theory, set theory, computability
10. `custom` -- Start with minimal default profile

Wait for the user to choose before proceeding.

### Step 3: Create directory structure and populate files

#### Case A: Fresh project (`.math/` does not exist)

1. Create `.math/` directory
2. Create `.math/config.json` from `templates/config.json` (version 0.2.0, empty `problems` array, empty `archived` array, no `current_problem`)
3. Create `.math/problems/` directory
4. Create `.math/archive/` directory
5. Generate problem slug: ask the user for a short title (e.g., "Neumann series convergence"). Slugify: lowercase, replace spaces with hyphens, keep first 3-4 keywords, check uniqueness against `config.json` `problems` array. Fallback: `{domain}-problem-1`.
6. Create `.math/problems/{slug}/` directory
7. Copy templates into the problem directory:
   - `.math/problems/{slug}/PROBLEM.md` from `templates/PROBLEM.md`
   - `.math/problems/{slug}/NOTATION.md` from `templates/NOTATION.md` (merged with domain preset, same as Phase 1 Step 3 item 3)
   - `.math/problems/{slug}/STATE.md` from `templates/STATE.md` (set `current_state: INITIALIZED`, `notation_configured: true/false`, add History entry)
   - `.math/problems/{slug}/JOURNAL.md` from `templates/JOURNAL.md` (set `problem` frontmatter field to slug)
8. Update `.math/config.json`:
   - Set `current_problem` to slug
   - Add problem entry to `problems` array:
     ```json
     {
       "slug": "{slug}",
       "title": "{user-provided title}",
       "domain": "{domain}",
       "notation_preset": "{domain}",
       "created": "{ISO timestamp}",
       "last_active": "{ISO timestamp}",
       "state": "INITIALIZED"
     }
     ```

#### Case B: Phase 1 layout migration (`.math/` exists with flat layout)

1. Read existing `.math/config.json` to get `domain` and `notation_preset`
2. Read `.math/PROBLEM.md` frontmatter to derive a slug from the problem title. Fallback: `{domain}-problem` if no title found.
3. Create `.math/problems/` and `.math/archive/` directories
4. Create `.math/problems/{slug}/` directory
5. Move `.math/PROBLEM.md` to `.math/problems/{slug}/PROBLEM.md`
6. Move `.math/NOTATION.md` to `.math/problems/{slug}/NOTATION.md`
7. Move `.math/STATE.md` to `.math/problems/{slug}/STATE.md`
8. Create `.math/problems/{slug}/JOURNAL.md` from `templates/JOURNAL.md` (set `problem` field to slug)
9. Rewrite `.math/config.json` with Phase 2 schema:
   - Set `version` to `"0.2.0"`
   - Set `current_problem` to slug
   - Create `problems` array with one entry (slug, title from PROBLEM.md, domain, notation_preset, created from original config, last_active now, state from STATE.md)
   - Create empty `archived` array

#### Case C: Adding a new problem to existing Phase 2 project

1. Generate problem slug (same as Case A Step 5)
2. Create `.math/problems/{slug}/` directory
3. Copy templates into the problem directory (same as Case A Step 7)
4. Update `.math/config.json`:
   - Set `current_problem` to the new slug
   - Add new problem entry to `problems` array
   - Update previous problem's `last_active` timestamp

### Step 4: Display summary

Show the user:

```
Math research project initialized in .math/

  Problem: {slug} ({domain})
  Location: .math/problems/{slug}/
  Files: PROBLEM.md, NOTATION.md, STATE.md, JOURNAL.md

Next: Submit a problem with /math:problem
```

If this was a migration (Case B), also show:

```
  Migrated from Phase 1 layout: files moved to .math/problems/{slug}/
```

If this was adding to an existing project (Case C), also show:

```
  Switched to new problem. {N} total active problems.
  Use /math:switch to navigate between problems.
```

## Error Handling

- If `presets/{domain}.md` is not found for a valid domain name, warn the user and fall back to the custom (empty) profile.
- If `.math/` directory creation fails, report the error and stop.
- If migration (Case B) fails mid-way, report which files were moved and which weren't.

## References

- `@presets/{domain}.md` -- Domain-specific notation conventions
- `@templates/PROBLEM.md` -- Problem statement template
- `@templates/NOTATION.md` -- Notation profile template
- `@templates/STATE.md` -- Research state template
- `@templates/JOURNAL.md` -- Research journal template
- `@templates/config.json` -- Configuration template (v0.2.0 multi-problem schema)
