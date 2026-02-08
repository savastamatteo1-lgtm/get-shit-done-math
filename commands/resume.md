---
description: Resume a math research session with full context restoration
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
---

# /math:resume -- Session Restoration

Resume a math research session by reading all problem artifacts, generating a structured dashboard with provenance tracking, and displaying a suggested next action.

## Path Resolution

1. Read `.math/config.json`
2. Get `current_problem` slug
3. Resolve all file paths as `.math/problems/{current_problem}/FILENAME.md`

## Process

### Step 1: Validate project exists

Check if `.math/config.json` exists.

- **If `.math/config.json` does not exist:** Tell the user: "No math project found. Run `/math:init` first." Stop here.
- **If it exists:** Proceed to Step 2.

### Step 2: Resolve active problem

Read `.math/config.json`. Extract the `current_problem` field.

- **If `current_problem` is empty string or missing:** Tell the user: "No active problem. Run `/math:init` to create one, or `/math:switch` to select an existing one." Stop here.
- **If `current_problem` is set:** Construct the problem directory path: `.math/problems/{current_problem}/`
- **If the problem directory does not exist:** Warn: "Problem directory for '{current_problem}' not found. State may be corrupted. Run `/math:init` to reinitialize." Stop here.

### Step 3: Read all artifacts

Read each file from the problem directory, extracting key data for the dashboard. Handle missing files gracefully -- future phases create LITERATURE.md, PROOF.md, COMPUTATION.md, so they may not exist yet.

**Files to read and data to extract:**

1. **PROBLEM.md** (required)
   - Frontmatter: `status`, `domain`, `type`, `output_type`
   - Body: first 3-5 lines after frontmatter (problem statement preview)
   - If missing: set problem_preview to "Problem not yet submitted. Run `/math:problem`."

2. **STATE.md** (required)
   - Frontmatter: `current_state`, `problem_defined`, `notation_configured`, `session_count`
   - If missing: warn user about corrupted state, suggest `/math:init`

3. **JOURNAL.md** (required)
   - Frontmatter: `total_entries`, `last_entry`, `strategies_tried`
   - Body: find the most recent entry in the `## Proof Attempts` section (the last `### PROOF-*` heading) for the "Last Approach Tried" dashboard section
   - Also scan for most recent `### LIT-*` and `### COMP-*` entries to extract provenance timestamps and agent names
   - If missing or corrupted frontmatter: warn "Journal needs repair" and show "No journal entries yet"

4. **NOTATION.md** (required)
   - Frontmatter: `domain`, `based_on_preset`, `modified`

5. **LITERATURE.md** (optional -- Phase 3)
   - If exists: extract paper count from frontmatter, key findings from body
   - If does not exist: set literature_summary to "No literature search yet"

6. **PROOF.md** (optional -- Phase 4)
   - If exists: extract step count, gap count, and confidence tier counts ([V], [S], [~]) from body content
   - If does not exist: set proof_summary to "No proof attempted yet"

7. **COMPUTATION.md** (optional -- Phase 5)
   - If exists: extract computation count, key results
   - If does not exist: set computation_summary to "No computations yet"

### Step 4: Generate DASHBOARD.md

Using the `templates/DASHBOARD.md` structure, fill all placeholders with the extracted data.

**Frontmatter fields:**
- `generated`: current ISO 8601 timestamp
- `problem`: the `current_problem` slug
- `state`: the `current_state` value from STATE.md
- `session_number`: STATE.md `session_count` + 1 (the new session number)

**Title:**
- `{PROBLEM_TITLE}`: from PROBLEM.md frontmatter `domain` and first line of body, or the problem slug if no body

**Problem section:**
- First 3-5 lines of PROBLEM.md body (after frontmatter)
- If problem not yet defined: "Problem not yet submitted. Run `/math:problem`."

**Proof Progress subsection:**
- If PROOF.md exists: show current strategy, steps completed/total, open gaps count, confidence tier counts as `{V_count} [V], {S_count} [S], {~_count} [~]`
- If PROOF.md does not exist: "No proof attempted yet"
- Provenance line: `_Source: {agent_name}, {timestamp}_` extracted from the most recent PROOF- journal entry. If no PROOF entries, show `_Source: N/A_`

**Literature Found subsection:**
- If LITERATURE.md exists: show paper count, key theorem/finding
- If LITERATURE.md does not exist: "No literature search yet"
- Provenance line: `_Source: {agent_name}, {timestamp}_` from the most recent LIT- journal entry. If no LIT entries, show `_Source: N/A_`

**Computations subsection:**
- If COMPUTATION.md exists: show computation count, key results
- If COMPUTATION.md does not exist: "No computations yet"
- Provenance line: `_Source: {agent_name}, {timestamp}_` from the most recent COMP- journal entry. If no COMP entries, show `_Source: N/A_`

**Last Approach Tried subsection:**
- From JOURNAL.md: find the most recent PROOF- entry in the body
- Show: approach description (from "What was tried" field), outcome (SUCCEEDED/FAILED/PARTIAL/etc.), journal entry ID
- If no PROOF journal entries: "No approaches tried yet"

**Suggested Next Action section:**
Generate a context-specific suggestion based on the current state. Use this decision tree in order -- the FIRST matching condition produces the suggestion:

1. **State is INITIALIZED and no problem defined** (`problem_defined` is false):
   "Submit a problem with `/math:problem` to begin your research."

2. **Problem defined but no literature searched** (no LIT- entries in JOURNAL.md `strategies_tried` or body):
   "Consider starting with a literature search to find relevant existing results. `/math:search` (available Phase 3)"

3. **Literature found but no proof attempted** (LIT- entries exist but no PROOF- entries):
   "Literature has been reviewed. Consider starting proof development. `/math:prove` (available Phase 4)"

4. **Proof in progress with open gaps** (PROOF.md exists and has gaps):
   "The proof has a gap at {gap_description}. Consider {specific suggestion based on gap type -- e.g., 'searching for a lemma to bridge this step' or 'trying a computational verification'}."
   Reference the specific gap from the most recent PROOF journal entry.

5. **Proof in progress and last approach failed/abandoned** (most recent PROOF- entry has outcome FAILED or ABANDONED):
   "The last approach ({strategy_type}) was {outcome} because: {reasoning}. Consider trying an alternative strategy. Review journal insights from {entry_id} before choosing a new approach."
   Reference actual journal entry data.

6. **Proof complete** (all PROOF- entries resolved, no open gaps):
   "Proof development appears complete. Consider generating LaTeX output. `/math:write` (available Phase 7)"

7. **Default** (no specific condition matched):
   "Review the current state and decide on next steps. Use `/math:help` to see available commands."

The suggestion MUST be **specific** -- referencing actual artifact data, entry IDs, strategy names, or gap descriptions when available. Never output a generic suggestion when artifact data can make it specific.

**Write the filled DASHBOARD.md** to `.math/problems/{current_problem}/DASHBOARD.md`.

### Step 5: Update STATE.md

1. Increment `session_count` by 1 in STATE.md frontmatter
2. Add a history entry to the History table at the bottom of STATE.md:

```
| {ISO_TIMESTAMP} | Session resumed | Session #{new_session_number} |
```

### Step 6: Display dashboard

Output the full contents of the generated DASHBOARD.md to the user. The dashboard IS the session restoration -- the user reads it and knows exactly where they left off, what has been tried, and what to do next.

### Step 7: Update last_active

Update the problem's `last_active` timestamp in `.math/config.json`:
- Find the entry in the `problems` array where `slug` matches `current_problem`
- Set its `last_active` to the current ISO 8601 timestamp

## Error Handling

- **Missing optional artifacts** (LITERATURE.md, PROOF.md, COMPUTATION.md): Expected in early phases. Show "not yet" messages in the dashboard. Do NOT error or warn.
- **Missing required artifacts** (PROBLEM.md, STATE.md, JOURNAL.md, NOTATION.md): These should exist after `/math:init`. If missing, warn and suggest `/math:init` to reinitialize.
- **Corrupted JOURNAL.md frontmatter:** Warn "Journal frontmatter appears corrupted" and attempt to regenerate from body content (count entries, find last timestamp). If body is also empty, show "Journal needs repair -- consider re-running `/math:init`."
- **Empty problem directory:** Suggest `/math:init` to reinitialize.
- **No entries in any category:** Normal for new problems. Dashboard shows "not yet" for all sections and suggests the first step.

## References

- `@templates/DASHBOARD.md` -- Dashboard structure template with placeholder format
- `@protocols/journal-protocol.md` -- Journal entry format for reading and parsing entries
- `@protocols/confidence-tiers.md` -- Confidence tier markers [V], [S], [~] for dashboard display
