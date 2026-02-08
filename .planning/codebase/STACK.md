# Technology Stack

**Analysis Date:** 2026-02-08

## Languages

**Primary:**
- JavaScript (Node.js) - Core CLI and installation scripts
- Markdown - Agent definitions, commands, and configuration documentation

## Runtime

**Environment:**
- Node.js >= 16.7.0

**Package Manager:**
- npm
- Lockfile: `package-lock.json` (present)

## Frameworks

**Core:**
- None - Pure Node.js CLI without external framework dependencies

**Build/Dev:**
- esbuild ^0.24.0 - Hook bundling and asset optimization

## Key Dependencies

**Critical:**
- No production dependencies - Zero external npm packages in production
- Only one dev dependency: esbuild (for bundling hooks)

**Infrastructure:**
- Node.js built-in modules: `fs`, `path`, `os`, `readline`, `child_process`

## Configuration

**Environment:**
- Runtime config detection via environment variables:
  - `CLAUDE_CONFIG_DIR` - Claude Code config location (defaults to `~/.claude`)
  - `OPENCODE_CONFIG_DIR` / `OPENCODE_CONFIG` - OpenCode config (defaults to `~/.config/opencode`)
  - `GEMINI_CONFIG_DIR` - Gemini CLI config (defaults to `~/.gemini`)
  - `XDG_CONFIG_HOME` - XDG Base Directory standard support for OpenCode

**Build:**
- `scripts/build-hooks.js` - Builds hooks to `hooks/dist/` directory
- No TypeScript, ESLint, or Prettier configuration
- npm `prepublishOnly` hook runs `npm run build:hooks` before publishing

## Installation & Distribution

**Package Entry:**
- `bin/install.js` - Main installation CLI executable
- Distributed via npm as `get-shit-done-cc` package

**Installation Targets:**
- Claude Code: `~/.claude/`
- OpenCode: `~/.config/opencode/`
- Gemini: `~/.gemini/`
- Local project: `./.claude/`, `./.opencode/`, `./.gemini/`

**File Distribution:**
- Commands: `commands/gsd/*` → `commands/` (nested) or `command/` (flat for OpenCode)
- Agents: `agents/*.md` → `agents/` (with runtime-specific conversion)
- Workflows: `get-shit-done/` → Target runtime config directory
- Hooks: `hooks/dist/*.js` → `hooks/` in target directory
- Documentation: `CHANGELOG.md` → Bundled in install

## Platform Requirements

**Development:**
- Node.js >= 16.7.0
- Unix-like shell (bash/zsh) for git operations
- Cross-platform support: Mac, Windows, Linux

**Production:**
- Node.js >= 16.7.0 in Claude Code, OpenCode, or Gemini CLI
- Git repository (for commit operations)
- Home directory with config subdirectories writable

## Build & Installation Flow

1. **Develop:** Markdown files, Node.js scripts in source tree
2. **Build:** `npm run build:hooks` copies hooks to `hooks/dist/`
3. **Publish:** npm publishes with `prepublishOnly` hook
4. **Install:** `npx get-shit-done-cc` runs `bin/install.js`
5. **Runtime:** Agents run in Claude Code/OpenCode/Gemini with sandboxed tool access

## Architecture Notes

**Minimal Dependencies Philosophy:**
- No production npm dependencies intentionally
- All utilities written in vanilla Node.js
- Reduces surface area and deployment complexity
- Installation bundles agents/commands/workflows as static markdown/JSON files

**Multi-Runtime Support:**
- Universal codebase targets three AI IDE platforms
- Runtime-specific file structure conversion at install time
  - Claude Code uses `/gsd:command` format with YAML frontmatter
  - OpenCode uses `/gsd-command` format with YAML tools object
  - Gemini uses TOML format with snake_case tool names
- Attribution processing per-runtime (commit author config)

---

*Stack analysis: 2026-02-08*
