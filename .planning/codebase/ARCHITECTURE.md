# Architecture

**Analysis Date:** 2026-02-08

## Pattern Overview

**Overall:** Orchestrator-agent workflow system with specialized agents spawned per task, state-driven pipeline orchestration, and document-as-interface paradigm.

**Key Characteristics:**
- **Multi-agent orchestration** — Thin orchestrators spawn specialized agents; agents write outputs, orchestrators integrate results
- **Document-driven state** — Project state stored in markdown files (.planning/) that travel between sessions and agents
- **Fresh context per task** — Each agent spawn receives clean 200k context; no accumulated garbage
- **Atomic commit workflow** — Each completed task commits separately with traceable history
- **Workflow-as-code** — Complex processes (research, planning, execution) defined as executable markdown workflows

## Layers

**Orchestration Layer:**
- Purpose: Parse user input, validate state, spawn agents with background execution (run_in_background=true), collect results, route to next step
- Location: `get-shit-done/workflows/` (workflow definitions) + CLI commands in `commands/gsd/`
- Contains: Workflow step definitions with process gates, validation logic, decision trees
- Depends on: `gsd-tools.js` utilities for config/phase/model resolution, git operations
- Used by: User-facing CLI commands trigger orchestrators

**Agent Layer:**
- Purpose: Perform specialized work (planning, execution, verification, research) within fresh context windows
- Location: `agents/gsd-*.md` (agent prompts with role, tools, philosophy, process)
- Contains: Structured agent instructions with role definition, philosophical guidelines, discovery protocols, step-by-step processes
- Depends on: Tools (Read, Write, Bash, Grep, Glob, Task, WebFetch), access to project state files
- Used by: Spawned via Task tool from orchestrators; agents read project state, write results to .planning/

**Command Layer:**
- Purpose: User-facing interfaces for workflow initiation
- Location: `commands/gsd/*.md` (command specs with execution context, process instructions)
- Contains: Command metadata (name, tools, arguments), objective statement, execution instructions that delegate to workflows
- Depends on: Workflows (referenced via @paths)
- Used by: Claude Code slash commands (loaded into ~/.claude/commands/gsd/)

**Utility Layer:**
- Purpose: Shared operations across workflows/agents (config loading, git, phase lookup, model resolution, state updates)
- Location: `get-shit-done/bin/gsd-tools.js`
- Contains: CLI commands for config, state, phase, model, slug, timestamp, todos, path verification, commit, summary verification
- Depends on: Node.js fs, path, execSync for git
- Used by: Workflows reference via bash `node ~/.claude/get-shit-done/bin/gsd-tools.js <command>`

**Reference/Template Layer:**
- Purpose: Shared patterns, UI guidelines, example structures
- Location: `get-shit-done/references/` (UI brand), `get-shit-done/templates/` (document templates)
- Contains: Project template defaults (requirements.md, roadmap.md, context.md, plan.md, etc.), UI guidelines
- Depends on: None
- Used by: Agents copy/instantiate templates when creating new planning documents

**Installation/Distribution Layer:**
- Purpose: Install GSD system to user's Claude Code, OpenCode, or Gemini config directories
- Location: `bin/install.js`, `hooks/` (pre-built in hooks/dist/)
- Contains: Multi-runtime installer (claude, opencode, gemini), tool adaptation layer, permission configurers
- Depends on: fs, path, readline for interactive setup
- Used by: `npx get-shit-done-cc` installs system to ~/.claude/ or ~/.config/opencode/ or ~/.gemini/

## Data Flow

**Typical Phase Execution Flow:**

1. **User initiates command** → `/gsd:plan-phase 1`
2. **Command loads** → `commands/gsd/plan-phase.md` executes, references workflow
3. **Orchestrator starts** → `get-shit-done/workflows/plan-phase.md` begins step execution
4. **State loaded** → `node gsd-tools.js state load --raw` reads config + STATE.md
5. **Research spawned** (if enabled) → Task tool spawns `gsd-phase-researcher` with fresh context, `run_in_background=true`
6. **Planning spawned** → Task tool spawns `gsd-planner` with research results loaded via @file
7. **Plan verification** (if enabled) → Task tool spawns `gsd-plan-checker` with plan + requirements
8. **Iteration loop** → If checks fail, planner revises, returns to checker
9. **Plans written** → Agent writes `01-1-PLAN.md`, `01-2-PLAN.md` to `.planning/phases/01/`
10. **Orchestrator commits** → `node gsd-tools.js commit "docs: plan phase 1"` with specific file list
11. **Execution** → Next orchestrator (`execute-phase`) reads plans, spawns parallel executors per task
12. **Execution waves** → Each executor runs in isolation with fresh context, commits per task
13. **Verification** → Verifier agent checks codebase against goals, writes UAT.md
14. **User testing** → `/gsd:verify-work 1` prompts manual testing, captures pass/fail

**State Management:**

- **Project state** stored in:
  - `.planning/config.json` — Model profile, workflow settings, parallelization config
  - `.planning/STATE.md` — Current position, blockers, decisions, notes
  - `.planning/ROADMAP.md` — Phases with status, requirements traceability
  - `.planning/phases/{NN}/*` — Phase context (CONTEXT.md), research (RESEARCH.md), plans (PLAN.md), summaries (SUMMARY.md), UAT (UAT.md)

- **State flows:** Agents read existing state via @file references, write new state to .planning/, orchestrator commits changes
- **Session continuity:** State files enable resumption via `/gsd:resume-work` after pausing mid-phase
- **Artifact traceability:** Each phase numbered sequentially (01, 02, etc.); plans numbered within phase (1-PLAN.md, 2-PLAN.md)

## Key Abstractions

**Workflow:**
- Purpose: Define a multi-step process with gates, iteration, and routing
- Examples: `get-shit-done/workflows/plan-phase.md`, `execute-phase.md`, `new-project.md`
- Pattern: XML structure with named steps (`<step name="...">`), sequential or conditional routing, subprocess calls (Task tool for agents)

**Agent:**
- Purpose: Specialized prompt+instruction set designed to solve one domain problem in a fresh context
- Examples: `agents/gsd-planner.md`, `gsd-executor.md`, `gsd-debugger.md`
- Pattern: YAML frontmatter (name, tools, color) → `<role>` section → `<philosophy>` section → `<process>` section with numbered steps

**Command:**
- Purpose: User-facing entry point that delegates to a workflow
- Examples: `commands/gsd/plan-phase.md`, `execute-phase.md`
- Pattern: YAML frontmatter (name, argument-hint, agent, allowed-tools) → `<objective>` → `<context>` → `<process>` (references workflow)

**Tool Wrapper (gsd-tools.js):**
- Purpose: Centralized utility for repeated patterns
- Examples: `resolve-model` (lookup model for agent + profile), `find-phase` (locate phase directory), `commit` (stage files, commit with message)
- Pattern: Command-driven CLI that outputs JSON or raw values; `--raw` flag for script usage

## Entry Points

**Installation Entry:**
- Location: `bin/install.js`
- Triggers: `npx get-shit-done-cc [args]`
- Responsibilities: Parse installation args (runtime, location), copy files to config dir, configure permissions, register hooks

**User Command Entry:**
- Location: `commands/gsd/*.md` (loaded into Claude Code as `/gsd:*` commands)
- Triggers: User types `/gsd:command` or `/gsd:command args`
- Responsibilities: Validate arguments, load execution context (@file references), delegate to orchestrator workflow

**Workflow Entry:**
- Location: `get-shit-done/workflows/*.md` (referenced by commands)
- Triggers: Command's `<process>` section points to workflow
- Responsibilities: Validate state, spawn agents with Task tool, collect outputs, iterate or route to next step

**Agent Entry:**
- Location: `agents/gsd-*.md` (spawned by orchestrator via Task tool)
- Triggers: Orchestrator executes `Task(subagent_type="gsd-*", run_in_background=true)`
- Responsibilities: Read project state (@file), perform work, write outputs to .planning/, return confirmation

## Error Handling

**Strategy:** Layered validation with orchestrator gates; agents assume valid input and focus on work; errors bubble up to orchestrator for decision (retry, route, abort).

**Patterns:**

1. **Orchestrator Validation** — State checks before spawning agents
   - Example: `verify-path-exists .planning/ROADMAP.md` before planning phase
   - Action: Fail with clear error message if precondition violated

2. **Agent Internal Validation** — Data quality checks before processing
   - Example: Planner verifies CONTEXT.md exists before creating tasks
   - Action: Document validation failure, return error to orchestrator

3. **Executor Verification** — Self-checks after task completion
   - Example: Executor writes code, commits, then verifies commit hash exists
   - Action: If verification fails, revert commit and return failure

4. **Orchestrator Recovery** — Iteration loops and fallbacks
   - Example: Plan checker fails plans → Planner revises → Re-check (loop max 3x)
   - Action: If max iterations exceeded, present to user with recommendations

5. **Git Safety** — All destructive ops gated by config
   - Example: Commit only if `commit_docs=true` and `.planning/` not gitignored
   - Action: Skip commit with reason if gate fails

## Cross-Cutting Concerns

**Logging:**
- Approach: Structured output to stdout (JSON or formatted text)
- Agents log work via SUMMARY.md file (written as artifact)
- Orchestrators log steps via console.log during process execution
- User sees progress in real-time as agents write files and return confirmations

**Validation:**
- Approach: Three-layer validation (orchestrator precondition → agent input → agent output)
- Orchestrators validate state before spawning agents
- Agents validate input files (CONTEXT.md, ROADMAP.md, etc.) before processing
- Executors verify commits exist in git history before declaring success

**Authentication:**
- Approach: Tool-level permissions configured in Claude Code settings
- GSD uses Read, Write, Bash, Glob, Grep, Task, WebFetch — user grants at install time
- Sensitive file protection via deny list in .claude/settings.json

**Context Management:**
- Approach: Agents receive only essential context via @file references; orchestrator doesn't load document contents
- Each agent has fresh 200k token budget
- Plans designed to execute in ~50% context (leave headroom for debugging/iteration)
- Large files split across multiple agents to avoid overload

**Git Integration:**
- Approach: `gsd-tools.js commit` wrapper centralizes logic
- Checks commit_docs config, verifies .planning/ not gitignored, stages files, commits
- Each task gets atomic commit with trace number (e.g., `feat(08-02): implement login endpoint`)
- Enables bisect to find breaking tasks, independent revert of failed work

---

*Architecture analysis: 2026-02-08*
