# Phase 3: Literature Search Agent - Research

**Researched:** 2026-02-08
**Domain:** Academic literature search APIs (arXiv, Semantic Scholar), citation verification, literature synthesis in agent-based systems
**Confidence:** HIGH

## Summary

Phase 3 delivers a literature search agent that queries arXiv and Semantic Scholar, verifies all citations against source APIs, synthesizes connections between found papers and the user's problem, and persists results in LITERATURE.md. The technical domain spans two external REST APIs (arXiv Atom/XML, Semantic Scholar JSON), a verification protocol to prevent hallucinated references, and a synthesis step that connects found results to the user's PROBLEM.md.

The key insight is that within the Claude Code agent model, the literature search agent uses the **WebFetch tool** to query API endpoints directly -- no npm packages, no Python scripts, no external dependencies. The arXiv API returns Atom XML at a simple GET URL; the Semantic Scholar API returns JSON at a simple GET URL. WebFetch can fetch these URLs and Claude can parse the structured responses. The verification step (LIT-04) is architecturally significant: every reference the agent presents must be independently confirmed by fetching it from the source API by its identifier (arXiv ID or DOI). This prevents the well-documented problem where ~40% of AI-generated citations contain fabrications.

The literature search agent follows the established pattern from Phase 1/2: a slash command (`/math:search`) spawns a specialized agent (`agents/literature-search.md`) that reads PROBLEM.md for context, queries external APIs via WebFetch, writes results to LITERATURE.md, and logs the search in JOURNAL.md via the session-manager agent. The LITERATURE.md template and a `protocols/literature-protocol.md` (defining the verification workflow and LITERATURE.md schema) are the new infrastructure pieces.

**Primary recommendation:** Build a literature search agent that uses WebFetch to query arXiv (Atom XML) and Semantic Scholar (JSON REST API) with no external dependencies. Use a two-phase search pattern: Phase A queries both APIs with domain-informed search terms derived from PROBLEM.md; Phase B verifies every candidate reference by fetching it individually by its identifier from the source API. Only verified references appear as "confirmed" in LITERATURE.md. Unverifiable references are tagged as "unconfirmed" and never presented as established citations.

## Standard Stack

### Core

| Library/Tool | Version | Purpose | Why Standard |
|-------------|---------|---------|-------------|
| Claude Code WebFetch tool | latest | HTTP requests to arXiv and Semantic Scholar APIs | Native Claude Code tool; no external dependency needed; can fetch any URL and parse the response |
| arXiv API | v1 (Atom 1.0) | Search mathematical preprints by keyword, author, category | Free, no auth required, comprehensive math coverage, returns structured XML |
| Semantic Scholar Academic Graph API | v1 (graph/v1) | Search published papers with citation metrics and DOI/arXiv cross-refs | Free tier (1 RPS unauthenticated), JSON response, 200M+ papers, fieldsOfStudy filter for Mathematics |
| Markdown + YAML frontmatter | -- | LITERATURE.md document schema for persisting results | Established project pattern from Phase 1/2 |

### Supporting

| Library/Tool | Version | Purpose | When to Use |
|-------------|---------|---------|-------------|
| Semantic Scholar API key | -- | Higher rate limits (authenticated: 1 RPS dedicated vs shared pool) | Optional; needed only if unauthenticated rate limit is hit during heavy searches |
| arXiv category codes | -- | Domain-specific search filtering (e.g., `cat:math.FA` for Functional Analysis) | Every search -- map user's domain from PROBLEM.md to arXiv category codes |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| WebFetch for API calls | Bash + curl | Both work. WebFetch is the idiomatic Claude Code approach and avoids shell escaping issues with complex URLs. Bash + curl gives raw output but requires manual parsing. **Use WebFetch** -- it is the standard tool for HTTP requests in Claude Code agents. |
| Semantic Scholar for published papers | Google Scholar | Google Scholar has no public API. Scraping violates ToS. Semantic Scholar has an official free API with 200M+ papers. **Use Semantic Scholar.** |
| Two-API approach (arXiv + S2) | arXiv only | arXiv covers preprints but misses published-only papers. Semantic Scholar covers both but has weaker math-specific search. Together they provide comprehensive coverage. **Use both.** |
| WebFetch parsing of XML | Bash + xmllint/xmlstarlet | External tool dependency. Claude can parse XML from WebFetch responses directly. **Use WebFetch parsing.** |

**Installation:**
```bash
# No new dependencies. Phase 3 uses only Claude Code built-in tools (WebFetch, Read, Write).
# Optional: Semantic Scholar API key for higher rate limits (set in .math/config.json).
```

## Architecture Patterns

### Recommended File Structure (additions for Phase 3)

```
get-shit-done-math/
├── commands/
│   └── search.md                    # UPDATED: Replace "coming soon" with active search command
├── agents/
│   └── literature-search.md         # NEW: Literature search agent
├── protocols/
│   ├── confidence-tiers.md          # Existing (Phase 1)
│   ├── journal-protocol.md          # Existing (Phase 2)
│   └── literature-protocol.md       # NEW: Verification workflow, LITERATURE.md schema, search strategy
├── templates/
│   └── LITERATURE.md                # NEW: Literature results template for .math/problems/{slug}/
```

### Pattern 1: Two-Phase Search with Verification

**What:** Every literature search follows a two-phase pattern: (A) broad discovery via API queries, then (B) individual verification of each candidate by its identifier.

**When to use:** Every `/math:search` invocation.

**Why this pattern matters (LIT-04):** ~40% of AI-generated citations contain errors or fabrications. The verification phase eliminates this by requiring every reference to be independently confirmed through the source API. A reference that cannot be fetched by its arXiv ID or DOI is tagged "unconfirmed" and clearly flagged.

**Flow:**

```
Phase A: Discovery
1. Read PROBLEM.md for domain, tags, problem statement, known results, references
2. Generate search queries (3-5 queries per API):
   - Query 1: Direct problem keywords (from problem statement)
   - Query 2: Domain + specific mathematical concepts (from tags)
   - Query 3: Known results/theorems mentioned (from known results section)
   - Query 4: Author/reference-based (from references section, if any)
   - Query 5: Related category browse (from domain -> arXiv category mapping)
3. Execute queries against arXiv API and Semantic Scholar API via WebFetch
4. Collect candidate papers with metadata

Phase B: Verification
5. For each candidate paper:
   a. Extract identifier (arXiv ID or DOI)
   b. Fetch the paper individually from the source API:
      - arXiv: WebFetch `http://export.arxiv.org/api/query?id_list={arxiv_id}`
      - S2: WebFetch `https://api.semanticscholar.org/graph/v1/paper/ARXIV:{id}` or `DOI:{doi}`
   c. Compare returned metadata (title, authors, year) against discovery result
   d. If match: mark as CONFIRMED
   e. If mismatch or fetch fails: mark as UNCONFIRMED
6. Only CONFIRMED references enter the main LITERATURE.md results section
7. UNCONFIRMED references are logged separately with the reason for non-confirmation
```

### Pattern 2: Domain-to-Category Mapping

**What:** The user's mathematical domain (from PROBLEM.md) maps to arXiv category codes for targeted search filtering.

**When to use:** Constructing arXiv API queries.

**Mapping table:**

| Plugin Domain | Primary arXiv Categories | Secondary Categories |
|--------------|-------------------------|---------------------|
| `algebra` | math.RA, math.GR, math.AC | math.RT, math.CT |
| `analysis` | math.FA, math.CA, math.CV | math.AP, math.SP, math.OA |
| `topology` | math.AT, math.GN, math.GT | math.DG, math.KT |
| `number-theory` | math.NT | math.AG, math.CO |
| `combinatorics` | math.CO | math.PR, math.RT |
| `algebraic-geometry` | math.AG | math.AC, math.CT, math.KT |
| `differential-geometry` | math.DG | math.AP, math.SG, math.MG |
| `probability` | math.PR | math.FA, math.ST, math.DS |
| `logic` | math.LO | math.CT, math.GN |

**Example arXiv query for an analysis problem with tags `[banach-spaces, operator-norm, convergence]`:**
```
http://export.arxiv.org/api/query?search_query=(cat:math.FA+OR+cat:math.CA)+AND+(all:banach+AND+all:operator+AND+all:convergence)&start=0&max_results=15&sortBy=relevance
```

### Pattern 3: LITERATURE.md Document Schema

**What:** Persistent literature results stored as structured markdown with YAML frontmatter for machine parsing by downstream agents (proof, writing, orchestrator).

**When to use:** After every completed search.

**Schema:**

```markdown
---
problem: "{slug}"
total_papers: 0
last_search: ""
sources_queried: []
confirmed_count: 0
unconfirmed_count: 0
---

# Literature: {Problem Title}

## Search History

| Date | Query Summary | arXiv Results | S2 Results | New Papers |
|------|--------------|---------------|------------|------------|
| | | | | |

## Confirmed References

(No confirmed references yet)

## Synthesis

(No synthesis yet -- connections between papers and the problem will appear here after search)

## Unconfirmed References

(No unconfirmed references yet)
```

**Individual paper entry format (within Confirmed References):**

```markdown
### [REF-001] {Title}
- **Authors:** {Author1}, {Author2}, ...
- **Year:** {YYYY}
- **Source:** arXiv | Semantic Scholar | Both
- **arXiv ID:** {arXiv ID or "N/A"}
- **DOI:** {DOI or "N/A"}
- **Abstract:** {Full abstract text}
- **Relevance:** {1-3 sentence explanation of why this paper is relevant to the user's problem}
- **Key Results:** {Specific theorems, lemmas, or techniques from this paper that apply}
- **Confidence:** [V] (verified against source API)
- **Verified:** {ISO timestamp}
```

### Pattern 4: Agent Delegation (search command -> literature-search agent)

**What:** The `/math:search` command is a thin orchestrator that resolves the problem context and spawns the `literature-search` agent via Task tool. The agent handles all API interaction, verification, synthesis, and LITERATURE.md writing.

**When to use:** Every `/math:search` invocation.

**Why:** Consistent with the established pattern from Phase 1 (`/math:problem` -> `math-intake` agent). The search is a complex multi-step operation (query construction, API calls, verification, synthesis, writing) that benefits from a dedicated agent with full context.

**Command -> Agent data flow:**
```
/math:search command:
  1. Reads .math/config.json -> current_problem
  2. Validates .math/problems/{slug}/ exists
  3. Validates PROBLEM.md has status != "draft"
  4. Passes to literature-search agent:
     - Problem directory path
     - Problem slug
     - User's optional search query (if provided as argument)
     - @protocols/literature-protocol.md
     - @protocols/confidence-tiers.md
     - @protocols/journal-protocol.md

literature-search agent:
  1. Reads PROBLEM.md for domain, tags, known results, references
  2. Reads NOTATION.md for notation context
  3. Reads existing LITERATURE.md (if any, to avoid duplicate searches)
  4. Constructs search queries
  5. Executes Phase A (discovery) via WebFetch
  6. Executes Phase B (verification) via WebFetch
  7. Synthesizes connections (Phase C)
  8. Writes/updates LITERATURE.md
  9. Calls session-manager agent to write LIT- journal entry
  10. Updates STATE.md
```

### Pattern 5: Synthesis as Structured Analysis

**What:** After verification, the agent synthesizes connections between found papers and the user's problem. This is NOT a general summary -- it identifies specific theorems, techniques, and results from papers that apply to the user's problem.

**When to use:** After every completed search, as the final section of LITERATURE.md.

**Example synthesis:**

```markdown
## Synthesis

### Applicable Theorems
- [V] **Theorem 3.3 from [REF-001] (Rudin):** Hahn-Banach extension theorem. Directly applicable to extending the functional in Step 2 of the problem.
- [V] **Theorem 2.1 from [REF-003] (Conway):** Neumann series convergence for bounded operators with ||T|| < 1. This IS the result the user is trying to prove -- can be used as a reference proof strategy.

### Technique Connections
- [S] The approach in [REF-002] (Kato, Section IV.1) uses a spectral theory argument that could provide an alternative proof path via the resolvent. This connects to the operator norm bound in the problem.

### Gaps in Literature
- No papers found addressing the specific case where X is non-separable. If the user's Banach space is non-separable, additional work may be needed beyond what the literature provides.

### Suggested Reading Order
1. [REF-003] Conway -- closest to the problem, provides direct proof strategy
2. [REF-001] Rudin -- foundational result used in the proof
3. [REF-002] Kato -- alternative approach via spectral theory
```

### Anti-Patterns to Avoid

- **Presenting unverified references as confirmed:** NEVER show a reference without the verification step. Even if Claude "knows" a paper exists from training data, it must be fetched from the API to confirm. This is the core of LIT-04.

- **Single monolithic search query:** Don't send one query and hope for the best. Use 3-5 targeted queries derived from different aspects of the problem (keywords, domain categories, known authors, cited references). Different queries surface different relevant papers.

- **Ignoring existing LITERATURE.md:** When the user runs `/math:search` again, the agent must read existing results to avoid duplicate work. New searches should complement, not replace, prior results.

- **Synthesis that just summarizes abstracts:** The synthesis section must connect papers TO THE USER'S PROBLEM. "This paper studies Banach spaces" is useless. "Theorem 3.3 from this paper provides the exact extension result needed for Step 2 of your proof" is valuable.

- **Fetching too many papers in one session:** Rate limits exist. arXiv recommends 3-second delays between requests. Semantic Scholar allows 1 RPS (authenticated) or shared pool (unauthenticated). Design for 10-20 candidate papers per search, not 100.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| HTTP requests | Custom HTTP client or curl scripts | WebFetch tool | Native Claude Code tool; handles URL fetching and content extraction |
| XML parsing (arXiv) | xmllint, xmlstarlet, or custom XML parser | Claude's text parsing of WebFetch response | Claude can extract structured data from XML text returned by WebFetch; Atom XML is simple and well-structured |
| JSON parsing (S2) | jq or custom JSON parser | Claude's native JSON understanding of WebFetch response | Claude natively understands JSON; Semantic Scholar returns clean JSON |
| Citation databases | Custom reference database | LITERATURE.md with structured entries | Markdown file is readable, versionable, and parseable by downstream agents. No database needed. |
| Deduplication | Custom hash/fingerprint matching | Agent comparison of title + authors + year | Agent can detect duplicates across arXiv and S2 results using simple metadata comparison; no algorithm needed |
| Rate limiting | Custom rate limiter | Sequential WebFetch calls with awareness of limits | WebFetch calls are naturally sequential; agent can space them. arXiv: 3s between calls. S2: 1 RPS. |
| BibTeX generation | Custom BibTeX formatter | Agent text formatting | BibTeX is a simple text format; agent can generate entries from structured metadata. Phase 7 (LaTeX) will consume these. |

**Key insight:** Phase 3 has zero external dependencies. The entire literature search pipeline runs through WebFetch (HTTP), Claude's text parsing (XML/JSON), and Write (LITERATURE.md). The intelligence is in the agent prompt (query construction, verification logic, synthesis), not in code.

## Common Pitfalls

### Pitfall 1: WebFetch URL Construction with Special Characters

**What goes wrong:** arXiv API queries contain URL-encoded characters (spaces as `+`, parentheses as `%28`/`%29`, colons in field prefixes). Malformed URLs return empty results or errors.
**Why it happens:** Agent constructs query strings without proper URL encoding.
**How to avoid:** The literature protocol must include exact URL templates with encoding rules. Agent instructions should show concrete URL examples for each query type. Field prefixes use colons (e.g., `cat:math.FA`) which must be URL-encoded as `cat%3Amath.FA` in the full query string -- though in practice arXiv is forgiving about this within the `search_query` parameter value.
**Warning signs:** Empty result sets from queries that should match papers, HTTP 400 errors from arXiv.

### Pitfall 2: arXiv XML Response Parsing Failures

**What goes wrong:** WebFetch returns the Atom XML response and Claude fails to extract all fields consistently, especially author lists (which are nested XML elements) or multiple categories.
**Why it happens:** Atom XML has nested structures (`<author><name>...</name></author>`) that are less intuitive to parse than flat JSON.
**How to avoid:** The literature protocol should define exactly which XML elements map to which LITERATURE.md fields. Provide worked parsing examples in the agent instructions. Key mappings: `<title>` -> title, `<author><name>` -> authors (iterate), `<summary>` -> abstract, `<id>` -> arXiv URL (extract ID from URL), `<arxiv:primary_category term="">` -> category, `<published>` -> year, `<arxiv:doi>` -> DOI if present. Test with WebFetch prompt that specifically asks for structured data extraction.
**Warning signs:** Missing authors, incomplete abstracts, or "N/A" for fields that should have values.

### Pitfall 3: Semantic Scholar Rate Limiting

**What goes wrong:** Unauthenticated requests hit the shared rate limit pool, causing HTTP 429 responses during verification phase when fetching individual papers.
**Why it happens:** Verification requires one API call per candidate paper. With 15-20 candidates, that is 15-20 sequential requests. At shared pool rates, this may exceed the limit.
**How to avoid:** Use the batch endpoint (`POST /paper/batch`) to verify multiple papers in a single request (up to 500 papers). This counts as one API call instead of 20. If batch is not feasible, space individual requests and handle 429 gracefully (wait and retry). Document the optional API key configuration in `.math/config.json` for users who need higher limits.
**Warning signs:** HTTP 429 responses, incomplete verification results, search hanging.

### Pitfall 4: Hallucinated References Sneaking Through

**What goes wrong:** Claude "knows" about a paper from training data and includes it in results without API verification. The paper may not exist, or the metadata (title, authors, year) may be wrong.
**Why it happens:** The agent is prompted to find relevant papers, and Claude's training data contains extensive mathematical knowledge. The temptation is to "fill in" results.
**How to avoid:** The verification protocol is the defense. The literature-protocol.md MUST enforce: (1) Every reference in LITERATURE.md must have a `Verified:` timestamp, (2) The verification entry must include the API URL used and the response status, (3) No reference enters the "Confirmed References" section without successful verification. Agent instructions must explicitly state: "Do NOT include papers from your training data that you cannot verify via the API. If you believe a paper exists but cannot fetch it, place it in the Unconfirmed References section with an explanation."
**Warning signs:** References without arXiv IDs or DOIs, references with vague metadata ("Smith et al., around 2018"), references that don't appear in the API.

### Pitfall 5: Poor Query Construction from Problem Statement

**What goes wrong:** The agent sends the problem statement verbatim as a search query. arXiv and Semantic Scholar return irrelevant results because the query is too specific (LaTeX notation) or too broad (full paragraph).
**Why it happens:** Mathematical problem statements contain LaTeX, notation, and lengthy descriptions that don't work well as API queries.
**How to avoid:** The agent must extract search-friendly terms from the problem statement. Strategy: (1) Use PROBLEM.md tags and domain for category filtering, (2) Extract key mathematical concepts (nouns, theorem names, technique names) from the problem statement, stripping LaTeX, (3) Use known results and references as additional query seeds, (4) Generate 3-5 distinct queries targeting different aspects. The literature protocol should include query construction examples for common math domains.
**Warning signs:** Zero results from arXiv, or results entirely outside the user's domain.

### Pitfall 6: Synthesis Disconnected from Problem

**What goes wrong:** The synthesis section summarizes papers individually rather than connecting them to the user's specific problem.
**Why it happens:** Summarizing abstracts is easier than analyzing how specific results apply to a specific problem.
**How to avoid:** The synthesis prompt must receive PROBLEM.md context and explicitly ask: "How does Theorem X from this paper help with the user's problem?" The synthesis section should reference specific steps, hypotheses, or gaps in the user's problem. Use the confidence tier markers: [V] for connections to verified theorems, [S] for agent-derived connections, [~] for speculative connections.
**Warning signs:** Generic paper summaries, no reference to the user's problem statement, no confidence tier markers in synthesis.

## Code Examples

### arXiv API Query via WebFetch

```
WebFetch URL: http://export.arxiv.org/api/query?search_query=(cat:math.FA+OR+cat:math.CA)+AND+(all:banach+AND+all:operator+AND+all:convergence)&start=0&max_results=10&sortBy=relevance
WebFetch Prompt: "Extract all paper entries from this Atom XML feed. For each entry, return: title, all author names, published date (year only), arXiv ID (the numeric part from the <id> URL, e.g., '2301.12345'), abstract (from <summary>), primary category, and DOI if present in <arxiv:doi>. Format as a numbered list."
```

**arXiv API key parameters:**
- Base URL: `http://export.arxiv.org/api/query`
- `search_query`: Boolean combination of field prefixes. Fields: `ti:` (title), `au:` (author), `abs:` (abstract), `cat:` (category), `all:` (all fields). Operators: `AND`, `OR`, `ANDNOT`. Grouping: `%28`/`%29` for parentheses.
- `id_list`: Comma-separated arXiv IDs for verification (e.g., `2301.12345,2302.67890`)
- `start`: Pagination offset (default 0)
- `max_results`: Results per page (default 10, max 2000)
- `sortBy`: `relevance` (default), `lastUpdatedDate`, `submittedDate`
- `sortOrder`: `ascending`, `descending`
- Rate limit: 3-second delay between calls recommended

### arXiv Verification via WebFetch

```
WebFetch URL: http://export.arxiv.org/api/query?id_list=2301.12345
WebFetch Prompt: "Extract the paper title, all author names, published year, and abstract from this Atom XML entry. If no entry is found, state 'No paper found for this ID'."
```

### Semantic Scholar API Query via WebFetch

```
WebFetch URL: https://api.semanticscholar.org/graph/v1/paper/search?query=Neumann+series+Banach+space+operator&fields=paperId,title,authors,year,abstract,externalIds,citationCount,fieldsOfStudy,url&limit=10&fieldsOfStudy=Mathematics
WebFetch Prompt: "Parse this JSON response. For each paper in the 'data' array, extract: paperId, title, all author names, year, abstract, externalIds (arXiv ID and DOI if present), citationCount, and fieldsOfStudy. Format as a numbered list."
```

**Semantic Scholar key parameters:**
- Base URL: `https://api.semanticscholar.org/graph/v1`
- Paper search: `GET /paper/search?query={terms}&fields={fields}&limit={n}`
- Paper lookup: `GET /paper/{paper_id}` where paper_id can be `ARXIV:{id}`, `DOI:{doi}`, or S2 paperId
- Batch lookup: `POST /paper/batch` with body `{"ids": ["ARXIV:2301.12345", "DOI:10.1xxx/..."]}` (max 500)
- Key fields: `paperId`, `title`, `authors`, `year`, `abstract`, `externalIds`, `citationCount`, `fieldsOfStudy`, `url`, `openAccessPdf`
- `fieldsOfStudy` filter: `Mathematics` (filters to math papers)
- Rate limit: Shared pool unauthenticated (1000 RPS total shared); authenticated: 1 RPS dedicated
- Auth header: `x-api-key: {key}` (optional, for higher limits)
- Pagination: `offset` + `limit` (relevance search), `token` (bulk search)

### Semantic Scholar Verification via WebFetch

```
WebFetch URL: https://api.semanticscholar.org/graph/v1/paper/ARXIV:2301.12345?fields=paperId,title,authors,year,abstract,externalIds
WebFetch Prompt: "Parse this JSON response. Extract: paperId, title, all author names, year, abstract, and externalIds (DOI and arXiv ID). If the response is an error (404 or empty), state 'Paper not found'."
```

### Semantic Scholar Batch Verification

```
Use Bash + curl for POST request (WebFetch is GET-only):

curl -s -X POST "https://api.semanticscholar.org/graph/v1/paper/batch" \
  -H "Content-Type: application/json" \
  -d '{"ids":["ARXIV:2301.12345","DOI:10.1016/j.jfa.2020.108780"]}' \
  --data-urlencode "fields=paperId,title,authors,year,abstract,externalIds"
```

Note: If POST requests are needed for batch verification and WebFetch only supports GET, the agent should use the Bash tool with curl. This is the one case where Bash is preferred over WebFetch for API calls.

### LITERATURE.md Template

```markdown
---
problem: "{PROBLEM_SLUG}"
total_papers: 0
confirmed_count: 0
unconfirmed_count: 0
last_search: ""
sources_queried: []
---

# Literature: {PROBLEM_TITLE}

## Search History

| Date | Query Summary | arXiv Results | S2 Results | New Confirmed |
|------|--------------|---------------|------------|---------------|

## Confirmed References

(No confirmed references yet)

## Synthesis

(No synthesis yet)

## Unconfirmed References

(No unconfirmed references yet)
```

### Journal Entry for Literature Search (written via session-manager)

```
category: LIT
title: "{Descriptive title, e.g., 'Broad survey for Neumann series convergence results'}"
agent_name: literature-search
strategy_type: broad-survey | targeted-search
tags: ["{concept1}", "{concept2}", "{concept3}"]
what_tried: "Searched arXiv (cat:math.FA) and Semantic Scholar (Mathematics) for: {query terms}. {N} candidates found, {M} confirmed."
outcome: SUCCEEDED | PARTIAL | FAILED
reasoning: "{Why this outcome -- e.g., 'Found 3 directly relevant papers and 2 tangentially related'}"
artifacts: "LITERATURE.md updated with {N} new confirmed references"
related_files: "[LITERATURE.md](LITERATURE.md)"
insight: "{What was learned about the literature landscape -- e.g., 'This is a well-covered textbook result. Three standard references exist. No recent preprints address novel aspects.'}"
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| AI generates citations from memory | API-verified citations with confirmation step | 2024-2025 (post-hallucination awareness) | Eliminates ~40% fabrication rate in AI-generated references |
| Google Scholar scraping | Semantic Scholar public API | 2023+ (S2 API matured) | Legal, reliable, structured JSON, 200M+ papers |
| Single search source | Multi-source (arXiv + Semantic Scholar) | Current best practice | Preprints (arXiv) + published papers (S2) = comprehensive coverage |
| Flat citation lists | Structured literature with synthesis | Emerging in AI research tools | Connections to user's specific problem, not just paper summaries |

**Deprecated/outdated:**
- Google Scholar API: Never existed as a public API. Scraping violates ToS. Do not attempt.
- arXiv OAI-PMH: Bulk metadata harvesting protocol. Overkill for targeted search; use the simpler query API.
- MathSciNet/zbMATH: Paywalled. Deferred to v2 (LIT-08) per REQUIREMENTS.md.

## Open Questions

1. **WebFetch behavior with API XML/JSON responses**
   - What we know: WebFetch fetches URL content and processes it with a prompt. It converts HTML to markdown. For API endpoints returning XML or JSON, the behavior depends on Content-Type handling.
   - What's unclear: Whether WebFetch faithfully preserves XML/JSON structure or attempts HTML-to-markdown conversion that could mangle structured data. arXiv returns `Content-Type: application/atom+xml`; Semantic Scholar returns `Content-Type: application/json`.
   - Recommendation: Test WebFetch with both APIs during early implementation. If WebFetch mangles structured responses, fall back to `Bash + curl` for raw API access and have the agent parse the raw text. This is the primary technical risk for Phase 3.

2. **Semantic Scholar batch endpoint via POST**
   - What we know: WebFetch likely only supports GET requests. The S2 batch endpoint uses POST.
   - What's unclear: Whether there is a GET-based alternative for batch verification, or whether Bash + curl is needed.
   - Recommendation: Use individual GET requests for small batches (< 10 papers). For larger verification sets, use `Bash + curl` for the POST batch endpoint. The agent should have both strategies available.

3. **API rate limit handling across multiple searches**
   - What we know: arXiv recommends 3s between calls. S2 allows 1 RPS (authenticated). A typical search involves 3-5 discovery queries + 10-20 verification calls = 15-25 total API requests per search.
   - What's unclear: Whether Claude Code's WebFetch has any built-in request timing, or whether the agent must manually manage delays.
   - Recommendation: For arXiv, batch verification using `id_list` parameter (up to 2000 IDs in one call) eliminates the need for per-paper verification calls. For S2, use the batch endpoint via curl. This reduces total API calls to approximately: 3-5 discovery + 1-2 verification = 5-7 total calls per search, well within rate limits.

4. **Optional Semantic Scholar API key storage**
   - What we know: S2 API key provides dedicated 1 RPS rate limit vs shared unauthenticated pool.
   - What's unclear: Where to store the key. Options: `.math/config.json` (project-level, checked into git -- bad for secrets), environment variable, or a separate `.math/.env` file.
   - Recommendation: Store in `.math/config.json` as `"semantic_scholar_api_key": ""` (empty by default). Document that this is optional and the key should not be committed to public repos. Alternative: read from `SEMANTIC_SCHOLAR_API_KEY` environment variable. The agent should check both locations, preferring env var.

## Sources

### Primary (HIGH confidence)
- [arXiv API User's Manual](https://info.arxiv.org/help/api/user-manual.html) -- Complete query syntax, field prefixes, Boolean operators, Atom XML response schema, pagination, sort options, rate limits, error handling
- [arXiv API Basics](https://info.arxiv.org/help/api/basics.html) -- Base URL, parameter overview, example queries
- [arXiv Mathematics Category Taxonomy](https://arxiv.org/archive/math/) -- Complete list of 32 math.XX category codes with descriptions
- [Semantic Scholar API Tutorial](https://www.semanticscholar.org/product/api/tutorial) -- Base URLs, endpoint documentation, rate limits, authentication, pagination methods
- [Semantic Scholar API Gist (community reference)](https://gist.github.com/alexandreteles/c8bc00830e97eefa961e26c49aa666e7) -- Complete field list, endpoint URLs, paper ID formats, batch endpoints, rate limit details
- Phase 1/2 codebase (`/Users/matteo/Desktop/Coding/get-shit-done-math/`) -- Established patterns: agent delegation, command structure, protocol references, document schemas, session-manager integration

### Secondary (MEDIUM confidence)
- [AI Hallucinations in Research (Enago Academy)](https://www.enago.com/academy/ai-hallucinations-research-citations/) -- ~40% AI citation error rate, verification best practices
- [Claude Code WebFetch documentation](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-fetch-tool) -- WebFetch tool capabilities, prompt-based extraction, URL restrictions
- [Inside Claude Code's Web Tools (Mikhail Shilkov)](https://mikhail.io/2025/10/claude-code-web-tools/) -- WebFetch implementation details, local Axios-based fetching

### Tertiary (LOW confidence)
- WebFetch XML/JSON handling behavior -- Not directly tested; based on documentation and community reports. Flagged for validation during implementation.
- Semantic Scholar fieldsOfStudy "Mathematics" filter effectiveness -- Not tested with mathematical queries specifically. Filter may be broad or narrow depending on S2's classification. Flagged for validation.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- arXiv and Semantic Scholar APIs are well-documented, stable, and widely used. WebFetch is a native Claude Code tool.
- Architecture: HIGH -- Follows established Phase 1/2 patterns (agent delegation, protocol references, structured markdown). LITERATURE.md schema is a natural extension.
- API integration: HIGH -- Both APIs are REST/GET with simple query parameters. Response formats (Atom XML, JSON) are well-documented.
- Verification protocol: HIGH -- Two-phase search-then-verify is a proven pattern. arXiv `id_list` and S2 paper lookup endpoints support individual verification.
- Synthesis quality: MEDIUM -- Synthesis depends on agent prompt quality. The pattern is well-defined but actual output quality depends on Claude's mathematical understanding and PROBLEM.md context.
- WebFetch XML/JSON handling: MEDIUM -- WebFetch is documented for HTML; behavior with API XML/JSON responses needs validation. Bash + curl is the fallback.
- Rate limit management: MEDIUM -- Theoretical analysis suggests 5-7 API calls per search is within limits, but real-world conditions (shared pool, server load) may vary.

**Research date:** 2026-02-08
**Valid until:** 60 days (APIs are stable; WebFetch behavior may evolve with Claude Code updates)
