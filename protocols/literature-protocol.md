# Literature Protocol

All agents that perform literature searches or consume literature results MUST follow this protocol. It defines the two-phase search-then-verify workflow, the LITERATURE.md document schema, domain-to-arXiv-category mappings, API endpoint references, query construction rules, and anti-hallucination safeguards. This protocol is the single source of truth for literature operations.

## 1. Purpose & Scope

This protocol governs all literature search operations in the math plugin. It applies whenever an agent:
- Searches for mathematical papers related to a problem
- Verifies paper metadata against source APIs
- Synthesizes connections between papers and the user's problem
- Writes or updates a LITERATURE.md document

**Referenced by:**
- Literature-search agent (primary consumer -- follows this protocol for all operations)
- Session-manager agent (writes LIT- category journal entries after searches)
- Proof agent (reads LITERATURE.md to find applicable theorems)
- Writing agent (reads LITERATURE.md for bibliography generation)

## 2. Two-Phase Search Workflow

Every literature search follows three phases in strict order: Discovery, Verification, Synthesis. No phase may be skipped.

### Phase A: Discovery

Query arXiv and Semantic Scholar to collect candidate papers.

**Steps:**
1. Read the problem's PROBLEM.md to extract: domain, tags, known results, references, mathematical concepts
2. Construct 3-5 distinct queries per API using the Query Construction Guidelines (Section 7)
3. Execute queries against both arXiv and Semantic Scholar
4. Collect candidate papers with metadata: title, authors, year, abstract, identifiers (arXiv ID, DOI, S2 paperId)
5. Deduplicate across sources: compare title + authors + year (case-insensitive title, last-name match for authors)
6. Record all queries in the Search History table

**Query generation targets** (generate 3-5 from these categories):
1. Direct problem keywords extracted from PROBLEM.md statement
2. Domain + mathematical concepts derived from PROBLEM.md tags
3. Known results or theorem names mentioned in PROBLEM.md Known Results section
4. Author or reference-based queries from PROBLEM.md References section
5. Related category browse using the domain-to-arXiv-category mapping (Section 5)

### Phase B: Verification

Verify every candidate paper against the source API individually. This is the anti-hallucination gate.

**Steps:**
1. For arXiv candidates: use the `id_list` parameter to batch-verify all arXiv IDs in a single call (see Section 6)
2. For Semantic Scholar candidates: use individual paper lookup by `ARXIV:{id}` or `DOI:{doi}` (see Section 6)
3. Compare returned metadata against discovery result:
   - Title must substantially match (minor formatting differences allowed)
   - At least one author last name must match
   - Year must match (or be within 1 year for preprint/publication date differences)
4. If metadata matches: mark as **CONFIRMED** with `Verified:` timestamp and API endpoint URL used
5. If metadata does NOT match or paper not found: mark as **UNCONFIRMED** with explanation of the discrepancy
6. If API returns an error (timeout, rate limit): retry once after waiting, then mark as **UNCONFIRMED** with reason "API error -- verification pending"

### Phase C: Synthesis

After verification, synthesize connections between confirmed papers and the user's problem. Use confidence tier markers from the [Confidence Tier Protocol](confidence-tiers.md).

**Steps:**
1. For each confirmed paper, identify specific theorems, techniques, and results that apply to the user's problem
2. Structure the synthesis into four subsections:
   - **Applicable Theorems:** Named theorems from confirmed papers that directly apply. Mark [V] for cited theorem statements.
   - **Technique Connections:** Methods or approaches from papers that could inform the user's proof strategy. Mark [S] for agent-derived connections with cited premises.
   - **Gaps in Literature:** What the user's problem needs that was NOT found in the search. Mark [~] for speculative assessments.
   - **Suggested Reading Order:** Recommended sequence for the user to read the confirmed papers, based on dependency and relevance.
3. Every claim in the synthesis MUST carry a confidence tier marker: [V], [S], or [~]

## 3. Anti-Hallucination Rules (LIT-04)

These rules are non-negotiable. They exist because language models can fabricate plausible-sounding paper titles, author names, and results. Every rule MUST be followed on every search.

1. **NEVER include papers from training data that cannot be verified via the API.** If the agent "knows" a paper exists but the API does not return it, the paper goes in Unconfirmed References with explanation.

2. **Every reference in LITERATURE.md MUST have a `Verified:` timestamp.** A reference without a verification timestamp is invalid and must not be presented as confirmed.

3. **No reference enters "Confirmed References" without successful API verification.** The Phase B verification step is mandatory -- there are no exceptions, even for "well-known" papers.

4. **If the agent believes a paper exists but cannot fetch it, place it in "Unconfirmed References" with explanation.** Example reasons: "arXiv API returned no results for this ID", "Title found in training data but not verified via Semantic Scholar", "API timeout -- verification pending".

5. **Unconfirmed references are clearly labeled and NEVER presented as established citations.** Downstream agents (proof, writing) MUST NOT cite unconfirmed references as [V] verified sources.

6. **Verification entries must include the API endpoint URL used.** This creates an audit trail: anyone can re-run the verification by visiting the URL.

## 4. LITERATURE.md Document Schema

Every problem's LITERATURE.md follows this exact schema. The template is at `templates/LITERATURE.md`.

### YAML Frontmatter

```yaml
---
problem: "{PROBLEM_SLUG}"
total_papers: 0
confirmed_count: 0
unconfirmed_count: 0
last_search: ""
sources_queried: []
---
```

| Field | Type | Description |
|-------|------|-------------|
| `problem` | string | The problem slug matching the directory name |
| `total_papers` | integer | Total papers across confirmed + unconfirmed |
| `confirmed_count` | integer | Number of confirmed (verified) references |
| `unconfirmed_count` | integer | Number of unconfirmed references |
| `last_search` | string | ISO 8601 timestamp of most recent search |
| `sources_queried` | array | List of sources queried (e.g., `["arxiv", "semantic_scholar"]`) |

### Section Structure

The document body has four sections in this order:

1. **Search History** -- table tracking all searches performed
2. **Confirmed References** -- verified papers with full metadata
3. **Synthesis** -- connections between papers and the user's problem
4. **Unconfirmed References** -- papers that could not be verified

### Search History Table

```markdown
## Search History

| Date | Query Summary | arXiv Results | S2 Results | New Confirmed |
|------|---------------|---------------|------------|---------------|
| 2026-02-10 | "operator norm" AND "Banach space" | 8 | 12 | 5 |
```

### Confirmed Reference Entry Format

Each confirmed reference follows this format:

```markdown
### REF-001: {Paper Title}
- **Authors:** {Author1, Author2, ...}
- **Year:** {YYYY}
- **Source:** {arXiv | Semantic Scholar | both}
- **arXiv ID:** {XXXX.XXXXX} (if available)
- **DOI:** {doi} (if available)
- **Abstract:** {full abstract text}
- **Relevance:** {1-3 sentences explaining why this paper matters for the user's problem}
- **Key Results:** {specific theorems, lemmas, or techniques from this paper}
- **Confidence:** {[V] for cited facts, [S] for derived relevance, [~] for speculative connections}
- **Verified:** {ISO timestamp} via {API endpoint URL}
```

### Unconfirmed Reference Entry Format

```markdown
### UREF-001: {Paper Title (as believed)}
- **Authors:** {Author(s) as believed}
- **Year:** {YYYY if known}
- **Source:** {where the agent encountered this reference}
- **Reason:** {why verification failed -- e.g., "arXiv API returned no results for ID 2301.12345", "Title not found in Semantic Scholar"}
- **Relevance:** {why the agent believes this paper is relevant}
- **Status:** Unconfirmed -- do not cite as established reference
```

### Numbering Rules

- REF-NNN for confirmed references: REF-001, REF-002, etc.
- UREF-NNN for unconfirmed references: UREF-001, UREF-002, etc.
- Numbering is sequential and never resets, even across multiple searches
- When adding new references to an existing LITERATURE.md, continue from the last number

## 5. Domain-to-arXiv-Category Mapping

Use this mapping to target queries to the correct arXiv categories. The `domain` field from PROBLEM.md determines the primary categories. Secondary categories are included when queries are broad or cross-domain.

| Domain | Primary Categories | Secondary Categories |
|--------|-------------------|---------------------|
| `algebra` | math.RA, math.GR, math.AC | math.RT, math.CT |
| `analysis` | math.FA, math.CA, math.CV | math.AP, math.SP, math.OA |
| `topology` | math.AT, math.GN, math.GT | math.DG, math.KT |
| `number-theory` | math.NT | math.AG, math.CO |
| `combinatorics` | math.CO | math.PR, math.RT |
| `algebraic-geometry` | math.AG | math.AC, math.CT, math.KT |
| `differential-geometry` | math.DG | math.AP, math.SG, math.MG |
| `probability` | math.PR | math.FA, math.ST, math.DS |
| `logic` | math.LO | math.CT, math.GN |

**Category code reference:**
- math.RA = Rings and Algebras, math.GR = Group Theory, math.AC = Commutative Algebra
- math.RT = Representation Theory, math.CT = Category Theory
- math.FA = Functional Analysis, math.CA = Classical Analysis, math.CV = Complex Variables
- math.AP = Analysis of PDEs, math.SP = Spectral Theory, math.OA = Operator Algebras
- math.AT = Algebraic Topology, math.GN = General Topology, math.GT = Geometric Topology
- math.DG = Differential Geometry, math.KT = K-Theory and Homology
- math.NT = Number Theory, math.AG = Algebraic Geometry, math.CO = Combinatorics
- math.PR = Probability, math.ST = Statistics Theory, math.DS = Dynamical Systems
- math.SG = Symplectic Geometry, math.MG = Metric Geometry, math.LO = Logic

**Usage:** When constructing arXiv queries, include `cat:{category}` in the `search_query` parameter to restrict results to relevant categories.

## 6. API Reference

These endpoints are codified here so the agent does not rely on training data. Use these exact URLs and parameters.

### arXiv API

**Base URL:** `http://export.arxiv.org/api/query`

**Search query:**
```
GET http://export.arxiv.org/api/query?search_query={query}&start=0&max_results=10&sortBy=relevance&sortOrder=descending
```

**Query syntax:**
- Field prefixes: `ti:` (title), `au:` (author), `abs:` (abstract), `cat:` (category), `all:` (all fields)
- Boolean operators: `AND`, `OR`, `ANDNOT`
- Grouping: use `%28` and `%29` for parentheses in URLs, or `(` and `)` in query strings before encoding

**Example search query:**
```
http://export.arxiv.org/api/query?search_query=ti:banach+AND+abs:fixed+point+AND+cat:math.FA&start=0&max_results=10&sortBy=relevance
```

**Batch verification by ID:**
```
http://export.arxiv.org/api/query?id_list=2301.12345,2302.67890,2303.11111
```

Use `id_list` for verification -- it fetches multiple papers in a single call, avoiding per-paper rate limits.

**Pagination:**
- `start`: zero-based offset (default 0)
- `max_results`: number of results (default 10, max 2000)

**Sort options:**
- `sortBy`: `relevance` (default), `lastUpdatedDate`, `submittedDate`
- `sortOrder`: `descending` (default), `ascending`

**Response format:** Atom 1.0 XML

**Key XML elements:**
```xml
<entry>
  <id>http://arxiv.org/abs/2301.12345v1</id>
  <title>Paper Title</title>
  <summary>Abstract text</summary>
  <author><name>Author Name</name></author>
  <published>2023-01-15T00:00:00Z</published>
  <arxiv:primary_category term="math.FA"/>
  <arxiv:doi>10.1234/example</arxiv:doi>
</entry>
```

To extract the arXiv ID from `<id>`: strip `http://arxiv.org/abs/` prefix and version suffix (e.g., `v1`).

**Rate limit:** 3-second delay between calls recommended. The `id_list` batch approach helps stay within limits during verification.

### Semantic Scholar API

**Base URL:** `https://api.semanticscholar.org/graph/v1`

**Paper search:**
```
GET https://api.semanticscholar.org/graph/v1/paper/search?query={terms}&fields=paperId,title,authors,year,abstract,externalIds,citationCount,fieldsOfStudy,url,openAccessPdf&limit=10&fieldsOfStudy=Mathematics
```

**Paper lookup (for verification):**
```
GET https://api.semanticscholar.org/graph/v1/paper/{id}?fields=paperId,title,authors,year,abstract,externalIds,citationCount,fieldsOfStudy,url,openAccessPdf
```

Where `{id}` can be:
- `ARXIV:{arxiv_id}` (e.g., `ARXIV:2301.12345`)
- `DOI:{doi}` (e.g., `DOI:10.1234/example`)
- S2 `paperId` (e.g., `649def34f8be52c8b66281af98ae884c09aef38b`)

**Batch lookup (for larger sets):**
```
POST https://api.semanticscholar.org/graph/v1/paper/batch?fields=paperId,title,authors,year,abstract,externalIds,citationCount,fieldsOfStudy,url,openAccessPdf
```

Request body:
```json
{"ids": ["ARXIV:2301.12345", "DOI:10.1234/example"]}
```

Use Bash + curl for the POST request since Claude Code tools do not natively support POST.

**Key fields to request:** `paperId,title,authors,year,abstract,externalIds,citationCount,fieldsOfStudy,url,openAccessPdf`

**Rate limit:** Shared unauthenticated pool. For higher limits, use an API key.

**API key configuration:**
- HTTP header: `x-api-key: {key}`
- Key sources (checked in order):
  1. Environment variable: `SEMANTIC_SCHOLAR_API_KEY`
  2. Config file: `.math/config.json` field `semantic_scholar_api_key`
- If no key is available, use unauthenticated access (lower rate limits but functional)

## 7. Query Construction Guidelines

When building queries from PROBLEM.md, follow these rules:

1. **Strip LaTeX notation** from search terms. Convert `\mathbb{R}^n` to "R^n" or "Euclidean space", `\|T\|` to "operator norm", etc.
2. **Extract key mathematical nouns** from the problem statement: theorem names, technique names, named spaces, named operators
3. **Use PROBLEM.md `tags`** as concept keywords for queries
4. **Use PROBLEM.md `domain`** to select arXiv categories via the mapping in Section 5
5. **Use PROBLEM.md `known_results` and `references`** as query seeds -- author names and theorem names make excellent targeted queries
6. **Generate 3-5 distinct queries** targeting different aspects:
   - Query 1: Core problem keywords (title + abstract search)
   - Query 2: Domain-specific category browse with key concepts
   - Query 3: Known theorem or technique names
   - Query 4: Author-based (if references provided)
   - Query 5: Broader related concepts (secondary categories)

**Example queries for a functional analysis problem about operator norms:**

arXiv:
```
http://export.arxiv.org/api/query?search_query=ti:operator+norm+AND+abs:bounded+linear+AND+cat:math.FA&max_results=10&sortBy=relevance
```

Semantic Scholar:
```
https://api.semanticscholar.org/graph/v1/paper/search?query=operator+norm+bounded+linear+functional+analysis&fields=paperId,title,authors,year,abstract,externalIds,citationCount,fieldsOfStudy,url,openAccessPdf&limit=10&fieldsOfStudy=Mathematics
```

## 8. Incremental Search Rules

When LITERATURE.md already exists from a prior search:

1. **Read it first** to understand what has already been found
2. **Do not re-run identical queries** -- check the Search History table for prior queries
3. **New searches complement prior results** -- they do not replace them
4. **Track all searches** in the Search History table, including new ones
5. **REF-NNN numbering continues** from the last entry -- never reset to REF-001
6. **UREF-NNN numbering continues** from the last unconfirmed entry
7. **Deduplicate across searches:** Before adding a new confirmed reference, check if the same paper (by title + authors + year) already exists in Confirmed References
8. **Update frontmatter counts** after each search: `total_papers`, `confirmed_count`, `unconfirmed_count`, `last_search`, `sources_queried`

## 9. Journal Integration

After every completed search operation, the agent writes a journal entry via the session-manager agent.

**Category:** `LIT-{NNN}` (Literature Searches)

**Strategy type selection:**
- `broad-survey` -- for initial searches on a new problem, general exploration of the literature landscape
- `targeted-search` -- for follow-up searches targeting specific theorems, authors, or techniques identified in prior results

**Required journal entry content:**
- **What was tried:** Summary of queries executed and APIs queried
- **Outcome:** SUCCEEDED (found relevant papers), PARTIAL (some results but gaps remain), FAILED (no relevant results)
- **Artifacts produced:** Link to LITERATURE.md
- **Insight/Takeaway:** Key observation about the literature landscape (e.g., "Most work on this topic is from 2018-2022 and focuses on finite-dimensional cases", "No direct results found -- closest work is in adjacent field X")

**Example journal entry:**
```
### LIT-001: Initial survey of operator norm convergence results
- **Timestamp:** 2026-02-10T14:30:00Z
- **Agent:** literature-search
- **Strategy type:** broad-survey
- **Tags:** [operator-norm, banach-space, convergence, spectral-radius]
- **What was tried:** 5 queries across arXiv and Semantic Scholar targeting operator norm convergence in Banach spaces
- **Outcome:** SUCCEEDED
- **Reasoning:** Found 7 confirmed papers directly addressing operator norm convergence criteria
- **Artifacts produced:** [LITERATURE.md](LITERATURE.md)
- **Related files:** [PROBLEM.md](PROBLEM.md)
- **Insight/Takeaway:** The literature clusters around two approaches: spectral radius methods (post-2015) and classical Neumann series (pre-2010). The spectral approach appears more aligned with the user's problem formulation.
```

## 10. Rate Limit Handling

Design searches for 10-20 candidate papers per search, not hundreds.

**arXiv:**
- Use `id_list` for batch verification -- one API call verifies all arXiv candidates
- Space discovery queries 3 seconds apart
- If HTTP 429 (rate limited): wait 10 seconds, retry once, then skip with note in Search History

**Semantic Scholar:**
- Use individual `GET /paper/{id}` for small batches (fewer than 10 papers)
- Use `POST /paper/batch` via Bash + curl for larger sets (10+ papers)
- If HTTP 429: wait 5 seconds, retry once, then skip with note in Search History

**Graceful degradation:**
- If one API is unavailable, complete the search with the other API alone
- Note the unavailable API in the Search History table
- Mark papers that could only be checked against one API with a note in their Verified field
