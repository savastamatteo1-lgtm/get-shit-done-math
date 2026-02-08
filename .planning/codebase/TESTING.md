# Testing Patterns

**Analysis Date:** 2026-02-08

## Test Framework

**Status:**
- No test framework configured
- No test files found in codebase (*.test.js, *.spec.js, jest.config.js, vitest.config.ts)
- No test dependencies in package.json (esbuild only)

**Current Approach:**
- Manual verification through CONTRIBUTING.md guidance
- Integration testing via CLI: `npm link` → `npx get-shit-done-cc`
- Agent-based verification in workflows (gsd-verifier.md)

## Testing Philosophy

**Testing is not automated but verification-driven:**

From CONTRIBUTING.md:
```bash
# Test locally
npm link
npx get-shit-done-cc
```

Testing happens at two levels:

1. **CLI Verification (manual):**
   - Install locally and test command execution
   - Test on Windows for path handling
   - Test on macOS/Linux for POSIX compatibility

2. **Agent-Based Verification:**
   - gsd-verifier agent inspects code for: stubs, TODOs, broken logic
   - Verification workflows check: file existence, git status, output correctness
   - Checkpoint-based halts at risky changes requiring human decision

## Verification Patterns (Agent-Based)

**Stub Detection:**
```bash
grep -c -E "TODO|FIXME|placeholder|not implemented|coming soon" "$path"
grep -n -E "TODO|FIXME|XXX|HACK" "$file"
```

Location: `agents/gsd-verifier.md`

**Empty Return Detection:**
```bash
grep -n -E "return null|return \[\]|return {}" "$file"
```

**Verification workflow pattern from `workflows/verify-phase.md`:**
- Stub detection: `TODO|FIXME|placeholder|not implemented|coming soon`
- Empty returns: `return null|return {}|return []`
- Placeholder content detection
- File existence checks
- Output correctness verification via bash commands

## Deviation Rules (Plan Execution)

From `agents/gsd-executor.md`:

**Rule 1: Auto-fix bugs**
- Trigger: Code doesn't work as intended
- Examples: Wrong queries, logic errors, type errors, null pointer exceptions

**Rule 2: Auto-add missing critical functionality**
- Trigger: Code missing essential features for correctness/security
- Examples: Missing error handling, no validation, no auth on protected routes

**Rule 3: Auto-fix blocking issues**
- Trigger: Something prevents completing current task
- Examples: Missing dependency, broken imports, missing env var

**Rule 4: Ask about architectural changes**
- Trigger: Fix requires significant structural modification
- Examples: New DB table, major schema changes, new service layer
- Action: STOP and return checkpoint with decision matrix

## What to Test

**Critical paths (must verify before shipping):**
- Installation on all platforms: Windows, macOS, Linux
- Path handling: Windows UNC paths, relative/absolute paths
- Hook execution: statusline display, update checks
- Command execution: /gsd:help, /gsd:install working
- Configuration: .env variables, settings.json parsing
- Git integration: branch creation, commits, tags

**Code quality checks (agent-based):**
- No unhandled promise rejections
- No console.log in production paths (except CLI output)
- No hardcoded paths (use path.join)
- All error cases handled
- Security: no exposed credentials, no eval(), no command injection

## Test Data & Fixtures

**Not implemented.** Manual testing approach:

**For installation testing:**
- Use fresh directory: `mkdir test-install && cd test-install`
- Run: `npx get-shit-done-cc --claude --local`
- Verify: files exist in `./.claude/`
- Test: `/gsd:help` command works in Claude Code

**For runtime testing:**
- Test with multiple Node versions: node 16, 18, 20
- Test on Windows, macOS, Linux
- Test with various shell environments: bash, zsh, fish, PowerShell

## Manual Verification Checklist

From CONTRIBUTING.md and GSD-STYLE.md:

**Before releasing:**
- [ ] Follows GSD style (no enterprise patterns, no filler)
- [ ] Updates CHANGELOG.md for user-facing changes
- [ ] Doesn't add unnecessary dependencies
- [ ] Works on Windows (test paths with backslashes)
- [ ] Run: `npm link`
- [ ] Run: `npx get-shit-done-cc`
- [ ] Test installation: `--global` and `--local`
- [ ] Test for all runtimes: Claude Code, OpenCode, Gemini
- [ ] Verify no hardcoded home directories
- [ ] Test hook execution (statusline, update checks)

## Coverage Expectations

**Not enforced.** No coverage tool configured.

**Coverage focus (implied):**
- Critical path: 100% verification (cannot ship broken)
- Platform compatibility: test on Windows, macOS, Linux
- Edge cases: Windows UNC paths, symlinks, special characters in paths
- Error cases: missing files, invalid JSON, network timeouts

## Test Organization Strategy

**No local tests.** Testing strategy is:

1. **Unit verification** → Manual CLI testing + agent inspection
2. **Integration testing** → Test on different platforms and Node versions
3. **Release testing** → Full manual checklist in CONTRIBUTING.md
4. **Continuous verification** → Agent-based checks on every commit (future: pre-commit hooks)

## Checkpoint Protocol (Plan Execution)

Test-like verification during execution:

From `agents/gsd-executor.md`:

```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Description of what was built</what-built>
  <how-to-verify>Numbered steps for user</how-to-verify>
  <resume-signal>Text telling user how to continue</resume-signal>
</task>
```

**Usage:**
- Halt execution before risky changes (architectural decisions)
- Provide numbered verification steps for user
- User tests change, then sends resume signal
- Executor continues with next task

## Platform Testing

**Required platforms:**
- Windows 10/11 (test UNC paths: `\\server\share`, long paths)
- macOS (Intel and Apple Silicon)
- Linux (Ubuntu, Fedora, Alpine)

**Node.js versions:**
- Node 16.7.0+ (minimum from package.json engines)
- Test on 16.x, 18.x, 20.x

**Shells:**
- bash (default on macOS/Linux)
- zsh (new default on macOS)
- PowerShell (Windows)
- fish (cross-platform)

## Error Path Testing

**Common error scenarios to test manually:**

1. **Missing dependencies:**
   - Node version too old
   - Missing npm
   - Missing git

2. **Permission errors:**
   - Can't write to config directory
   - Can't create symlinks (Windows)
   - Running as non-root on Linux

3. **Invalid input:**
   - Unknown runtime: `--unknown-runtime`
   - Both `--global` and `--local` together
   - `--config-dir` without path

4. **State errors:**
   - Corrupted settings.json
   - Missing .planning/ directory
   - Git not initialized

## Future Testing Improvements

**Potential additions (not currently implemented):**
- Automated CLI testing via npm test (execute install scenarios)
- Jest/Vitest with mocked file system
- Pre-commit hooks to catch stubs and unfinished work
- CI/CD matrix testing (Windows, macOS, Linux + multiple Node versions)
- Docker-based testing for platform consistency

---

*Testing analysis: 2026-02-08*
*Strategy: Agent-based verification + manual CLI testing + checkpoint protocols*
*No automated test framework configured — focus on verification through execution*
