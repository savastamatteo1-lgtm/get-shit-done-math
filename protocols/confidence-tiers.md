# Confidence Tier Protocol

All agents in the math plugin MUST apply confidence tier markers to every claim, step, and result they produce. This protocol is the single source of truth for tier definitions and usage.

## Tier Definitions

### VERIFIED [V]

Cited from published literature: a known theorem, published result, or established fact.

**Requirements:**
- Paper or textbook citation (author, title, year minimum)
- Theorem, lemma, or proposition number when available
- Exact statement as published (paraphrasing must be noted)

**Example:**
```
[V] By the Hahn-Banach theorem (Rudin, Functional Analysis, Thm 3.3),
every bounded linear functional on a subspace extends to the full space
with the same norm.
```

### SUGGESTED [S]

Derived by the system with explicit justification from verified premises. The deduction chain is shown and each premise is cited.

**Requirements:**
- Cite which verified facts are used (by their [V] references)
- Show the deduction chain or reasoning steps
- Each step must follow logically from stated premises

**Example:**
```
[S] From [V] Theorem 3.3 and [V] Lemma 2.1, it follows by direct
substitution that the operator norm is bounded above by M. The bound
is tight because the supremum is attained on the unit ball.
```

### SPECULATIVE [~]

System conjecture, heuristic reasoning, creative leap, analogy, or unverified claim. This includes any step that cannot be fully justified from verified premises.

**Requirements:**
- State why this is speculative (what is missing or assumed)
- State what would verify or refute it

**Example:**
```
[~] The pattern suggests this extends to the infinite-dimensional case,
though no proof is known. Verification: check against counterexample
families in [Chen 2020].
```

## Active Warnings

When presenting content that contains speculative steps, agents MUST display a warning block at the top of the output.

**Format:**
```
WARNING: This [proof/analysis] contains N speculative step(s) that need verification.

Speculative steps:
- Step K: [brief reason it is speculative]
- Step M: [brief reason it is speculative]
```

**Rules:**
- Count all [~] markers in the output
- If count > 0, display the warning before any content
- List each speculative step with its position and a one-line reason
- Any proof with speculative steps is NOT considered complete

## Manual Override

Users can promote or demote the confidence tier of any claim. Overridden markers use an asterisk to indicate manual intervention.

**Override markers:**
- `[V*]` -- Manually promoted to VERIFIED (original tier was [S] or [~])
- `[S*]` -- Manually set to SUGGESTED (original tier was [V] or [~])
- `[~*]` -- Manually demoted to SPECULATIVE (original tier was [V] or [S])

**Note format:** When a tier is overridden, append:
```
Tier manually set by user (original: [tier])
```

**Rules:**
- Overrides persist across sessions (stored in problem state)
- Agents must respect overrides -- do not re-classify overridden claims
- Override history is tracked for audit purposes

## Usage in Agent Output

### Inline Markers

Place the tier marker at the start of each claim or step:

```
[V] The Banach fixed-point theorem guarantees a unique fixed point.
[S] Applying this to our operator T, the fixed point is the solution.
[~] The convergence rate appears to be geometric with ratio 0.5.
```

### Summary Block

At the top of any proof or analysis, display a confidence summary:

```
Confidence: X verified, Y suggested, Z speculative
```

This gives the reader an immediate sense of how much of the work is established versus provisional.

### Warning Threshold

Any output containing one or more speculative [~] steps triggers the Active Warning (see above). There is no "acceptable" number of speculative steps that bypasses the warning.

## Notation Conflict Interaction

When a cited source uses different notation than the user's notation profile (defined in NOTATION.md), agents MUST flag the notation translation.

**Compound marker:** `[V][NC]` means "verified claim, but notation was translated from the source's convention to the user's convention."

**Rules:**
- The [NC] flag appears alongside any tier marker: `[V][NC]`, `[S][NC]`, `[~][NC]`
- The details of the notation translation are logged in the Notation Conflicts Log section of NOTATION.md
- The translated notation must be mathematically equivalent to the original

**Example:**
```
[V][NC] The spectral radius rho(T) < 1 implies convergence.
(Source uses r(T); translated to rho(T) per notation profile.)
```

## Design Rationale

Plain text markers [V], [S], [~] are used instead of emoji or color codes because they are:
- Universally readable in any terminal, editor, or renderer
- Greppable (e.g., `grep "\[~\]"` finds all speculative claims)
- Survive copy-paste across platforms
- Compatible with LaTeX output (can be mapped to margin notes or annotations)
