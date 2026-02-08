---
description: Show all available /math commands with status indicators
allowed-tools:
  - Read
---

# /math:help -- Command Reference

Display the complete list of `/math:*` commands with their status and descriptions.

## Process

Display the following command reference. Do not read any files -- this content is self-contained.

```
/math Command Reference
========================

Active Commands (Phases 1-3)
----------------------------

  /math:init       Initialize a math research project
                   Creates .math/ directory with notation presets,
                   problem template, and configuration.

  /math:problem    Submit a structured mathematical problem
                   Guided wizard collects problem statement, domain,
                   output type, and references step-by-step.

  /math:notation   View and configure notation profile
                   View, switch domain presets, add/edit symbols,
                   manage LaTeX packages, set presentation style.

  /math:search     Search mathematical literature
                   Search arXiv and Semantic Scholar for papers
                   relevant to your problem, with verified references
                   and synthesis.
                   Usage: /math:search (broad), /math:search "query" (focused)

  /math:help       Show this command reference

  /math:status     View research state dashboard
                   Shows problem status, notation config, and
                   contextual next steps.


Coming Soon
-----------

  /math:prove        Collaborate on proofs                 (Phase 4)
  /math:compute      Run computations                     (Phase 5)
  /math:orchestrate  Full research pipeline                (Phase 6)
  /math:write        Generate LaTeX output                 (Phase 7)


Confidence Tiers
----------------

  All mathematical output is tagged with confidence levels:

  [V]  Verified    -- Proven, cited, or computationally confirmed
  [S]  Suggested   -- Reasonable inference, not fully verified
  [~]  Speculative -- Exploratory, may be incorrect
  [NC] Notation Conflict -- Source uses different notation than profile


Getting Started
---------------

  1. Run /math:init to set up your project
  2. Run /math:problem to submit a problem
  3. Use /math:notation to customize your notation profile
  4. Run /math:status to see your research state
```
