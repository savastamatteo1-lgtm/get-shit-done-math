---
description: Initialize a math research project with notation presets and document templates
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---

# /math:init -- Initialize Math Research Project

Create the `.math/` working directory with notation profile, problem template, state tracker, and configuration.

## Process

### Step 1: Check for existing project

Check if `.math/` directory already exists.

- **If `.math/` exists:** Display its contents (list files and show config.json summary). Ask the user: "A math research project already exists. Reinitialize? This will overwrite existing files. (yes/no)". If user says no, abort. If yes, proceed and overwrite.
- **If `.math/` does not exist:** Proceed to Step 2.

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

### Step 3: Create .math/ directory and populate files

Create the `.math/` directory and populate it with project files:

1. **Read the selected preset:** Read `presets/{domain}.md` to get domain-specific notation conventions. If user chose `custom`, skip preset loading.

2. **Create .math/PROBLEM.md:** Read `templates/PROBLEM.md` and copy it to `.math/PROBLEM.md` as-is (empty template ready for /math:problem).

3. **Create .math/NOTATION.md:** Read `templates/NOTATION.md` for structure. Merge the selected preset content into the template:
   - Set frontmatter: `domain: {domain}`, `based_on_preset: {domain}`, `modified: false`, `last_modified: {ISO timestamp}`
   - Fill the Symbol Conventions table with rows from the preset
   - Fill LaTeX Packages section (base + domain-specific) from the preset
   - Fill Presentation Style subsections from the preset
   - If `custom`: leave template mostly empty, set `based_on_preset: ""`, just populate the base LaTeX packages

4. **Create .math/STATE.md:** Read `templates/STATE.md` and copy to `.math/STATE.md`. Update:
   - `current_state: INITIALIZED`
   - `notation_configured: true` (false if custom with no changes)
   - Add a History table entry: current timestamp, "Project initialized", "Domain: {domain}"

5. **Create .math/config.json:** Read `templates/config.json` and copy to `.math/config.json`. Fill:
   - `domain: "{domain}"`
   - `notation_preset: "{domain}"` (empty string if custom)
   - `created: "{ISO timestamp}"`

### Step 4: Display summary

Show the user:

```
Math research project initialized in .math/

  Notation profile: {domain}
  Files created: PROBLEM.md, NOTATION.md, STATE.md, config.json

Next: Submit a problem with /math:problem
```

## Error Handling

- If `presets/{domain}.md` is not found for a valid domain name, warn the user and fall back to the custom (empty) profile.
- If `.math/` directory creation fails, report the error and stop.

## References

- `@presets/{domain}.md` -- Domain-specific notation conventions
- `@templates/PROBLEM.md` -- Problem statement template
- `@templates/NOTATION.md` -- Notation profile template
- `@templates/STATE.md` -- Research state template
- `@templates/config.json` -- Configuration template
