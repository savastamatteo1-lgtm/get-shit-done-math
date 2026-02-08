# External Integrations

**Analysis Date:** 2026-02-08

## APIs & External Services

**None Required for Core GSD:**
- GSD is self-contained and does not require external API calls
- All planning, orchestration, and workflow management happen locally

**Optional Runtime Services (Project-Specific):**
- When scanning codebases during planning/execution phases, agents may detect and work with external APIs
- Examples: Stripe, SendGrid, Twilio, OpenAI, custom webhooks
- Not integrated into GSD itself; GSD helps detect and route to them

**Tool Integrations:**
- WebFetch - Used by agents (`gsd-planner`, `gsd-executor`) to fetch documentation during research
- WebSearch - Used by research agents (`gsd-project-researcher`, `gsd-phase-researcher`) for library/framework documentation lookups

## MCP (Model Context Protocol) Integration

**Context7 Library Documentation MCP:**
- Agents: `gsd-project-researcher`, `gsd-phase-researcher`, `gsd-planner`
- Tools: `mcp__context7__*` (dynamic MCP tool prefix)
- Available Methods:
  - `mcp__context7__resolve-library-id` - Convert library name to ID (e.g., "react" → ID)
  - `mcp__context7__query-docs` - Query library documentation with libraryId + search query
- Purpose: Research phase agents use Context7 to answer framework/library questions without manual lookup

## Data Storage

**Local File System Only:**
- No databases required
- All configuration and state stored as JSON/Markdown files:
  - `.planning/config.json` - Project workflow configuration
  - `.planning/STATE.md` - Project status and task tracking
  - `.planning/phases/` - Phase plans and summaries
  - `.planning/todos/` - Task lists
  - `settings.json` in runtime config dirs (`~/.claude/`, `~/.config/opencode/`, `~/.gemini/`)

**Caching:**
- Update check cache: `~/.claude/cache/gsd-update-check.json`
- Hook status cache during sessions (in-memory)

## Authentication & Identity

**No External Auth Provider:**
- GSD is local-only, no user accounts or authentication required
- Works with CLI runtimes (Claude Code, OpenCode, Gemini) that handle their own auth
- Git commits use local user.name/user.email from git config

**Runtime-Specific Configuration:**
- Claude Code: Uses `~/.claude/settings.json` (Claude Code manages model access)
- OpenCode: Uses `~/.config/opencode/opencode.json` (OpenCode handles permissions)
- Gemini: Uses `~/.gemini/settings.json` (Gemini CLI manages model access)

## Monitoring & Observability

**Error Tracking:**
- None - No error reporting service
- Errors logged to stdout/stderr during execution

**Logs:**
- Console output (stdout/stderr) during CLI operations
- No persistent logging
- Verbose output available via agent execution logs in runtime IDE

**Update Checking:**
- `gsd-check-update.js` hook checks npm registry for newer version
- Cache location: `~/.claude/cache/gsd-update-check.json`
- Frequency: On session start (SessionStart hook)
- No network request if cache exists and is fresh

## CI/CD & Deployment

**Hosting:**
- npm package registry (npmjs.com)
- GitHub repository: https://github.com/glittercowboy/get-shit-done

**CI Pipeline:**
- None detected in codebase
- Manual publish via npm (handled by maintainer)
- Pre-publish hook runs build: `npm run build:hooks`

**Distribution:**
- NPM package: `get-shit-done-cc` v1.12.0
- Installation method: `npx get-shit-done-cc`
- Supports interactive and non-interactive installs

## Environment Configuration

**Required env vars for runtime detection:**
- `CLAUDE_CONFIG_DIR` (optional) - Custom Claude Code config location
- `OPENCODE_CONFIG_DIR` or `OPENCODE_CONFIG` (optional) - OpenCode config location
- `GEMINI_CONFIG_DIR` (optional) - Gemini config location
- `XDG_CONFIG_HOME` (optional) - XDG Base Directory standard for OpenCode

**No secrets needed:**
- GSD itself requires no API keys, tokens, or credentials
- When scanning projects, GSD may detect `.env` files but does not read/access them

**User project configuration:**
- Stored in `.planning/config.json` after initialization
- Settings: model_profile, commit_docs, branching_strategy, workflow flags, parallelization options

## Webhooks & Callbacks

**Incoming:**
- None - GSD does not expose any webhook endpoints

**Outgoing:**
- Git commit hooks via `settings.json`:
  - `SessionStart` hook - Runs update check
  - `statusLine` hook - Returns formatted status for IDE display
- Custom hooks can be registered in runtime settings (user-configurable)

## Git Integration

**Usage:**
- Git operations via `child_process.execSync('git ...')`
- Commands used:
  - `git check-ignore` - Verify if `.planning/` is gitignored
  - `git add` - Stage planning documents
  - `git commit` - Create commits for planning artifacts
  - `git rev-parse --short HEAD` - Get commit hash
  - `git cat-file -t` - Verify commit hashes in verification

**Commit Processing:**
- Attribution handling: Can modify/remove `Co-Authored-By` lines per runtime config
- Claude Code uses: `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>`
- OpenCode: Uses `disable_ai_attribution` setting
- Gemini: Uses `attribution.commit` setting

## File Conversion & Runtime Compatibility

**Claude Code Format:**
- File: `.md` with YAML frontmatter
- Structure: `/commands/gsd/`, `/agents/`
- Command format: `/gsd:command`
- Frontmatter fields: name, description, tools, color, allowed-tools

**OpenCode Format:**
- File: `.md` with YAML frontmatter
- Structure: `/command/` (flat, no nesting)
- Command format: `/gsd-command`
- Frontmatter fields: description, tools (object with boolean values)
- Path references: `~/.claude/` → `~/.config/opencode/`

**Gemini Format:**
- File: `.toml` (converted from markdown)
- Structure: `/command/`
- Command format: `/gsd-command`
- Tool names: snake_case (e.g., read_file not Read)
- Experimental agents enabled for custom agent support

## Statusline & Session Hooks

**Statusline Display:**
- Hook: `gsd-statusline.js`
- Reads JSON from stdin (Claude Code session context)
- Displays: Model name | Current task | Directory | Context window usage (% bar)
- Reads todos from: `~/.claude/todos/` based on session_id
- Shows update available notification if cache indicates new version

**Update Check Hook:**
- Hook: `gsd-check-update.js`
- Runs at session start
- Checks npm registry for newer version
- Caches result to avoid repeated checks

## Permissions & Access Control

**OpenCode Permissions:**
- GSD requests read permission for `~/.config/opencode/get-shit-done/*`
- GSD requests external_directory permission for GSD docs path
- Configuration: `opencode.json` permission blocks

**Claude Code & Gemini:**
- No special permissions needed beyond standard file access
- Agents have access to sandboxed tools (Read, Write, Bash, etc.)

---

*Integration audit: 2026-02-08*
