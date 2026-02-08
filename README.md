<div align="center">

# get-shit-done-math

**AI-powered mathematical research assistant for Claude Code.**

[![npm version](https://img.shields.io/npm/v/get-shit-done-math?style=for-the-badge&logo=npm&logoColor=white&color=CB3837)](https://www.npmjs.com/package/get-shit-done-math)
[![GitHub stars](https://img.shields.io/github/stars/savastamatteo1-lgtm/get-shit-done-math?style=for-the-badge&logo=github&color=181717)](https://github.com/savastamatteo1-lgtm/get-shit-done-math)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)

<br>

```bash
npx get-shit-done-math
```

</div>

---

## What This Is

A structured research workflow for mathematical problem-solving inside Claude Code. Define a problem, set up notation, search the literature, build proofs, run computations, and produce LaTeX output — all tracked in a persistent workspace with confidence-tagged reasoning.

The system maintains your research state across sessions: problem definitions, notation conventions, literature references, proof progress, and a research journal. Switch between multiple problems, resume where you left off, and archive completed work.

---

## Getting Started

### Install

```bash
npx get-shit-done-math
```

The installer adds `/math:*` commands to your Claude Code environment. Verify with:

```
/math:help
```

### Update

```bash
npx get-shit-done-math@latest
```

### Quick Start

```
/math:init                          # Create .math/ workspace
/math:problem                       # Define your research problem
/math:notation                      # Set notation conventions (or use a domain preset)
/math:search                        # Search arXiv + Semantic Scholar for relevant work
/math:status                        # Check current state of all problems
```

---

## How It Works

The workflow moves through a series of stages, each producing structured documents that feed into the next.

```
init -> problem -> notation -> search -> prove -> compute -> orchestrate -> write
 |         |          |          |         |         |            |           |
 v         v          v          v         v         v            v           v
.math/  PROBLEM.md NOTATION.md LITERATURE PROOF.md  COMPUTE.md  STRATEGY   OUTPUT.tex
        defines    conventions  .md        develops  numerical   coordi-    final
        the        for symbols  known      the       validation  nates      document
        question   and terms    results    argument              stages
```

**Active stages** (Phases 1-3): `init`, `problem`, `notation`, `search`, `status`, `resume`, `switch`, `archive`

**Coming soon** (Phases 4-7): `prove`, `compute`, `orchestrate`, `write`

---

## Commands

### Active

| Command | Description |
|---------|-------------|
| `/math:init` | Create or upgrade the `.math/` workspace with directory structure, config, and templates |
| `/math:problem` | Define a new research problem with domain, key questions, and known results |
| `/math:notation` | Set notation conventions — manually or from a domain preset |
| `/math:search` | Search arXiv and Semantic Scholar for relevant papers; results go to `LITERATURE.md` |
| `/math:status` | Show the current state of all problems, active problem, and progress |
| `/math:resume` | Restore context from a previous session using the research journal |
| `/math:switch` | Switch the active problem in a multi-problem workspace |
| `/math:archive` | Archive a completed or paused problem |
| `/math:help` | Show all commands and usage |

### Coming Soon

| Command | Description |
|---------|-------------|
| `/math:prove` | Develop proofs with confidence-tagged steps and gap tracking |
| `/math:compute` | Run numerical experiments and symbolic computations to support proofs |
| `/math:orchestrate` | Coordinate multi-stage research strategies across proof and compute |
| `/math:write` | Generate LaTeX output from accumulated research artifacts |

---

## Features

### Domain Presets

Nine built-in notation presets cover common mathematical domains. Load one during `/math:notation` to get standard symbol conventions, common operators, and naming patterns for your field.

| Preset | Covers |
|--------|--------|
| `algebra` | Groups, rings, fields, modules, homomorphisms |
| `analysis` | Metric spaces, convergence, measure, integration |
| `topology` | Open/closed sets, continuity, compactness, homotopy |
| `number-theory` | Primes, congruences, Dirichlet characters, L-functions |
| `combinatorics` | Graphs, generating functions, binomial coefficients |
| `algebraic-geometry` | Schemes, sheaves, cohomology, varieties |
| `differential-geometry` | Manifolds, connections, curvature, differential forms |
| `probability` | Random variables, expectations, distributions, processes |
| `logic` | First-order logic, models, computability, set theory |

Presets can be extended or overridden per problem.

### Confidence Tiers

Every claim, step, and result is tagged with a confidence marker:

| Marker | Tier | Meaning |
|--------|------|---------|
| `[V]` | Verified | Cited from published literature with full reference |
| `[S]` | Suggested | Derived from verified premises with explicit justification |
| `[~]` | Speculative | Conjecture, heuristic, or unverified claim |

Outputs containing speculative steps display a warning with the count and location of each `[~]` marker. Users can manually override tiers using `[V*]`, `[S*]`, `[~*]` notation, and overrides persist across sessions.

The markers are plain text by design — greppable, copy-paste safe, and compatible with LaTeX export.

### Literature Search

`/math:search` queries arXiv and Semantic Scholar, verifies results against source APIs, and writes a structured `LITERATURE.md` file. Each entry includes:

- Full citation (authors, title, year, venue)
- Relevance assessment to your specific problem
- Key results and techniques
- Connections to other papers in the collection
- Confidence tier for each referenced claim

Search with a broad sweep based on your problem definition, or target specific topics:

```
/math:search                                 # Broad search from problem definition
/math:search "spectral gap bounded operator"  # Targeted topic search
```

### Multi-Problem Workspace

A single `.math/` directory supports multiple concurrent research problems. Each problem gets its own subdirectory under `.math/problems/` with independent state:

```
.math/
  config.json            # Active problem pointer, workspace settings
  JOURNAL.md             # Research journal across all problems
  problems/
    spectral-gap/
      PROBLEM.md
      NOTATION.md
      LITERATURE.md
      STATE.md
    fixed-point/
      PROBLEM.md
      NOTATION.md
      ...
```

Use `/math:switch` to change the active problem, `/math:archive` to shelve completed work.

### Research Journal and Session Resumption

The research journal (`JOURNAL.md`) tracks dated entries of what you worked on, what you found, and what questions remain. When you return to a problem after a break, `/math:resume` reads the journal and restores your context — no manual re-explanation needed.

---

## Architecture

The system follows an **orchestrator-agent-document** pattern built on the [Get Shit Done](https://github.com/glittercowboy/get-shit-done) framework for Claude Code. Each `/math:*` command is a thin orchestrator that coordinates specialized agents, and all state is persisted as markdown documents in `.math/`.

```
Command (orchestrator)
  -> Agent (specialized logic)
    -> Document (.math/**/*.md)
      -> Protocol (confidence-tiers, journal, literature)
```

Protocols enforce cross-cutting concerns: confidence tagging, journal logging, and literature verification apply uniformly across all agents.

---

## Roadmap

| Phase | Name | Status |
|-------|------|--------|
| 1 | Foundation — workspace, problem definition, notation, domain presets | Complete |
| 2 | Session State — multi-problem support, journal, resume, switch, archive | Complete |
| 3 | Literature Search — arXiv + Semantic Scholar integration | Complete |
| 4 | Proof Engine — structured proof development with gap tracking | Planned |
| 5 | Compute — numerical experiments and symbolic computation | Planned |
| 6 | Orchestration — multi-stage research strategy coordination | Planned |
| 7 | LaTeX Output — document generation from research artifacts | Planned |

---

## Configuration

### Recommended Permissions

For uninterrupted workflow, allow the tools the math commands need. Add to `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(date:*)",
      "Bash(mkdir:*)",
      "Bash(ls:*)",
      "WebFetch"
    ]
  }
}
```

The literature search agent uses `WebFetch` to query arXiv and Semantic Scholar APIs.

### Notation Setup

During `/math:init` or `/math:notation`, you can:

1. **Load a domain preset** — Pre-configured conventions for your field
2. **Define custom notation** — Specify your own symbol conventions
3. **Extend a preset** — Start from a preset and override specific symbols

Notation conventions are stored in `NOTATION.md` per problem and are respected by all agents. When cited sources use different notation, the system flags conflicts with `[NC]` markers and logs translations.

---

## Contributing

Contributions welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

<div align="center">

Built on the [Get Shit Done](https://github.com/glittercowboy/get-shit-done) framework.

</div>
