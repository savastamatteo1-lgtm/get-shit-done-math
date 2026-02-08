# Codebase Structure

**Analysis Date:** 2026-02-08

## Directory Layout

```
get-shit-done-math/
├── bin/                          # Installation and bootstrapping
│   └── install.js                # Multi-runtime installer (Claude, OpenCode, Gemini)
├── agents/                       # Specialized agent prompts (spawned by orchestrators)
│   ├── gsd-planner.md            # Creates executable phase plans
│   ├── gsd-executor.md           # Executes plans, writes code, commits
│   ├── gsd-verifier.md           # Verifies deliverables match phase goals
│   ├── gsd-debugger.md           # Systematic debugging with state
│   ├── gsd-phase-researcher.md   # Researches implementation approach
│   ├── gsd-project-researcher.md # Researches domain for new project
│   ├── gsd-roadmapper.md         # Creates phase roadmap from requirements
│   ├── gsd-plan-checker.md       # Validates plans achieve goals
│   ├── gsd-integration-checker.md # Validates integration patterns
│   ├── gsd-codebase-mapper.md    # Analyzes codebase (this agent)
│   └── gsd-research-synthesizer.md # Compiles research results
├── commands/                     # User-facing command specifications
│   └── gsd/                      # GSD command specs
│       ├── new-project.md        # Initialize project: questions → research → requirements → roadmap
│       ├── new-milestone.md      # Start next version cycle
│       ├── discuss-phase.md      # Capture implementation decisions
│       ├── plan-phase.md         # Create execution plan for phase
│       ├── execute-phase.md      # Run all plans for phase
│       ├── verify-work.md        # User acceptance testing
│       ├── complete-milestone.md # Archive milestone, tag release
│       ├── audit-milestone.md    # Verify milestone definition met
│       ├── quick.md              # Ad-hoc task (skip research/plan-check)
│       ├── add-phase.md          # Append phase to roadmap
│       ├── insert-phase.md       # Insert urgent phase between phases
│       ├── remove-phase.md       # Remove future phase
│       ├── plan-milestone-gaps.md # Create phases from verification failures
│       ├── pause-work.md         # Pause mid-phase (handoff note)
│       ├── resume-work.md        # Resume from pause
│       ├── add-todo.md           # Capture task for later
│       ├── check-todos.md        # List pending todos
│       ├── debug.md              # Systematic debugging
│       ├── list-phase-assumptions.md # See intended approach before planning
│       ├── set-profile.md        # Switch model profile (quality/balanced/budget)
│       ├── settings.md           # Configure workflow agents
│       ├── progress.md           # Where am I? What's next?
│       ├── update.md             # Update GSD with changelog
│       ├── help.md               # Show all commands
│       └── join-discord.md       # Join community
├── get-shit-done/                # GSD system files (copied to user's config)
│   ├── bin/
│   │   └── gsd-tools.js          # Shared CLI utilities (config, state, phase, model, git, slug, time)
│   ├── workflows/                # Process definitions (multi-step orchestrators)
│   │   ├── new-project.md        # Full project init
│   │   ├── new-milestone.md      # Next version cycle
│   │   ├── discuss-phase.md      # Capture user decisions
│   │   ├── plan-phase.md         # Research + plan + verify loop
│   │   ├── execute-phase.md      # Execute plans in parallel waves
│   │   ├── verify-work.md        # UAT workflow
│   │   ├── complete-milestone.md # Archive and release
│   │   ├── quick.md              # Ad-hoc task
│   │   └── map-codebase.md       # Parallel mapper agents
│   ├── templates/                # Document templates
│   │   ├── project.md            # Project vision template
│   │   ├── requirements.md       # v1/v2 requirements
│   │   ├── roadmap.md            # Phase roadmap
│   │   ├── context.md            # Phase-specific user decisions
│   │   ├── phase-prompt.md       # Phase intro/scope
│   │   ├── plan.md               # Executable plan template
│   │   ├── summary.md            # What was built
│   │   ├── uat.md                # User testing checklist
│   │   ├── debug.md              # Debugging session template
│   │   ├── config.json           # Default project config
│   │   └── debug-subagent-prompt.md # Debug runner template
│   └── references/               # Shared guidelines
│       └── ui-brand.md           # UI/design reference patterns
├── hooks/                        # Git/session hooks (executed by Claude Code)
│   ├── gsd-statusline.js         # Shows model, task, context usage in statusline
│   ├── gsd-check-update.js       # Checks for GSD updates at session start
│   └── dist/                     # Pre-bundled hooks (included in npm distribution)
├── scripts/                      # Build scripts
│   └── build-hooks.js            # Bundles hooks/ into hooks/dist/ for distribution
├── .planning/                    # Documentation about this system (planning docs)
│   └── codebase/                 # Auto-generated by gsd-codebase-mapper agent
│       ├── STACK.md              # Tech stack analysis
│       ├── INTEGRATIONS.md       # External services
│       ├── ARCHITECTURE.md       # System design (this doc)
│       ├── STRUCTURE.md          # Directory layout (this doc)
│       ├── CONVENTIONS.md        # Code patterns
│       ├── TESTING.md            # Test structure
│       └── CONCERNS.md           # Technical debt
├── assets/                       # Images for docs
├── README.md                     # User-facing documentation
├── CHANGELOG.md                  # Release notes
├── GSD-STYLE.md                  # Manifesto and philosophy
├── LICENSE                       # MIT license
└── package.json                  # npm package metadata
```

## Directory Purposes

**bin/**
- Purpose: Installation entry point
- Contains: `install.js` (multi-runtime installer, tool adaptation, permission config)
- Key files: `install.js` (1530 lines, handles Claude/OpenCode/Gemini installation)

**agents/**
- Purpose: Specialized agent instructions that Claude follows when spawned
- Contains: One .md file per agent role; each agent is a prompt with philosophy, discovery protocol, process steps
- Key files:
  - `gsd-planner.md` — Creates plans; enforces user decision fidelity
  - `gsd-executor.md` — Executes plans; writes code, tests, commits
  - `gsd-verifier.md` — Checks deliverables; diagnoses failures
  - `gsd-debugger.md` — Systematic debugging with persistent state

**commands/gsd/**
- Purpose: User-facing command definitions loaded into Claude Code as `/gsd:command`
- Contains: ~28 command specs; each has name, tools, objective, execution context
- Key files:
  - `plan-phase.md` — Main planning command
  - `execute-phase.md` — Main execution command
  - `new-project.md` — Full initialization

**get-shit-done/workflows/**
- Purpose: Multi-step orchestrator processes that spawn agents and manage gates
- Contains: One workflow per complex process (plan, execute, verify, etc.)
- Key files:
  - `plan-phase.md` — Research → Plan → Verify loop
  - `execute-phase.md` — Wave-based parallel execution
  - `map-codebase.md` — Parallel agent mapper

**get-shit-done/templates/**
- Purpose: Markdown/JSON templates that agents instantiate when creating planning docs
- Contains: Default document structure for all planning artifacts
- Key files:
  - `plan.md` — Task template (XML structure for executors)
  - `requirements.md` — v1/v2 structure
  - `roadmap.md` — Phase list structure
  - `config.json` — Default settings

**get-shit-done/bin/**
- Purpose: Shared utility CLI for workflows/agents
- Contains: `gsd-tools.js` (config load, state update, phase find, model resolve, slug gen, commit, etc.)
- Key files: `gsd-tools.js` (643 lines, NODE.js CLI with model profile table)

**hooks/**
- Purpose: Git and session hooks that run automatically in Claude Code
- Contains: Node.js hooks that integrate with Claude's hook system
- Key files:
  - `gsd-statusline.js` — Shows model, context usage in status bar
  - `gsd-check-update.js` — Checks for updates at session start
  - `dist/` — Pre-bundled versions included in npm package

**scripts/**
- Purpose: Build and development scripts
- Contains: Bundler for hooks
- Key files: `build-hooks.js` (pre-bundles hooks for distribution)

**.planning/codebase/**
- Purpose: Auto-generated codebase analysis documents
- Contains: Stack, integrations, architecture, structure, conventions, testing, concerns
- Generated by: `/gsd:map-codebase` orchestrator spawning 4 parallel mapper agents

## Key File Locations

**Entry Points:**
- `bin/install.js` — Installation entry; sets up ~/.claude/, ~/.config/opencode/, or ~/.gemini/
- `commands/gsd/*.md` — User command files; loaded as `/gsd:command` in Claude Code
- `get-shit-done/workflows/*.md` — Orchestrator workflows; referenced by commands via @path

**Configuration:**
- `.planning/config.json` — Project config (model_profile, workflow settings, parallelization, branching strategy)
- `get-shit-done/templates/config.json` — Default config template
- `package.json` — npm metadata, version, distribution files

**Core Logic:**
- `agents/gsd-planner.md` — Planning algorithm, task decomposition, user decision enforcement
- `agents/gsd-executor.md` — Code execution, testing, committing
- `agents/gsd-verifier.md` — Goal verification, failure diagnosis
- `get-shit-done/bin/gsd-tools.js` — Shared utilities (model resolution, phase lookup, git wrapper)

**Testing/Validation:**
- `agents/gsd-plan-checker.md` — Validates plans before execution
- `agents/gsd-integration-checker.md` — Validates integration patterns
- `get-shit-done/workflows/verify-work.md` — User acceptance testing

## Naming Conventions

**Files:**
- Agent files: `gsd-{agent-role}.md` (e.g., `gsd-planner.md`, `gsd-executor.md`)
- Command files: `{command-name}.md` (e.g., `plan-phase.md`, `execute-phase.md`)
- Workflow files: `{workflow-name}.md` (e.g., `plan-phase.md`, `execute-phase.md`)
- Template files: `{artifact}.md` or `.json` (e.g., `plan.md`, `summary.md`, `config.json`)
- Hook files: `gsd-{hook-name}.js` (e.g., `gsd-statusline.js`, `gsd-check-update.js`)

**Directories:**
- `.planning/` — All project planning artifacts (generated during workflow)
- `.planning/phases/{NN}/` — Phase-specific docs (NN = 01, 02, 03, etc.)
- `.planning/research/` — Research artifacts (created during new-project)
- `.planning/quick/` — Ad-hoc task artifacts (created during quick mode)
- `.planning/todos/pending/` — Uncompleted todos
- `.planning/todos/completed/` — Completed todos
- `get-shit-done/` — Core GSD system files (distributed to user's config dir)
- `agents/`, `commands/`, `hooks/` — Source files that get copied during install

**Phase Artifacts:**
- `01-CONTEXT.md` — User decisions for phase 01
- `01-RESEARCH.md` — Research findings
- `01-1-PLAN.md` — First plan for phase 01
- `01-2-PLAN.md` — Second plan for phase 01
- `01-1-SUMMARY.md` — Summary of first plan execution
- `01-VERIFICATION.md` — UAT results and any failures
- `01-UAT.md` — User acceptance test checklist

## Where to Add New Code

**New Workflow/Feature:**
- **New orchestrator process** → Create `get-shit-done/workflows/{name}.md`
  - Structure: Define steps, gates, agent spawning via Task tool
  - Reference in command via `@~/.claude/get-shit-done/workflows/{name}.md`

- **New agent** → Create `agents/gsd-{role}.md`
  - Structure: YAML frontmatter → role → philosophy → discovery → process
  - Spawn via Task tool with `subagent_type="gsd-{role}"`, `run_in_background=true`

- **New command** → Create `commands/gsd/{name}.md`
  - Structure: YAML frontmatter → objective → execution context → process
  - Process section delegates to workflow file via @path

- **New utility function** → Add to `get-shit-done/bin/gsd-tools.js`
  - Pattern: Named function `cmd{CamelCase}`, add to switch statement in main()
  - Called via `node ~/.claude/get-shit-done/bin/gsd-tools.js {command}`

**New Template Document:**
- **Template file** → Create in `get-shit-done/templates/{artifact}.md`
- **Usage:** Agents copy via `cat ~/.claude/get-shit-done/templates/{artifact}.md` and instantiate
- **Pattern:** Include placeholders like `[USER VISION]`, `[PHASE NAME]` for substitution

**Hook/Integration:**
- **Hook file** → Create in `hooks/{name}.js`
- **Build:** Run `npm run build:hooks` to bundle to `hooks/dist/`
- **Pattern:** Export a function or CLI interface matching Claude Code hook spec

**Testing Analysis Documents:**
- **Generated by mapper** → `gsd-codebase-mapper` spawned from `/gsd:map-codebase`
- **Location:** Written to `.planning/codebase/{DOCUMENT}.md`
- **Pattern:** Follow template structure in `<step name="write_documents">` of mapper agent

## Special Directories

**`get-shit-done/templates/`:**
- Purpose: Document templates copied/instantiated by agents
- Generated: No (source of truth)
- Committed: Yes

**`.planning/`:**
- Purpose: Project planning artifacts generated during workflow
- Generated: Yes (created by agents during /gsd:new-project, /gsd:plan-phase, etc.)
- Committed: Yes (if commit_docs=true in config)

**`hooks/dist/`:**
- Purpose: Pre-bundled hooks for distribution in npm package
- Generated: Yes (via `npm run build-hooks`)
- Committed: Yes (consumed at install time)

**`.planning/research/`:**
- Purpose: Research artifacts from domain investigation
- Generated: Yes (created by research agents)
- Committed: Yes (provides context for future phases)

**`.planning/phases/{NN}/`:**
- Purpose: All artifacts for one phase
- Generated: Yes (one directory per phase)
- Committed: Yes (phases need history for auditing)

---

*Structure analysis: 2026-02-08*
