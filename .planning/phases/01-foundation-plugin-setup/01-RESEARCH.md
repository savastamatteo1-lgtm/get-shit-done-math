# Phase 1: Foundation & Plugin Setup - Research

**Researched:** 2026-02-08
**Domain:** Claude Code plugin architecture, document-driven math research workflows, structured problem intake, notation systems, confidence protocols
**Confidence:** HIGH

## Summary

Phase 1 delivers the foundational architecture that all downstream agents depend on: plugin installation, slash command surface, problem intake wizard, notation profile system, and the confidence tier protocol. This is a Claude Code plugin -- the "stack" is markdown files (commands, agents, skills), a thin Node.js installer, and structured markdown document schemas. There are no external library dependencies for this phase.

The key technical insight is that Claude Code now has a formal plugin system with `.claude-plugin/plugin.json` manifests, namespaced commands, and auto-discovery of components. The math plugin should use this system rather than the legacy GSD npx installer approach. Commands placed in `commands/` get auto-namespaced as `/plugin-name:command-name`. The plugin name in `plugin.json` controls the namespace prefix -- setting it to `"math"` gives us `/math:problem`, `/math:notation`, etc., exactly matching the user's decision.

The core technical challenges are: (1) designing the PROBLEM.md schema with YAML frontmatter that downstream agents can parse for routing, (2) creating notation profile presets that cover major mathematical domains without overwhelming the user, (3) defining the confidence tier protocol as a shared markdown convention that all future agents will follow, and (4) structuring the slash commands so Phase 1 commands work while future commands show "coming soon" messages. All of these are design and prompt-engineering problems, not library problems.

**Primary recommendation:** Build as a Claude Code plugin using the modern `.claude-plugin/plugin.json` pattern with `commands/` for slash commands and `agents/` for the problem-intake agent. Use `skills/` for the notation profile system. All document schemas (PROBLEM.md, NOTATION.md, confidence tiers) are pure markdown with YAML frontmatter -- no code, no dependencies.

<user_constraints>

## User Constraints (from CONTEXT.md)

### Locked Decisions

#### Problem Intake Flow
- Guided wizard: step-by-step prompts collecting one field at a time
- Required fields: problem statement, math domain, desired output type
- Optional fields: known results, notation preferences, references
- Problem statement accepts natural language with optional inline LaTeX -- system converts to full LaTeX internally and shows it back to the user for confirmation
- Math domain selection: preset list of common domains (algebra, analysis, topology, number theory, combinatorics, etc.) with option to type a custom domain
- Output stored as structured PROBLEM.md that downstream agents can consume

#### Notation Profile Design
- Domain-based presets: system offers notation presets by math domain (e.g., "algebraic geometry" preset includes standard conventions for sheaves, varieties, etc.) -- user can tweak after selection
- Per-project scope: each research session/project has its own notation profile stored in the project directory
- Full style guide coverage: profiles include symbol conventions (fraktur ideals, blackboard bold sets), preferred LaTeX packages (amsmath, mathtools, etc.), AND presentation style (theorem numbering, proof structure, formatting conventions)
- Notation conflict handling: flag + translate -- system translates all output to match the user's profile, but flags where the original source (e.g., cited paper) used different conventions

#### Confidence Tier Presentation
- Visual markers: subtle color/icon markers next to each step -- visible but not disruptive to reading flow
- Source-based tier criteria:
  - VERIFIED = cited from published literature (known theorem, published result)
  - SUGGESTED = derived by the system with explicit justification from verified premises
  - SPECULATIVE = system's conjecture, heuristic reasoning, creative leap, or analogy
- Manual override: user can promote or demote any tier level -- system notes it was manually set
- Active warnings: system flags clear warnings when presenting proofs with speculative steps (e.g., "This proof contains 2 speculative steps that need verification")

#### Slash Command Surface
- Namespaced convention: all commands under /math:* (e.g., /math:problem, /math:notation, /math:search, /math:compute)
- Full surface on install: all planned commands visible from Phase 1 -- unavailable ones display "Available after Phase X" message
- Phase 1 active commands: /math:problem (submit problem), /math:notation (set notation profile), /math:help (command listing), /math:status (research state dashboard), /math:init (project setup)
- Future commands shown but disabled: /math:prove, /math:search, /math:compute, /math:write, /math:orchestrate
- Installation: npm install -g, then /math:init in Claude Code to set up project directory

### Claude's Discretion
- Exact list of domain presets for notation profiles
- Specific icon/color choices for confidence tier markers
- PROBLEM.md internal schema and field naming
- Help command formatting and layout
- Status dashboard information density

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope

</user_constraints>

## Standard Stack

### Core

| Library/Tool | Version | Purpose | Why Standard |
|-------------|---------|---------|-------------|
| Claude Code Plugin System | latest | Plugin distribution, command namespacing, auto-discovery | Official plugin system with `.claude-plugin/plugin.json` manifest; provides automatic namespacing (`/math:*`), marketplace distribution, and component auto-discovery |
| Node.js | >= 16.7.0 | Plugin installer script (bin/install.js) | Inherited from GSD; the installer bootstraps the plugin directory structure |
| Markdown + YAML frontmatter | -- | All commands, agents, skills, document schemas | Core GSD paradigm; document-as-interface for agent communication |
| Claude Code built-in tools | latest | Read, Write, Bash, Glob, Grep for agent operations | Native tools available to all agents in Claude Code |

### Supporting

| Library/Tool | Version | Purpose | When to Use |
|-------------|---------|---------|-------------|
| esbuild | ^0.24.0 | Hook bundling (dev only) | Only if Phase 1 includes hooks (likely not needed) |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Plugin system (.claude-plugin/) | Legacy GSD npx pattern (bin/install.js copying files to ~/.claude/) | Plugin system is the official modern approach; provides namespacing, marketplace distribution, /plugin install/uninstall. Legacy pattern still works but is being superseded. Use plugin system for new work. |
| commands/ directory for slash commands | skills/ directory with SKILL.md | Both work in plugin system. `commands/` uses simple `.md` files (like GSD). `skills/` uses directories with `SKILL.md` and can include supporting files. Use `commands/` for simple commands, `skills/` when command needs supporting files (e.g., notation presets data). |

**Installation (two approaches -- decision needed):**

Approach A -- Modern Plugin System (recommended):
```bash
# User installs via plugin marketplace or --plugin-dir
claude --plugin-dir ./get-shit-done-math
# Or via marketplace:
/plugin install get-shit-done-math
```

Approach B -- GSD-style npx (legacy but proven):
```bash
# User runs npx to copy files to ~/.claude/
npx get-shit-done-math --claude --global
```

**Recommendation:** Use Approach A (plugin system) as primary distribution. The plugin system provides proper namespacing, install/uninstall/enable/disable, and avoids file conflicts with other plugins. However, keep the npx installer as a fallback since the user specified "npm install -g, then /math:init" which aligns with the GSD pattern. Both can coexist.

## Architecture Patterns

### Recommended Plugin Structure

```
get-shit-done-math/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest: name="math"
├── commands/                    # Slash commands (auto-namespaced as /math:*)
│   ├── problem.md               # /math:problem - guided problem intake wizard
│   ├── notation.md              # /math:notation - notation profile management
│   ├── help.md                  # /math:help - command reference
│   ├── status.md                # /math:status - research state dashboard
│   ├── init.md                  # /math:init - project directory setup
│   ├── prove.md                 # /math:prove - "Available after Phase 4"
│   ├── search.md                # /math:search - "Available after Phase 3"
│   ├── compute.md               # /math:compute - "Available after Phase 5"
│   ├── write.md                 # /math:write - "Available after Phase 7"
│   └── orchestrate.md           # /math:orchestrate - "Available after Phase 6"
├── agents/                      # Specialized agents
│   └── math-intake.md           # Problem intake agent (guided wizard)
├── bin/
│   └── install.js               # npx installer (GSD-compatible)
├── templates/                   # Document schema templates
│   ├── PROBLEM.md               # PROBLEM.md template with frontmatter
│   ├── NOTATION.md              # Notation profile template
│   └── config.json              # Math project config template
├── presets/                     # Notation domain presets
│   ├── algebra.md               # Algebraic notation conventions
│   ├── analysis.md              # Analysis notation conventions
│   ├── topology.md              # Topology notation conventions
│   ├── number-theory.md         # Number theory conventions
│   ├── combinatorics.md         # Combinatorics conventions
│   ├── algebraic-geometry.md    # Algebraic geometry conventions
│   ├── differential-geometry.md # Differential geometry conventions
│   ├── probability.md           # Probability/statistics conventions
│   └── logic.md                 # Mathematical logic conventions
├── protocols/                   # Shared protocols for all agents
│   └── confidence-tiers.md      # VERIFIED/SUGGESTED/SPECULATIVE protocol
├── package.json                 # npm package for npx distribution
└── README.md                    # User documentation
```

### Recommended Project Directory (created by /math:init)

```
.math/                           # Math research workspace (project-level)
├── config.json                  # Project-level math config
├── NOTATION.md                  # Active notation profile
├── PROBLEM.md                   # Current problem definition
├── STATE.md                     # Research state (orchestrator state machine)
└── sessions/                    # Archived research sessions
    └── session-001/
        ├── PROBLEM.md
        └── NOTATION.md
```

**Why `.math/` not `.planning/math/`:** The math plugin is independent from GSD's `.planning/` directory. Using `.math/` keeps concerns separated -- GSD manages its own state in `.planning/`, the math plugin manages math research state in `.math/`. Users may have both installed simultaneously.

### Pattern 1: Plugin Command with "Coming Soon" Gate

**What:** Slash commands for future phases exist from day one but display a message explaining when they become available.
**When to use:** All Phase 2+ commands.
**Example:**

```markdown
---
description: Search mathematical literature (arXiv, Semantic Scholar)
---

This command is not yet available.

**`/math:search` will be available after Phase 3: Literature Search Agent.**

This command will allow you to:
- Search arXiv and Semantic Scholar for papers relevant to your problem
- Retrieve verified references with full metadata
- Synthesize connections between found papers and your research

To see what's available now, run `/math:help`.
```

### Pattern 2: Guided Wizard via Agent Spawning

**What:** The `/math:problem` command spawns a specialized intake agent that conducts a step-by-step conversation, collecting one field at a time.
**When to use:** Problem intake, notation profile setup.
**Example:**

```markdown
---
description: Submit a structured mathematical problem for research
---

<objective>
Guide the user through submitting a mathematical problem. Collect information step-by-step, one field at a time. Write the result as a structured PROBLEM.md.
</objective>

<process>
1. Ask for the problem statement (natural language with optional LaTeX)
2. Convert to full LaTeX and show back for confirmation
3. Ask for mathematical domain (offer preset list + custom option)
4. Ask for desired output type (proof, computation, exploration, conjecture testing)
5. Ask for known results (optional)
6. Ask for notation preferences (optional -- offer to run /math:notation)
7. Ask for references (optional)
8. Write PROBLEM.md to .math/ directory
9. Show summary and next steps
</process>
```

### Pattern 3: YAML Frontmatter for Machine-Readable Routing

**What:** All document schemas use YAML frontmatter that the orchestrator can parse without understanding mathematical content.
**When to use:** Every `.math/*.md` artifact.
**Example:**

```markdown
---
status: defined
domain: algebraic-geometry
type: proof
output_type: formal-proof
confidence_summary:
  verified: 0
  suggested: 0
  speculative: 0
tags: [sheaves, cohomology, vanishing-theorems]
created: 2026-02-08
last_modified: 2026-02-08
notation_profile: algebraic-geometry
---

# Problem Statement
...
```

### Pattern 4: Notation Profile as Structured Document

**What:** Notation profiles are markdown documents with structured sections covering symbols, packages, and style conventions.
**When to use:** Every project's NOTATION.md.
**Example:**

```markdown
---
domain: algebraic-geometry
based_on_preset: algebraic-geometry
modified: true
last_modified: 2026-02-08
---

# Notation Profile

## Symbol Conventions

| Concept | Notation | LaTeX | Notes |
|---------|----------|-------|-------|
| Ideal | Fraktur lowercase | \mathfrak{a}, \mathfrak{p} | Standard for commutative algebra |
| Field | Blackboard bold | \mathbb{F}, \mathbb{K} | |
| Sheaf | Calligraphic | \mathcal{F}, \mathcal{O}_X | |
| Variety | Roman uppercase | X, Y, Z | |

## LaTeX Packages

- amsmath, amssymb, amsthm (always loaded)
- mathtools (enhanced amsmath)
- mathrsfs (script letters for sheaves)
- tikz-cd (commutative diagrams)

## Presentation Style

- Theorem numbering: sequential within sections (Theorem 2.1, 2.2, ...)
- Proof structure: begin with strategy statement, then formal steps
- Equation numbering: only number referenced equations
- Bibliography style: alpha (e.g., [Har77])

## Notation Conflicts Log

| Source Notation | Our Notation | Context |
|----------------|--------------|---------|
| (none yet) | | |
```

### Pattern 5: Confidence Tier Protocol as Shared Reference

**What:** A protocol document that all agents load via @-reference, defining exactly how to mark confidence tiers.
**When to use:** Loaded into every agent's context in Phase 4+.
**Example:**

```markdown
# Confidence Tier Protocol

## Tier Definitions

### VERIFIED [V]
- **Marker:** [V] or green checkmark
- **Meaning:** Cited from published literature with exact reference
- **Requirements:** Must include paper/textbook citation, theorem/lemma number, and exact statement
- **Example:** [V] By the Hahn-Banach theorem (Rudin, Functional Analysis, Thm 3.3), ...

### SUGGESTED [S]
- **Marker:** [S] or blue arrow
- **Meaning:** Derived by the system with explicit justification from verified premises
- **Requirements:** Must cite which verified facts are used and show the deduction chain
- **Example:** [S] From [V] Theorem 3.3 and [V] Lemma 2.1, it follows by direct substitution that...

### SPECULATIVE [~]
- **Marker:** [~] or amber question mark
- **Meaning:** System conjecture, heuristic reasoning, analogy, or creative leap
- **Requirements:** Must state why this is speculative and what would verify it
- **Example:** [~] The pattern suggests this extends to the infinite-dimensional case, though no proof is known. Verification: check against counterexample families in [Chen 2020].

## Active Warnings

When presenting content with speculative steps:
- Count speculative steps and display warning at top
- Format: "This [proof/analysis] contains N speculative step(s) that need verification."
- List which steps are speculative with brief reason

## Manual Override

Users can promote/demote tiers. When overridden:
- Mark as "[V*]", "[S*]", or "[~*]" (asterisk indicates manual override)
- Add note: "Tier manually set by user (original: [tier])"
```

### Anti-Patterns to Avoid

- **Monolithic intake command:** Don't put the entire wizard logic inline in the command .md file. Use a separate agent (math-intake.md) that the command spawns via Task tool. Keeps the command file clean and the agent reusable.
- **Hardcoded notation in agent prompts:** Don't embed notation conventions directly in agent system prompts. Always load NOTATION.md via @-reference so it can be customized per-project.
- **Mixing plugin files with project files:** Plugin files live in the plugin directory (installed to ~/.claude/ or loaded via --plugin-dir). Project files live in .math/ within the user's project. Never write plugin state to the project directory or vice versa.
- **Over-engineering the state machine in Phase 1:** STATE.md should be minimal in Phase 1 -- just tracking whether a problem has been defined and whether notation is configured. The full orchestrator state machine comes in Phase 6.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Command namespacing | Custom namespace parsing in command files | Claude Code plugin system (`plugin.json` with `name: "math"`) | Plugin system auto-namespaces all commands as `/math:*`; handles conflicts, discovery, enable/disable |
| LaTeX rendering in terminal | Custom LaTeX-to-Unicode converter | Raw LaTeX source in terminal output | Mathematicians read LaTeX source fluently; conversion loses information and adds complexity |
| YAML frontmatter parsing | Custom parser in Node.js | Claude's native text parsing + grep for agents, `grep '^status:'` for orchestrator | Agents can read YAML natively; orchestrator only needs simple field extraction |
| Domain preset storage | Database or JSON config system | Markdown files in `presets/` directory | Markdown is readable, editable, version-controllable; agents can load presets via @-reference |
| Installation UX | Custom interactive installer from scratch | Adapt GSD's proven `bin/install.js` pattern | GSD installer handles Claude/OpenCode/Gemini runtimes, settings.json, path replacement -- just customize for math namespace |

**Key insight:** Phase 1 has zero external dependencies. Everything is markdown files, YAML frontmatter, and Claude Code's built-in plugin machinery. The complexity is in design (schemas, protocols, presets), not in code.

## Common Pitfalls

### Pitfall 1: Notation Profile Scope Confusion

**What goes wrong:** User configures notation profile globally but expects it to be per-project. Or user has two research projects with different notation and they bleed into each other.
**Why it happens:** The plugin is global but notation is project-specific. If NOTATION.md is stored in the plugin directory rather than the project directory, all projects share one profile.
**How to avoid:** NOTATION.md lives in `.math/` within the project directory, never in the plugin directory. The `/math:init` command creates `.math/NOTATION.md` from a preset. The plugin only stores preset templates -- the active profile is always project-local.
**Warning signs:** User reports notation from one project appearing in another project.

### Pitfall 2: Problem Statement LaTeX Conversion Failures

**What goes wrong:** User types natural language with inline LaTeX like "Show that $\int_0^1 f(x) dx = 0$ for all $f \in L^2$" and the system's LaTeX conversion either mangles the existing LaTeX or produces invalid LaTeX from the natural language portions.
**Why it happens:** The boundary between "natural language" and "LaTeX" is fuzzy. Users mix them freely.
**How to avoid:** Don't try to convert natural language to LaTeX automatically. Instead: (1) Accept the input as-is, (2) wrap the entire statement in a LaTeX display environment, (3) show the result back to the user for confirmation, (4) let the user edit if needed. The confirmation step is critical -- never silently convert.
**Warning signs:** User repeatedly correcting the LaTeX output, or LaTeX compilation errors downstream.

### Pitfall 3: Confidence Tiers Defined But Never Enforced

**What goes wrong:** The confidence tier protocol is designed in Phase 1 but no agent in Phase 1 actually produces tiered output. By the time Phase 4 (proof agent) arrives, the protocol has drifted or been forgotten.
**Why it happens:** No agent in Phase 1 needs tiers. The protocol exists as documentation only.
**How to avoid:** Include the confidence tier protocol document in a location that Phase 4+ agents will naturally find it. Store as `protocols/confidence-tiers.md` in the plugin directory. When future agent definitions are written, they reference this protocol via @-reference. Also include a simple example in the /math:help output so users see tiers from day one.
**Warning signs:** Phase 4 planner designs a different tier system because they didn't find the Phase 1 protocol.

### Pitfall 4: Slash Commands Not Discoverable

**What goes wrong:** User installs the plugin but doesn't know what commands are available. They try `/math` and nothing happens. They try `/help` and see GSD commands but not math commands.
**Why it happens:** The plugin system namespaces commands but doesn't always make them visible in the main help.
**How to avoid:** `/math:help` must be the first thing the user runs after install. The install script or README should prominently mention this. The help command should list ALL commands (active and coming-soon) with clear status indicators.
**Warning signs:** Users asking "what commands does this plugin have?" after installation.

### Pitfall 5: .math/ Directory Conflicts with Existing Files

**What goes wrong:** `/math:init` creates `.math/` in the user's project but the directory already exists (another tool or manual files).
**Why it happens:** No existence check before creation.
**How to avoid:** `/math:init` must check for existing `.math/` directory and handle gracefully: show what's there, ask user to confirm overwrite or choose a different directory. Never silently overwrite.
**Warning signs:** User loses files after running `/math:init`.

### Pitfall 6: Installer Fragility Across Runtimes

**What goes wrong:** The GSD npx installer pattern works for Claude Code but breaks on OpenCode or Gemini because of different directory structures, command naming, or frontmatter formats.
**Why it happens:** GSD's installer does complex path/frontmatter conversion for each runtime. Math plugin must replicate this.
**How to avoid:** For Phase 1, target Claude Code only. The plugin system (`--plugin-dir`) is Claude Code-specific. Don't attempt OpenCode/Gemini support until the plugin is stable. If using the npx pattern, start by forking GSD's install.js and customizing, not rewriting from scratch.
**Warning signs:** Installation failures on non-Claude-Code runtimes.

## Code Examples

### plugin.json Manifest

```json
// Source: Claude Code plugin documentation (code.claude.com/docs/en/plugins-reference)
{
  "name": "math",
  "version": "0.1.0",
  "description": "AI-powered mathematical research assistant: literature search, proof collaboration, computation, and LaTeX output",
  "author": {
    "name": "get-shit-done-math"
  },
  "homepage": "https://github.com/glittercowboy/get-shit-done-math",
  "repository": "https://github.com/glittercowboy/get-shit-done-math",
  "license": "MIT",
  "keywords": ["math", "research", "proof", "latex", "claude-code"]
}
```

### Active Command Example (/math:init)

```markdown
---
description: Initialize a math research project in the current directory
---

<objective>
Set up the .math/ directory structure for a new math research project.
Creates NOTATION.md from a domain preset and an empty PROBLEM.md template.
</objective>

<process>
1. Check if .math/ already exists
   - If yes: show contents, ask to confirm reinitialize or abort
   - If no: proceed

2. Create directory structure:
   ```
   .math/
   ├── config.json
   ├── NOTATION.md
   ├── PROBLEM.md (empty template)
   └── STATE.md
   ```

3. Ask user for primary mathematical domain (for notation preset):
   - Offer preset list: algebra, analysis, topology, number theory, combinatorics,
     algebraic geometry, differential geometry, probability, logic, custom
   - If custom: start with a minimal default profile

4. Load the selected preset into NOTATION.md

5. Display summary:
   - "Math research project initialized in .math/"
   - "Notation profile: [domain] (customize with /math:notation)"
   - "Next: submit a problem with /math:problem"
</process>
```

### Disabled Command Example (/math:prove)

```markdown
---
description: Collaborate on developing mathematical proofs (coming soon)
---

**`/math:prove` is not yet available.**

This command will be activated in **Phase 4: Proof Collaboration Agent**.

When available, `/math:prove` will allow you to:
- Get proof strategy suggestions (direct, contradiction, induction, contrapositive)
- Receive structured reasoning chains with confidence tiers
- Identify gaps in partial proofs
- Paste existing LaTeX proofs for gap analysis

**Currently available commands:**
- `/math:problem` -- submit a structured math problem
- `/math:notation` -- configure notation profile
- `/math:help` -- see all commands
- `/math:status` -- view research state
- `/math:init` -- initialize project
```

### PROBLEM.md Schema

```markdown
---
status: defined
domain: analysis
subdomain: functional-analysis
type: proof
output_type: formal-proof
tags: [banach-spaces, operators, spectral-theory]
created: 2026-02-08
last_modified: 2026-02-08
notation_profile: analysis
---

# Problem Statement

Let $T: X \to X$ be a bounded linear operator on a Banach space $X$. Show that
if $\|T\| < 1$, then $I - T$ is invertible and $(I - T)^{-1} = \sum_{n=0}^{\infty} T^n$.

## Mathematical Domain

Functional Analysis (Banach Spaces, Operator Theory)

## Desired Output

Formal proof suitable for publication, with all steps justified.

## Known Results

- The Neumann series converges when $\|T\| < 1$ (standard result)
- The partial sums form a Cauchy sequence in the operator norm

## Notation Preferences

See .math/NOTATION.md (analysis preset)

## References

- Rudin, Functional Analysis, Chapter 10
- Conway, A Course in Functional Analysis, Section II.1
```

### STATE.md Schema (Phase 1 -- Minimal)

```markdown
---
current_state: INITIALIZED
problem_defined: false
notation_configured: true
session_count: 0
---

# Research State

## Current Status

Project initialized. No problem submitted yet.

## History

| Timestamp | Event | Details |
|-----------|-------|---------|
| 2026-02-08 14:30 | Project initialized | Domain: analysis, Notation: analysis preset |
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Claude Code commands in `~/.claude/commands/` | Plugin system with `.claude-plugin/plugin.json` | Claude Code 1.0.33+ (2025-2026) | Auto-namespacing, marketplace distribution, install/uninstall/enable/disable |
| Simple `.md` files as commands | Skills system with `SKILL.md` in directories | Claude Code 2025-2026 | Skills can include supporting files, Claude can auto-invoke them |
| GSD npx `bin/install.js` copying to `~/.claude/` | `/plugin install` from marketplace or `--plugin-dir` for local | Claude Code plugin system (2025-2026) | Cleaner install/uninstall, versioning, no file conflicts |

**Important note on GSD vs plugin system:** The GSD codebase this project inherits uses the legacy npx installer approach. The modern Claude Code plugin system is the current standard. Both approaches work. The plugin system is recommended for new plugins but the npx pattern is battle-tested in GSD. The planner should decide whether to use plugin system, npx pattern, or both.

## Notation Domain Presets (Claude's Discretion)

Recommended preset list covering major mathematical domains. Each preset is a markdown file defining standard notation conventions for that domain.

| Preset | Key Conventions | Common Packages |
|--------|----------------|-----------------|
| **algebra** | Multiplicative group notation, fraktur ideals, blackboard bold fields, ring homomorphisms as $\varphi$ | amsmath, amssymb |
| **analysis** | Epsilon-delta, $\|f\|_p$ norms, $L^p$ spaces, measure notation $d\mu$ | amsmath, amssymb, mathtools |
| **topology** | Open sets as $U, V$, continuous maps as $f: X \to Y$, homotopy as $\simeq$ | amsmath, amssymb, tikz-cd |
| **number-theory** | $\mathbb{Z}, \mathbb{Q}, \mathbb{F}_p$, Legendre symbol, Dirichlet characters | amsmath, amssymb |
| **combinatorics** | Binomial coefficients $\binom{n}{k}$, generating functions, graph notation | amsmath, amssymb |
| **algebraic-geometry** | Calligraphic sheaves $\mathcal{F}$, structure sheaf $\mathcal{O}_X$, cohomology $H^i$ | amsmath, amssymb, mathrsfs, tikz-cd |
| **differential-geometry** | Differential forms $\omega$, covariant derivative $\nabla$, Lie bracket $[X,Y]$ | amsmath, amssymb, tensor (optional) |
| **probability** | $\mathbb{P}, \mathbb{E}$, $\sigma$-algebras, filtrations $\mathcal{F}_t$ | amsmath, amssymb, mathtools |
| **logic** | $\models, \vdash$, first-order quantifiers, sequents | amsmath, amssymb, proof (optional) |

## Confidence Tier Visual Markers (Claude's Discretion)

**Recommended markers for terminal output:**

| Tier | Marker | Terminal Display | Rationale |
|------|--------|-----------------|-----------|
| VERIFIED | `[V]` | `[V]` (no special formatting needed -- text is sufficient) | Clean, scannable, works in all terminals |
| SUGGESTED | `[S]` | `[S]` | Distinct from [V], clearly not verified |
| SPECULATIVE | `[~]` | `[~]` | Tilde universally connotes approximation/uncertainty |
| Manual override | `[V*]`, `[S*]`, `[~*]` | Asterisk suffix | Clear that tier was manually set |

**Rationale for text markers over color/emoji:** Terminal environments vary widely. Not all terminals support colors. Emoji rendering is inconsistent. Plain text markers `[V]`, `[S]`, `[~]` are universally readable, greppable, and unambiguous. They also survive copy-paste into documents.

## Open Questions

1. **Plugin system vs npx installer for distribution**
   - What we know: Claude Code plugin system (`.claude-plugin/`) is the modern standard. GSD uses npx installer (`bin/install.js`). Both work. Plugin system provides namespacing, marketplace, enable/disable. Npx is simpler and proven.
   - What's unclear: Whether the user's stated "npm install -g, then /math:init" preference maps better to npx or plugin system. Plugin system uses `--plugin-dir` for local dev and `/plugin install` for marketplace.
   - Recommendation: Build as a plugin (`.claude-plugin/plugin.json`) with `--plugin-dir` for development. Also provide `npx get-shit-done-math` for GSD-compatible installation. Both can coexist. Let the planner decide the primary distribution path.

2. **Where to store the confidence tier protocol**
   - What we know: The protocol must be loadable by all future agents via @-reference. It must live in the plugin directory, not the project directory.
   - What's unclear: Exact path within the plugin. Could be `protocols/confidence-tiers.md`, or `get-shit-done-math/references/confidence-tiers.md` (GSD pattern), or `skills/confidence/SKILL.md`.
   - Recommendation: Store as `protocols/confidence-tiers.md` at plugin root. Agents will load via `@~/.claude/plugins/math/protocols/confidence-tiers.md` or `@${CLAUDE_PLUGIN_ROOT}/protocols/confidence-tiers.md`. The `protocols/` directory is a natural fit for shared conventions.

3. **Project directory naming: `.math/` vs `.planning/math/`**
   - What we know: GSD uses `.planning/`. The architecture research suggested `.planning/math/`. But math plugin is independent from GSD.
   - What's unclear: Whether users will have both GSD and math plugin installed simultaneously.
   - Recommendation: Use `.math/` as the project directory. This avoids collision with `.planning/` and makes the math workspace independent. If the user also uses GSD, both directories coexist cleanly.

4. **How /math:status works without session persistence (Phase 2)**
   - What we know: Phase 1 delivers `/math:status` but session persistence is Phase 2.
   - What's unclear: What status to show when there's no session history.
   - Recommendation: In Phase 1, `/math:status` reads `.math/STATE.md` and `.math/PROBLEM.md` to show: whether a problem is defined, what domain is selected, notation profile status, and which commands are available. Simple read-and-display, no session tracking.

## Sources

### Primary (HIGH confidence)
- [Claude Code Plugins Reference](https://code.claude.com/docs/en/plugins-reference) -- Plugin manifest schema, component structure, namespacing rules, installation scopes, CLI commands
- [Claude Code Create Plugins](https://code.claude.com/docs/en/plugins) -- Plugin quickstart, directory layout, testing with `--plugin-dir`, skills vs commands
- GSD codebase analysis (`/Users/matteo/Desktop/Coding/get-shit-done-math/`) -- Command structure, agent patterns, installer, config system
- GSD project research (`.planning/research/`) -- Architecture patterns, pitfalls, stack decisions

### Secondary (MEDIUM confidence)
- [Claude Code Slash Commands](https://code.claude.com/docs/en/slash-commands) -- Referenced in search results; confirms namespace format and command discovery
- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) -- Community patterns for plugins
- [Claude Code Plugin Repository](https://github.com/anthropics/claude-code/blob/main/plugins/README.md) -- Official plugin examples

### Tertiary (LOW confidence)
- Notation presets content -- Based on mathematical domain knowledge from training data; specific conventions should be validated with mathematicians
- Terminal marker choices -- Recommendation based on general CLI UX principles; actual user preference may differ

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- Claude Code plugin system is well-documented; GSD patterns are directly observable in codebase
- Architecture: HIGH -- Plugin directory structure follows official documentation; document schemas are straightforward markdown design
- Pitfalls: HIGH -- Derived from concrete GSD experience + Claude Code plugin documentation
- Notation presets: MEDIUM -- Domain conventions are well-known but specific symbol tables need mathematician validation
- Confidence tier markers: MEDIUM -- Design decision within Claude's discretion; recommendation is well-reasoned but alternatives are valid

**Research date:** 2026-02-08
**Valid until:** 60 days (plugin system is stable; notation presets are timeless; Claude Code plugin docs may evolve)
