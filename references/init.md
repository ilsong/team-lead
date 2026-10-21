# init — set up team lead instructions

Adds a team lead block to the chosen platforms' instruction files and creates agent definitions for the chosen roles. Formats are in `platforms.md`.

## 1. Ask

Use the platform's question tool (`platforms.md` §1). Put the recommended option first, marked "(Recommended)". Pre-select what's obvious (platforms from `CLAUDE.md`·`.claude/`·`AGENTS.md`·`.codex/`·`GEMINI.md` and `command -v claude codex gemini`). If a team lead block already exists, use its values as defaults.

**Round 1**
1. **Platforms** (multi) — Claude Code / Codex / Gemini CLI / other
2. **Roles** (multi) — `worker` (implementation, always recommended) / `reviewer` (code review, read-only) / `researcher` (research, read-only, cheap model) / `verifier` (runs tests, browser, emulator)
3. **Delegation methods** (multi, `platforms.md` §4) — subagent (Recommended) / child chat session / other agent CLI / manual handoff
4. **Model level** — Balanced (Recommended) / Quality / Economy

**Round 2**
- If more than one method: method and executor per role (e.g. reviewer = `codex exec`). Default: subagent on each platform.
- If child chat session: open only after user approval (Recommended) / freely at the lead's discretion.
- **When the lead is on** — only after `team-lead assign` (Recommended) / every session.
- **Where to record**, for each platform — project-personal (Recommended, e.g. `CLAUDE.local.md`) / project (e.g. `CLAUDE.md`, `AGENTS.md`) / global (e.g. `~/.claude/CLAUDE.md`). Offer only the scopes that platform has.

## 2. Write

For each chosen platform:
- **Block**: fill `templates/block.md` and append it to the end of the instruction file for the chosen scope (create the file if missing). If a `team-lead:start`…`end` block already exists, replace it; if the scope changed, remove it from the old file.
  - `{{TRIGGER}}`: "When assigned with `team-lead assign`" or "In every session".
  - `{{ROLE_ROWS}}`: `| role | what it does | method · executor (model / effort) |` for chosen roles.
  - `{{METHODS}}`: one bullet per chosen method other than subagent, condensed from `platforms.md` §4 (the CLI command to run; the child-chat opening rule in bold). If child chat sessions weren't chosen: "- Open separate chat sessions only when the user asks."
- **Agents**: one definition per chosen role from `templates/roles/<role>.md` in that platform's format (`platforms.md` §2). Global scope → global agents folder; otherwise the project one. Platforms without agent definitions get none.

Change nothing else.

## 3. Wrap up

- Table of changed files and each role's method, executor, model/effort.
- Don't commit.
- New agent definitions may only be picked up from a new session.
- Next: `team-lead assign` in a new session (Claude Code `/team-lead assign`, Codex `$team-lead assign`), or just give a task if every session is the lead.
