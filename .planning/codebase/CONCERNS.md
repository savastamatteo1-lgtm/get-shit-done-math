# Codebase Concerns

**Analysis Date:** 2026-02-08

## Tech Debt

**Command Execution via spawn() in hooks:**
- Issue: `hooks/gsd-check-update.js` spawns Node.js subprocess with inline code string, passing file paths via JSON.stringify
- Files: `hooks/gsd-check-update.js` (lines 25-56)
- Impact: While properly escaped with JSON.stringify, subprocess pattern creates maintenance surface and makes debugging harder
- Fix approach: Consolidate update check logic into main hook file or extract to separate module

**Large Install Script:**
- Issue: `bin/install.js` is 1529 lines, handling multiple concerns: argument parsing, config directory resolution, format conversion (Claude→OpenCode→Gemini), file operations, settings cleanup
- Files: `bin/install.js`
- Impact: Difficult to modify one aspect without unintended side effects; testing requires mocking filesystem extensively
- Fix approach: Extract into modules: `cli-args.js`, `config-dirs.js`, `format-converters.js`, `install-operations.js`

**Complex Frontmatter Conversion Logic:**
- Issue: Multiple format conversion functions (`convertClaudeToOpencodeFrontmatter`, `convertClaudeToGeminiAgent`, `convertClaudeToGeminiToml`) using regex-based YAML parsing
- Files: `bin/install.js` (lines 268-583)
- Impact: Brittle to YAML edge cases (quoted strings, nested structures); three similar functions with different rules risk divergence
- Fix approach: Use proper YAML parser library (e.g., `js-yaml`); create unified format converter with pluggable target format rules

**Git Command Execution Pattern:**
- Issue: `get-shit-done/bin/gsd-tools.js` uses `execSync` with manual shell escaping (lines 93-123)
- Files: `get-shit-done/bin/gsd-tools.js` (execGit function, isGitIgnored function)
- Impact: Shell escape validation `/^[a-zA-Z0-9._\-/=:@]+$/` passes through git commands but could allow injection with edge case arguments
- Fix approach: Use simpler child_process.spawnSync with array args (avoids shell), or validate whitelist of safe git commands only

## Known Bugs

**Update Check Cache File Location Hardcoded:**
- Symptoms: Update check always uses `~/.claude/cache/` even when GSD installed to custom location
- Files: `hooks/gsd-check-update.js` (line 12)
- Trigger: Run `/gsd:settings` with non-default Claude config directory
- Workaround: Cache file may not reflect actual installed version if using custom install paths

**Statusline Hardcoded to Claude Config:**
- Symptoms: Hook registration in settings assumes Claude Code; Gemini statusline setup not verified
- Files: `bin/install.js` (lines 1234-1240, 1287-1292)
- Trigger: Install for Gemini runtime, check if statusline displays correctly
- Workaround: Manual statusline configuration in Gemini settings.json required

**OpenCode Permission Paths Not Validated:**
- Symptoms: If `get-shit-done` directory doesn't exist, permissions are still written and later permission checks may fail
- Files: `bin/install.js` (configureOpencodePermissions, lines 972-1030)
- Trigger: Uninstall GSD, then install again
- Workaround: Manually remove stale permissions from opencode.json

## Security Considerations

**API Key Prevention in Map Codebase (CRITICAL FIX - v1.11.2):**
- Risk: `/gsd:map-codebase` could read `.env` files and commit API keys to git
- Files: `agents/gsd-codebase-mapper.md` (should have forbidden_files list)
- Current mitigation: Agent prompt includes forbidden_files block; installer validates against `.env*` patterns before copying
- Recommendations: Enforce deny-list in agent headers; add pre-commit hook to reject files matching `*secret*|*credential*|.env*|*.key` patterns

**Spawn Process Subprocess Handling:**
- Risk: `hooks/gsd-check-update.js` spawns child process; if npm registry is compromised, could execute arbitrary code
- Files: `hooks/gsd-check-update.js` (lines 25-46)
- Current mitigation: timeout set to 10000ms; child.unref() allows hook to exit regardless
- Recommendations: Pin npm package hash; consider offline update check against local VERSION file only

**Git Command Injection Risk (Low):**
- Risk: If arg passes shell escape check but contains special chars, could break git command parsing
- Files: `get-shit-done/bin/gsd-tools.js` (isGitIgnored, execGit functions)
- Current mitigation: Whitelist validation and quoting; most git commands are read-only
- Recommendations: Use spawnSync instead of execSync to eliminate shell entirely

**File Path Traversal in Install:**
- Risk: User-controlled `--config-dir` path not validated; could install to arbitrary locations
- Files: `bin/install.js` (expandTilde function, getGlobalDir)
- Current mitigation: Path.join normalizes traversal attempts; process.cwd used as base
- Recommendations: Validate that resolved path starts with home directory; reject paths with `..` components

## Performance Bottlenecks

**Synchronous File I/O Throughout Install:**
- Problem: `bin/install.js` uses synchronous fs operations (readFileSync, writeFileSync, mkdirSync, rmSync)
- Files: `bin/install.js` (throughout, especially copyWithPathReplacement lines 648-691)
- Cause: For large installations with many files, recursive copy is slow; no parallelization
- Improvement path: Use fs.promises for concurrent operations; batch file operations; add progress reporting

**Regex-Based YAML Parsing:**
- Problem: `convertClaudeToOpencodeFrontmatter` and `convertClaudeToGeminiAgent` parse YAML line-by-line with regex matching
- Files: `bin/install.js` (lines 440-542, 370-438)
- Cause: No streaming; re-parses same fields multiple times; inefficient for large markdown files
- Improvement path: Single-pass YAML parsing with proper library; cache parsed frontmatter

**Git Status Check on Every Install:**
- Problem: `isGitIgnored` runs `git check-ignore` command for each file during install
- Files: `get-shit-done/bin/gsd-tools.js` (line 431 isGitIgnored called in cmdCommit)
- Cause: Spawns subprocess for single path check; no caching
- Improvement path: Read .gitignore once at startup; batch check multiple paths; cache results per session

## Fragile Areas

**Multi-Runtime Format Conversion:**
- Files: `bin/install.js` (format conversion functions)
- Why fragile: Three similar but divergent conversion implementations (OpenCode, Gemini markdown, Gemini TOML); easy to miss edge case in one while fixing another
- Safe modification: Create test suite with sample files for each format before refactoring; ensure all three targets produce equivalent output
- Test coverage: No unit tests for format converters; rely on manual testing during install

**Settings.json Cleanup Logic:**
- Files: `bin/install.js` (cleanupOrphanedHooks lines 714-766, cleanupOrphanedFiles lines 696-709)
- Why fragile: Maintains hardcoded lists of old file/hook names; if patterns change, orphaned items accumulate
- Safe modification: Scan filesystem to detect stale items rather than hardcoding names; use versioning in settings to track migrations
- Test coverage: No automated tests; manual verification only

**Phase Lookup and Normalization:**
- Files: `get-shit-done/bin/gsd-tools.js` (normalizePhaseName lines 126-133, cmdFindPhase lines 372-414)
- Why fragile: Phase naming allows decimal notation (1.1) but sorting is string-based; "2" > "10" alphabetically
- Safe modification: Use numeric sort; add tests for phase discovery with phases 1-10+
- Test coverage: No unit tests; integration tests assume standard phase naming

**Hook Command Building:**
- Files: `bin/install.js` (buildHookCommand lines 170-174)
- Why fragile: Path building for Windows uses forward slashes; assumes hook filenames match exactly; if hook renamed, settings not updated
- Safe modification: Store hook path resolution in config; validate hook paths exist before building commands
- Test coverage: No tests for cross-platform hook path handling

## Scaling Limits

**Memory Usage on Large Installs:**
- Current capacity: Install can handle projects with 1000+ markdown files; all files loaded into memory during copy
- Limit: Breaks at ~50MB of total markdown content due to synchronous reading
- Scaling path: Implement streaming copy; process files in chunks; add memory warning at startup

**Git Command Performance:**
- Current capacity: `execGit` handles typical 100-200 line commits; works fine for planning doc commits
- Limit: Large commits (>500 changed files) may timeout; no progress reporting
- Scaling path: Use libgit2 bindings instead of shell commands; implement commit batching

**Model Profile Resolution:**
- Current capacity: 11 agent types with 3 profile options each
- Limit: Adding new agent requires manual entry into MODEL_PROFILES table; no dynamic discovery
- Scaling path: Load model profiles from config file; support plugin agent registration

## Dependencies at Risk

**No Production Dependencies:**
- Risk: esbuild is devDependency only; hooks bundled at build time
- Impact: If hooks need runtime updates, users must reinstall
- Migration plan: Consider minimal runtime deps (none currently recommended)

**Shell Command Execution Dependency:**
- Risk: System must have git, npm, node available in PATH
- Impact: Docker, CI/CD, minimal environments may not have all tools
- Migration plan: Pre-validate tools at install time; provide clear error messages

## Missing Critical Features

**No Uninstall Verification:**
- Problem: Uninstall removes files but doesn't verify they were actually deleted
- Blocks: Can't reliably re-install after uninstall; orphaned config entries may remain
- Impact: Users may have mixed old/new configurations

**No Install Rollback:**
- Problem: If install fails partway through, no automatic cleanup
- Blocks: User must manually delete partial install directory
- Impact: Install state is ambiguous after failure

**No Config Migration Helper:**
- Problem: Config.json schema changes between versions; no migration script provided
- Blocks: Users upgrading with manual config must manually update fields
- Impact: Silent failures if new required fields missing

## Test Coverage Gaps

**Installer Logic Not Tested:**
- What's not tested: Argument parsing, config directory resolution, file copying, format conversion, settings cleanup
- Files: `bin/install.js` (all functions)
- Risk: Changes to install logic could break without detection; only discovered by users
- Priority: High — install is critical entry point

**Format Converters Not Tested:**
- What's not tested: OpenCode frontmatter conversion, Gemini agent conversion, edge cases (unicode, special YAML, quoted strings)
- Files: `bin/install.js` (convertClaudeToOpencodeFrontmatter, convertClaudeToGeminiAgent, convertClaudeToGeminiToml)
- Risk: Format divergence between runtimes; users report "commands don't work in [runtime]" with no way to reproduce
- Priority: High — format conversion affects all three runtimes

**Hook Integration Not Tested:**
- What's not tested: Hook execution in actual Claude Code/OpenCode/Gemini sessions, timing, error handling, cache file management
- Files: `hooks/gsd-check-update.js`, `hooks/gsd-statusline.js`
- Risk: Silent failures; users unaware hooks aren't running
- Priority: Medium — detected when statusline doesn't show or updates never appear

**Git Operations Not Tested:**
- What's not tested: execGit error handling, phase lookup edge cases, commit with special characters, .gitignore detection
- Files: `get-shit-done/bin/gsd-tools.js`
- Risk: Git integration breaks with certain project states (detached HEAD, no remotes, large commits)
- Priority: Medium — impacts commit-docs workflow

---

*Concerns audit: 2026-02-08*
