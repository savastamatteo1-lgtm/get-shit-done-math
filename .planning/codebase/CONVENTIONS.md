# Coding Conventions

**Analysis Date:** 2026-02-08

## Naming Patterns

**Files:**
- kebab-case for all files (install.js, build-hooks.js, gsd-executor.md)
- Markdown files use kebab-case (gsd-statusline.js, execute-phase.md)
- Workflow files: kebab-case matching their purpose (execute-phase.md, audit-milestone.md)
- Command files: kebab-case organized in subdirectories (commands/gsd/execute-phase.md)

**Functions:**
- camelCase for all function definitions (promptRuntime, expandTilde, getGlobalDir)
- No async prefix — async functions look like regular functions (copyWithPathReplacement, handleStatusline)
- Handlers use handleX pattern (handleStatusline, handleBranching)
- Getter functions use getX or resolveX pattern (getGlobalDir, getCommitAttribution, resolveModelProfile)

**Variables:**
- camelCase for regular variables (selectedRuntimes, settingsPath, gsdUpdateCheck)
- CAPS_UNDERSCORES for environment variables and constants (HOOKS_DIR, DIST_DIR, CLAUDE_CONFIG_DIR)
- Underscore prefix for private module state (e.g., attributionCache, but no strict enforcement)

**Types & XML Tags:**
- kebab-case for XML semantic tags (<execution_context>, <step name="load_project_state">)
- snake_case for XML attributes (priority="first", type="auto", gate="blocking")
- PascalCase for step names when referencing (LoadProjectState, ExecuteTasks)

## Code Style

**Formatting:**
- No enforced linter detected (no .eslintrc, .prettierrc, biome.json, tsconfig.json)
- Node.js style: semicolons present throughout (const fs = require('fs');)
- Line length: varies, no strict limit observed (long lines in install.js reach 180+ chars)
- String quotes: single quotes preferred in Node.js, but mixed in context
- Indentation: 2 spaces consistently

**Language & Tone (for .md files):**
- Imperative voice: "Execute tasks", "Read STATE.md", "Copy commands"
- NO temporal language: describe current state only (banned: "we changed", "previously")
- NO filler language: absent "Let me", "Just", "Simply", "I'd be happy to"
- NO sycophancy: absent "Great!", "Awesome!", "Excellent!"
- Technical precision required: "JWT auth with refresh rotation using jose" not "authentication implemented"
- Character preservation: Always maintain diacritics (TÂCHES, not TACHES)

**Linting:**
- No linter configured in package.json
- Manual code review expected
- Consistency maintained through explicit style guide (GSD-STYLE.md)

## Import Organization

**Pattern (Node.js/JavaScript):**
1. Built-in modules first (fs, path, os, readline)
2. Local relative imports (./path, ../path)
3. Comments clarify purpose inline

**Example from `bin/install.js`:**
```javascript
const fs = require('fs');
const path = require('path');
const os = require('os');
const readline = require('readline');

// Colors
const cyan = '\x1b[36m';
```

**No aliases observed:** Direct relative paths used throughout.

## Error Handling

**Pattern:**
- throw errors for blocking issues with context: `console.error(message); process.exit(1);`
- Try-catch blocks silently fail on non-critical operations (JSON parse, file read)
- Silent failures common: `catch (e) {}` used to prevent statusline/hook failures

**Example from `hooks/gsd-statusline.js`:**
```javascript
try {
  const todos = JSON.parse(fs.readFileSync(path.join(todosDir, files[0].name), 'utf8'));
  const inProgress = todos.find(t => t.status === 'in_progress');
  if (inProgress) task = inProgress.activeForm || '';
} catch (e) {}  // Silently fail on file system errors - don't break statusline
```

**Exit codes:** 1 for errors, 0 for success

**Validation:**
- Arguments validated before use: `if (!nextArg || nextArg.startsWith('-'))`
- Directory checks: `if (!fs.existsSync(srcDir))`
- JSON validation with try-catch

## Logging

**Framework:**
- console.log and console.error for output
- No structured logging library

**Patterns:**
- Colorized output using ANSI escape codes: `\x1b[36m` (cyan), `\x1b[32m` (green), `\x1b[33m` (yellow)
- Status symbols: `${green}✓${reset}` for success, `${yellow}⚠${reset}` for warning
- Messages prefixed with two spaces: `  ${green}✓${reset} Installed commands/gsd`
- progress bar format: `█` (filled), `░` (empty)

**Usage in install.js:**
```javascript
console.log(`  ${green}✓${reset} Installed ${count} commands to command/`);
console.error(`  ${yellow}Cannot specify both --global and --local${reset}`);
```

## Comments

**When to Comment:**
- Document WHY not WHAT: `// Silently fail - don't break statusline on parse errors`
- Explain non-obvious logic: `// Scale: 80% real usage = 100% displayed`
- Reference external context: `// OpenCode follows XDG Base Directory spec`

**Avoid:**
- Obvious comments on code: `// increment counter`
- Redundant descriptions

**TODO Comments:**
- Format: `// TODO: description` (simple, no username)
- Link to issue number when applicable: `// TODO: Fix race condition (issue #123)`
- Avoid TODOs in released code — prefer tracking via GitHub issues

## Function Design

**Size:**
- Functions 50-150 lines observed
- No strict limit, but break into helpers when logic is reusable
- `install()` is 210 lines because it orchestrates multiple concerns

**Parameters:**
- Functions vary: 0-6 parameters observed
- Use object parameters for >3 args: `getGlobalDir(runtime, explicitDir = null)`
- Destructuring used: `function createTestUser(overrides?: Partial<User>)`

**Return Values:**
- Explicit returns required
- void functions print to console/process.stdout
- Functions return objects with multiple values: `{ settingsPath, settings, statuslineCommand, runtime }`

## Module Design

**Exports (Node.js):**
- module.exports for main exports
- No ES6 modules detected (uses require/module.exports)
- Single responsibility per file: `build-hooks.js` only copies hooks

**Barrel Files:**
- index.ts for re-exporting not used in this codebase
- Flat structure preferred: one file = one responsibility

**File Organization:**
- `/bin` — CLI entry points
- `/commands/gsd` — Command definitions
- `/agents` — Agent definitions
- `/get-shit-done` — Workflow and template reference library
- `/hooks` — Hook scripts (statusline, update checks)
- `/scripts` — Build and maintenance scripts

## Commit Message Format

**Conventional Commits:**
```
{type}({scope}): {description}
```

**Types:**
- `feat` — New feature
- `fix` — Bug fix
- `docs` — Documentation only
- `refactor` — Code cleanup, no behavior change
- `chore` — Dependencies, config, maintenance
- `revert` — Undo previous commit

**Examples:**
```
feat(install): add Gemini CLI support
fix(install): use absolute paths on Windows (#207)
docs(readme): update installation instructions
refactor(orchestrator): extract context loading
chore: bump esbuild to 0.24.0
```

**Scope:** Feature area or component (install, executor, orchestrator)

**Rules:**
- One commit per task during execution
- Stage files individually (never `git add .`)
- Include `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>` for AI-generated code
- No commits for TODOs or incomplete work (use branches)

## XML/Markdown Conventions (Documentation)

**XML for semantic structure:**
- `<objective>` — What and why
- `<execution_context>` — Prerequisites and references
- `<process>` — Container for steps
- `<step name="step_name">` — Individual action
- `<task type="auto|checkpoint:human-verify|checkpoint:decision">` — Work unit
- `<if mode="condition">` — Conditional blocks

**Attributes:**
- `priority="first"` or `priority="second"` — Step ordering
- `type="auto"` — Autonomous execution
- `type="checkpoint:human-verify"` — Requires verification
- `type="checkpoint:decision"` — Requires user choice
- `gate="blocking"` — Must complete before continuing

**Step naming:**
- snake_case: `name="load_project_state"`, `name="execute_tasks"`

**@-References:**
- Static: `@~/.claude/get-shit-done/workflows/execute-phase.md`
- Conditional: `@.planning/DISCOVERY.md (if exists)`
- Signal lazy loading for context system

## Special Conventions

**Placeholder conventions:**
- Square brackets for template values: `[Project Name]`, `[Description]`
- Curly braces for dynamic values: `{phase}-{plan}-PLAN.md`, `{slug}`

**No enterprise patterns:**
- Banned: Story points, sprints, RACI matrices, change management
- Banned: Team coordination, human dev time estimates
- Banned: Vague language ("improve performance", "fix bugs")

**Diacritical characters:**
- ALWAYS preserve: "TÂCHES" not "TACHES"
- Apply to all output and documentation

---

*Convention analysis: 2026-02-08*
*Based on GSD-STYLE.md, bin/install.js, hooks, and workflow patterns*
