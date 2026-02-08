---
description: View and configure the notation profile for your math research project
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---

# /math:notation -- Manage Notation Profile

View, switch, or customize the notation conventions used across your math research project.

## Process

### Step 1: Validate project exists

Check if `.math/` directory exists.

- **If `.math/` does not exist:** Tell the user: "No math project found. Run `/math:init` first to set up your project." Stop here.
- **If `.math/` exists:** Proceed to Step 2.

### Step 2: Display current profile summary

Read `.math/NOTATION.md` and show a summary:

```
Notation Profile
================
Domain: {domain from frontmatter}
Preset: {based_on_preset} {if modified: "(customized)"}
Symbol conventions: {count} defined
Additional packages: {list or "none"}
```

### Step 3: Offer options

Present the user with these options:

```
What would you like to do?

  a. View full profile -- Display entire NOTATION.md
  b. Switch domain preset -- Change to a different domain's conventions
  c. Add/edit symbol -- Add or modify a symbol convention
  d. Add/remove package -- Manage LaTeX packages
  e. Edit presentation style -- Modify theorem numbering, proof structure, etc.
```

Wait for the user to choose.

### Step 4: Execute chosen action

**Option a: View full profile**
- Read and display the entire `.math/NOTATION.md` content.

**Option b: Switch domain preset**
- Show the preset list: algebra, analysis, topology, number-theory, combinatorics, algebraic-geometry, differential-geometry, probability, logic
- Read the selected preset from `presets/{domain}.md`
- Ask: "Keep your custom modifications or start fresh from the preset? (keep/fresh)"
  - **keep:** Merge preset symbols into existing table (add new rows, don't remove custom ones). Update domain and based_on_preset in frontmatter.
  - **fresh:** Replace entire notation content with preset. Reset modified to false.
- Write updated `.math/NOTATION.md`

**Option c: Add/edit symbol convention**
- Ask: "What concept? (e.g., 'Inner product')"
- Ask: "What notation? (e.g., \langle x, y \rangle)"
- Ask: "LaTeX command? (e.g., `\langle x, y \rangle`)"
- Ask: "Notes? (optional, press enter to skip)"
- Add or update the row in the Symbol Conventions table in `.math/NOTATION.md`

**Option d: Add/remove LaTeX package**
- Show current packages (base + additional)
- Ask: "Add or remove a package? (add/remove)"
  - **add:** Ask for package name and purpose. Add to Additional packages section.
  - **remove:** Show removable packages (cannot remove base: amsmath, amssymb, amsthm). Ask which to remove.
- Write updated `.math/NOTATION.md`

**Option e: Edit presentation style**
- Show current settings for: theorem numbering, proof structure, equation numbering, bibliography style
- Ask which to change
- Collect new preference
- Write updated `.math/NOTATION.md`

### Step 5: Update metadata

After any change (options b through e):
- Set `modified: true` in frontmatter
- Set `last_modified: {ISO timestamp}` in frontmatter
- Write the updated `.math/NOTATION.md`

Confirm: "Notation profile updated."

## References

- `@presets/*.md` -- Domain notation presets for switching
- `.math/NOTATION.md` -- Current project notation profile
