# Phase 1: Foundation & Plugin Setup - Context

**Gathered:** 2026-02-08
**Status:** Ready for planning

<domain>
## Phase Boundary

Deliver the foundational document architecture for the math research assistant plugin: structured problem intake via guided wizard, notation profile system with domain presets, confidence tier protocol (VERIFIED/SUGGESTED/SPECULATIVE), and the full slash command surface with Phase 1 commands active and future commands shown as coming soon. No agent capabilities, no session persistence — those are later phases.

</domain>

<decisions>
## Implementation Decisions

### Problem Intake Flow
- Guided wizard: step-by-step prompts collecting one field at a time
- Required fields: problem statement, math domain, desired output type
- Optional fields: known results, notation preferences, references
- Problem statement accepts natural language with optional inline LaTeX — system converts to full LaTeX internally and shows it back to the user for confirmation
- Math domain selection: preset list of common domains (algebra, analysis, topology, number theory, combinatorics, etc.) with option to type a custom domain
- Output stored as structured PROBLEM.md that downstream agents can consume

### Notation Profile Design
- Domain-based presets: system offers notation presets by math domain (e.g., "algebraic geometry" preset includes standard conventions for sheaves, varieties, etc.) — user can tweak after selection
- Per-project scope: each research session/project has its own notation profile stored in the project directory
- Full style guide coverage: profiles include symbol conventions (fraktur ideals, blackboard bold sets), preferred LaTeX packages (amsmath, mathtools, etc.), AND presentation style (theorem numbering, proof structure, formatting conventions)
- Notation conflict handling: flag + translate — system translates all output to match the user's profile, but flags where the original source (e.g., cited paper) used different conventions

### Confidence Tier Presentation
- Visual markers: subtle color/icon markers next to each step — visible but not disruptive to reading flow
- Source-based tier criteria:
  - VERIFIED = cited from published literature (known theorem, published result)
  - SUGGESTED = derived by the system with explicit justification from verified premises
  - SPECULATIVE = system's conjecture, heuristic reasoning, creative leap, or analogy
- Manual override: user can promote or demote any tier level — system notes it was manually set
- Active warnings: system flags clear warnings when presenting proofs with speculative steps (e.g., "This proof contains 2 speculative steps that need verification")

### Slash Command Surface
- Namespaced convention: all commands under /math:* (e.g., /math:problem, /math:prove, /math:search, /math:compute)
- Full surface on install: all planned commands visible from Phase 1 — unavailable ones display "Available after Phase X" message
- Phase 1 active commands: /math:problem (submit problem), /math:notation (set notation profile), /math:help (command listing), /math:status (research state dashboard), /math:init (project setup)
- Future commands shown but disabled: /math:prove, /math:search, /math:compute, /math:write, /math:orchestrate
- Installation: npm install -g, then /math:init in Claude Code to set up project directory

### Claude's Discretion
- Exact list of domain presets for notation profiles
- Specific icon/color choices for confidence tier markers
- PROBLEM.md internal schema and field naming
- Help command formatting and layout
- Status dashboard information density

</decisions>

<specifics>
## Specific Ideas

- User sees the full vision from day one — disabled commands show which phase unlocks them, setting expectations for the complete pipeline
- Both /math:help (command reference) and /math:status (research state dashboard) available — help for discoverability, status for situational awareness
- Notation conflict flagging is important: when literature agent cites a paper using different conventions, user should know the original notation was different from what they're seeing

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 01-foundation-plugin-setup*
*Context gathered: 2026-02-08*
