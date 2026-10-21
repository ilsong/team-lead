# init — set up team lead instructions

Adds a team lead block to the chosen platforms' instruction files and creates agent definitions for the chosen roles. Formats are in `platforms.md`.

## 1. Ask

Use the platform's question tool (`platforms.md` §1), at most 4 questions per round. Put the recommended option first, marked "(Recommended)". Pre-select what's obvious (platforms from `CLAUDE.md`·`.claude/`·`AGENTS.md`·`.codex/`·`GEMINI.md` and `command -v claude codex gemini`). If a team lead block already exists, use its values as defaults.

**Round 1**
1. **Platforms** (multi) — Claude Code / Codex / Gemini CLI / other
2. **Preset roles** (multi, all three recommended) — `worker` (implementation) / `reviewer` (code review, read-only) / `verifier` (runs tests, browser, emulator)
3. **Custom roles** — always ask, separately from presets: none / add (the user describes each: name, what it does, read-only or writes files, model level high/mid/low)
4. **Delegation methods** (multi, `platforms.md` §4) — subagent (Recommended) / child chat session / other agent CLI / manual handoff

**Round 2** (ask only what applies)
- **Model level** — Balanced (Recommended) / Quality / Economy
- If more than one method: method and executor per role (e.g. reviewer = `codex exec`). Default: subagent on each platform.
- If child chat session: open only after user approval (Recommended) / freely at the lead's discretion.
- **Before dispatch** — show the breakdown and wait for approval (Recommended) / dispatch right away.

**Round 3**
- **Parallel write tasks** — separate git worktree each (Recommended) / separate `git clone` each / same checkout, split by files.
- If worktree or clone: **when a task is done** — lead merges its branch locally (Recommended) / open a PR per task / leave the branch for the user.
- **Where to record**, for each platform — project-personal (Recommended, e.g. `CLAUDE.local.md`) / project (e.g. `CLAUDE.md`, `AGENTS.md`) / global (e.g. `~/.claude/CLAUDE.md`). Offer only the scopes that platform has.

## 2. Write

For each chosen platform:
- **Block**: fill `templates/block.md` and append it to the end of the instruction file for the chosen scope (create the file if missing). If a `team-lead:start`…`end` block already exists, replace it; if the scope changed, remove it from the old file.
  - `{{ROLE_ROWS}}`: `| role | what it does | method · executor (model / effort) |` for chosen roles.
  - `{{METHODS}}`: one bullet per chosen method other than subagent, condensed from `platforms.md` §4 (the CLI command to run; the child-chat opening rule in bold). If child chat sessions weren't chosen: "- Open separate chat sessions only when the user asks."
  - `{{APPROVAL}}`: "Before dispatching, show the breakdown (task · role · parallel or not) and wait for the user's approval." or "Dispatch without asking; tell the user the breakdown as you go."
  - `{{ISOLATION}}`:
    - worktree: "Give each parallel write task its own git worktree (the platform's isolation feature or `git worktree add`). Read-only tasks share this checkout. When a task is done, <done>."
    - clone: "Give each parallel write task its own `git clone` next to this repo. Read-only tasks share this checkout. When a task is done, <done>."
    - same checkout: "All executors work in this checkout; split scope so no two edit the same file at the same time."
    - `<done>`: "merge its branch locally and remove the worktree/clone" / "push its branch and open a PR" / "leave its branch and tell the user".
- **Agents**: one definition per role in that platform's format (`platforms.md` §2). Preset roles come from `templates/roles/<role>.md`; custom roles are written from the user's description in the same shape (`access`: read or write, `tier` from the chosen level). Global scope → global agents folder; otherwise the project one. Platforms without agent definitions get none.

Change nothing else.

## 3. Wrap up

- Table of changed files and each role's method, executor, model/effort.
- Don't commit.
- New agent definitions may only be picked up from a new session.
- Next: `team-lead assign` in a new session (Claude Code `/team-lead assign`, Codex `$team-lead assign`).
