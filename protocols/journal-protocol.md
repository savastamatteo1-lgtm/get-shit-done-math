# Journal Protocol

All agents in the math plugin that perform significant research actions MUST follow this protocol when writing journal entries. This protocol defines the entry format, required fields, strategy taxonomy, frontmatter update rules, and dead-end detection procedure. It is the single source of truth for journal writing.

## Journal Entry Format

Every journal entry follows this template exactly. All fields are required unless marked optional.

```
### {CATEGORY}-{NNN}: {Title}
- **Timestamp:** {ISO_TIMESTAMP}
- **Agent:** {agent-name}
- **Strategy type:** {strategy_type from taxonomy}
- **Tags:** [{tag1}, {tag2}, {tag3}]
- **What was tried:** {description of what was attempted}
- **Outcome:** {SUCCEEDED | FAILED | PARTIAL | ABANDONED | IN_PROGRESS}
- **Reasoning:** {why this outcome occurred, why abandoned or continuing}
- **Artifacts produced:** {list of files created/modified, or "None"}
- **Related files:** {markdown links to relevant artifacts}
- **Insight/Takeaway:** {REQUIRED -- what was learned, even from success}
```

**Insight/Takeaway is ALWAYS required, even for successes.** This is the core philosophy of the research journal: every research action teaches something. A successful proof attempt still has insight ("The key step was recognizing X", "Completeness of the space was essential because Y"). An agent MUST NOT skip this field or fill it with generic text.

## Entry Categories

Journal entries are organized into four categories, each with a sequential numbering prefix:

| Category | Prefix | Use For |
|----------|--------|---------|
| Proof Attempts | `PROOF-{NNN}` | Proof attempts, gap identifications, strategy changes |
| Literature Searches | `LIT-{NNN}` | Literature searches, paper findings, synthesis |
| Computations | `COMP-{NNN}` | Computations, verifications, counterexample searches |
| General Notes | `NOTE-{NNN}` | General observations, user notes, cross-problem insights |

Numbers are sequential within each category, zero-padded to three digits (PROOF-001, PROOF-002, LIT-001, etc.). When writing a new entry, the agent reads the existing entries in that category to determine the next number.

Entries are appended to the corresponding section in JOURNAL.md (Proof Attempts, Literature Searches, Computations, General Notes). Within each section, entries are chronological (newest at bottom).

## Frontmatter Update Protocol

When writing a journal entry, the agent MUST also update JOURNAL.md frontmatter:

1. **Increment `total_entries`** by 1
2. **Update `last_entry`** to the current ISO 8601 timestamp
3. **For proof-related entries (PROOF- category):** Add or update an entry in the `strategies_tried` array with these fields:

```yaml
strategies_tried:
  - id: "PROOF-001"
    strategy: "{strategy_type}"
    status: "{lowercase outcome: succeeded/failed/partial/abandoned/in-progress}"
    tags: ["{tag1}", "{tag2}", "{tag3}"]
```

If a PROOF entry updates an existing strategy (same `id`), update the `status` and `tags` in place rather than adding a duplicate.

The `strategies_tried` array is the machine-scannable index that enables dead-end detection. It must always be kept in sync with the PROOF- entries in the journal body.

## Strategy Type Taxonomy

All agents must use one of these 14 strategy types when writing journal entries. The strategy type determines how dead-end detection matches prior attempts.

| Strategy Type | Use For | Example |
|---|---|---|
| `direct` | Direct proof / construction | "Construct the inverse directly" |
| `contradiction` | Proof by contradiction | "Assume not invertible, derive contradiction" |
| `induction` | Any form of induction | "Induction on dimension n" |
| `contrapositive` | Proof by contrapositive | "Show if not B then not A" |
| `construction` | Explicit construction | "Construct a counterexample" |
| `estimation` | Bound/estimate argument | "Bound the operator norm" |
| `reduction` | Reduce to known result | "Reduce to finite-dimensional case" |
| `compactness` | Compactness argument | "Extract convergent subsequence" |
| `density` | Density/approximation argument | "Approximate by smooth functions" |
| `duality` | Duality argument | "Pass to dual space" |
| `computation` | Computational verification | "Compute for small cases" |
| `broad-survey` | Literature survey | "Search for related results" |
| `targeted-search` | Targeted literature search | "Find results on specific topic" |
| `gap-identification` | Identifying proof gaps | "Analyze where proof fails" |

If a strategy does not fit any of these types, the agent should use the closest match and note the distinction in the entry's "What was tried" field. The taxonomy may be extended in future phases.

## Dead-End Detection Protocol

Before attempting a new proof strategy, the agent MUST perform dead-end detection:

### Detection Flow

1. Read JOURNAL.md frontmatter `strategies_tried` array
2. Compare proposed strategy type and tags against prior entries
3. **Match criteria:** Same strategy type OR 2+ shared tags with any FAILED or ABANDONED entry
4. If match found:
   a. Read the matched entry's full body text from the journal
   b. Assess whether the current attempt is meaningfully different
   c. Check if conditions have changed since the prior attempt:
      - New literature found? (check LITERATURE.md dates vs prior attempt date)
      - New lemmas proved? (check PROOF.md for new completed steps)
      - New computations? (check COMPUTATION.md for new results)
   d. If conditions changed: use the **Changed Conditions** nudge
   e. If conditions unchanged: use the **Unchanged Conditions** nudge
5. If user proceeds after nudge: create a new journal entry noting the intentional re-attempt with a reference to the prior entry
6. If no match: proceed normally, create a journal entry for the new attempt

### Changed Conditions Nudge

When conditions have changed since a prior failed/abandoned attempt, proactively suggest retrying:

```
Previous attempt '{entry_title}' ({entry_id}) used {strategy_type}
and was {outcome} because: {reasoning}.

However, since then you have: {new_developments}.
This may address the original blocker. Worth retrying?
```

### Unchanged Conditions Nudge

When conditions have NOT changed, gently inform the user of the prior attempt:

```
Note: A similar approach was tried before -- see {entry_id}.
Strategy: {strategy_type}. Outcome: {outcome}.
Insight: {insight}.

Proceed with the new attempt anyway?
```

### Important Rules

- Dead-end detection INFORMS but never BLOCKS. The agent mentions prior attempts and asks "Proceed anyway?" but always allows the user to continue.
- The primary match criterion is strategy TYPE, not tags. Tags provide secondary confirmation.
- The agent reads the matched entry's full text before nudging -- it makes a judgment call about whether the similarity is meaningful, not just mechanical tag matching.
- If multiple prior entries match, surface the most recent and most relevant one.

## Outcome Values

Every journal entry must use one of these five outcome values:

| Outcome | Definition |
|---------|------------|
| `SUCCEEDED` | Goal fully achieved |
| `FAILED` | Goal not achieved, no path forward identified |
| `PARTIAL` | Some progress made, but incomplete |
| `ABANDONED` | Deliberately stopped in favor of a different approach |
| `IN_PROGRESS` | Currently active, not yet resolved |

When an IN_PROGRESS entry is later resolved, the agent updates the original entry's outcome and adds a new entry documenting the resolution.

## Writing Guidelines

- **Granularity:** Write entries at "significant action" granularity -- not every micro-step, but every meaningful research action (literature found, computation run, proof step attempted, gap identified, strategy change).
- **Titles:** Use specific, descriptive titles. Write "Contradiction via non-invertibility" not "Attempted proof". Write "Search for Neumann series convergence results" not "Literature search".
- **Tags:** Use mathematical concepts and techniques as tags, 3-5 per entry. Tags should be specific enough for meaningful matching (e.g., "operator-norm", "banach-space", "convergence") but not so specific that no two entries share tags.
- **Related files:** Use markdown links relative to the problem directory (e.g., `[PROOF.md](PROOF.md)`, `[LITERATURE.md](LITERATURE.md)`).
- **Insight/Takeaway:** Write 1-3 sentences capturing the key learning. Focus on what was discovered about the problem, the approach, or the mathematical structure -- not just restating the outcome.
