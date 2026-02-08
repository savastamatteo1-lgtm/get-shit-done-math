---
phase: 02-session-state-research-journal
plan: 02
subsystem: commands
tags: [multi-problem, path-resolution, config-json, slug, workspace-switching, archiving, backward-compatibility]

# Dependency graph
requires:
  - phase: 01-foundation
    provides: "Phase 1 command pattern (YAML frontmatter + Process steps), math-intake agent, init/status/problem/notation commands"
  - phase: 02-01
    provides: "config.json v0.2.0 schema with problems array, current_problem pointer, archived array; JOURNAL.md template"
provides:
  - "Multi-problem /math:init with per-problem directory creation under .math/problems/{slug}/"
  - "/math:switch command for workspace switching between active problems"
  - "/math:archive command for problem completion and archiving"
  - "Path resolution pattern: all commands resolve files through config.json current_problem"
  - "Backward compatibility migration from Phase 1 flat layout to per-problem directories"
  - "Math-intake agent writes to per-problem directory and updates config.json problem entry"
affects: [02-03, phase-03, phase-04, phase-05]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Path resolution pattern: read config.json -> get current_problem -> resolve .math/problems/{current_problem}/FILENAME.md"
    - "Backward compatibility migration: detect Phase 1 flat layout, offer upgrade to multi-problem structure"
    - "Problem slug convention: lowercase, hyphen-separated, 3-4 keywords, unique within config.json problems array"
    - "Workspace switching: numbered list with [*] current marker and relative timestamps"

key-files:
  created:
    - "commands/switch.md"
    - "commands/archive.md"
  modified:
    - "commands/init.md"
    - "commands/status.md"
    - "commands/problem.md"
    - "commands/notation.md"
    - "agents/math-intake.md"

key-decisions:
  - "Init handles 3 cases: fresh project, Phase 1 migration, add problem to existing Phase 2 project"
  - "All commands use consistent path resolution pattern through config.json current_problem"
  - "Archive preserves all files in .math/archive/{slug}/ with final journal entry"
  - "Switch includes auto-resume: shows problem state and journal entry count after switching"

patterns-established:
  - "Path resolution block: documented at top of each command for consistent config.json -> current_problem -> directory resolution"
  - "Legacy detection: check for .math/PROBLEM.md at root to identify Phase 1 layout needing migration"
  - "Problem lifecycle: INITIALIZED -> PROBLEM_DEFINED -> ... -> archived (with directory move)"

# Metrics
duration: 4min
completed: 2026-02-08
---

# Phase 2 Plan 02: Multi-Problem Commands Summary

**Per-problem directory structure under .math/problems/{slug}/ with path resolution through config.json, workspace switching via /math:switch, and problem archiving via /math:archive**

## Performance

- **Duration:** 4 min
- **Started:** 2026-02-08T16:34:32Z
- **Completed:** 2026-02-08T16:38:53Z
- **Tasks:** 2
- **Files modified:** 7

## Accomplishments

- Rewrote /math:init to create per-problem directories with PROBLEM.md, NOTATION.md, STATE.md, JOURNAL.md under .math/problems/{slug}/, handling fresh projects, Phase 1 migrations, and adding problems to existing projects
- Updated all Phase 1 commands (status, problem, notation) and math-intake agent to resolve file paths through config.json current_problem pointer instead of hardcoded .math/ root paths
- Created /math:switch with numbered active problem list, [*] current marker, relative timestamps, and auto-resume summary after switching
- Created /math:archive with user confirmation, directory move to .math/archive/, config.json cleanup, current_problem reassignment, and final journal entry

## Task Commits

Each task was committed atomically:

1. **Task 1: Update /math:init and Phase 1 commands for path resolution** - `939b7f0` (feat)
2. **Task 2: Create /math:switch and /math:archive commands** - `7210415` (feat)

## Files Created/Modified

- `commands/init.md` - Rewritten for multi-problem support with 3 cases (fresh, migration, add new) and JOURNAL.md creation
- `commands/status.md` - Path resolution through config.json, shows active/archived problem counts, reads from per-problem directory
- `commands/problem.md` - Passes resolved problem directory path to math-intake agent via config.json
- `commands/notation.md` - Reads/writes .math/problems/{current_problem}/NOTATION.md instead of .math/NOTATION.md
- `agents/math-intake.md` - Writes to per-problem directory, updates config.json problem entry (title, state, last_active)
- `commands/switch.md` - New command: lists active problems with [*] marker, relative timestamps, switches current_problem
- `commands/archive.md` - New command: moves problem to .math/archive/, updates config.json arrays, adds final journal entry

## Decisions Made

- Init supports 3 distinct cases (fresh, migration, add-to-existing) rather than a single flow, keeping each path clear and explicit
- All commands share a documented Path Resolution block at the top for consistency and easy auditing
- Archive adds a final NOTE journal entry to preserve the archive event in the problem's history
- Switch shows an auto-resume summary (state + journal count) after switching to reduce context-switching friction

## Deviations from Plan

None -- plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None -- no external service configuration required.

## Next Phase Readiness

- All commands now operate on per-problem directories, ready for Phase 2 Plan 03 (session management agent)
- /math:switch and /math:archive enable multi-problem workflow for all future agents
- Journal protocol (from 02-01) and JOURNAL.md creation (from this plan) are ready for session management commands
- Path resolution pattern is established for any new commands in Phase 3+

---
*Phase: 02-session-state-research-journal*
*Completed: 2026-02-08*
