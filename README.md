# team-lead

An [Agent Skill](https://agentskills.io) that sets up, per project, a way of working where the main session acts as the **team lead (orchestrator)** and hands detailed work to subagents, child chat sessions, or other agent CLIs. Works with Claude Code, Codex, and any agent that reads skills.

```
team-lead init     # Set up roles, delegation methods, and operating rules for this project (asks as it goes)
team-lead assign   # Make the current chat session the team lead
```

| Agent | How to invoke |
|---|---|
| Claude Code | `/team-lead init`, `/team-lead assign` |
| Codex | `$team-lead init`, `$team-lead assign` |
| Others | Say the name and mode, e.g. "run the team-lead skill with init" |

## Why

A session that does everything alone fills up its context as the conversation grows, and ends up using an expensive model for simple implementation too. team-lead splits the roles.

| Who | Does |
|---|---|
| Team lead (main session) | Design, task breakdown, parallel dispatch, reviewing results, integration (asks for PR/merge approval) |
| `worker` | Implementation and unit tests |
| `reviewer` | Code review (read-only) |
| `researcher` | Code/doc research (read-only, cheap model) |
| `verifier` | Runtime verification (tests, browser, emulator, etc.) — runs in the background; the lead doesn't wait |

- The lead finds tasks that don't depend on each other and dispatches them **in parallel, all at once**; parallel tasks that write files are isolated in worktrees.
- Each role has a fixed model and effort level, so cost and quality stay consistent without specifying them every time.
- Merges always wait for user approval. Executor reports are checked against the diff and tests before being passed on to the user.

## Delegation methods

Chosen in init, and can differ per role.

| Method | What | Example |
|---|---|---|
| Subagent (default) | Delegate within the same session. Nothing to clean up | Claude Code `.claude/agents/`, Codex `.codex/agents/` |
| Child chat session | Open a separate chat in the app. You set the opening rule (ask every time / free) | Claude desktop sessions, Codex app threads |
| Other agent CLI | Run another agent headless. Different agents collaborate | Claude lead → review with `codex exec` |
| Manual handoff | The lead writes a brief file and a person runs it in another tool | Anywhere |

## Install

**Personal (use in all my projects)** — install with the [`skills` CLI](https://github.com/vercel-labs/skills), choosing the agents you use.

```bash
npx skills add ilsong/team-lead -g
```

To pick agents, add `-a claude-code -a codex`. You can also clone with git:

```bash
git clone https://github.com/ilsong/team-lead.git ~/.claude/skills/team-lead   # Claude Code
git clone https://github.com/ilsong/team-lead.git ~/.agents/skills/team-lead   # Codex and others
```

Installing changes no project. Only projects where you run `init` get set up.

**Team sharing (anyone who clones the repo)** — choose "Share with team" in `init` and a copy of the skill goes into the project's `.claude/skills/team-lead/` (Claude Code) and `.agents/skills/team-lead/` (Codex and others). Commit the created files and teammates don't need to install anything. To add it yourself, from the project root:

```bash
npx skills add ilsong/team-lead --copy
```

**Update**: run the install command again, or `git pull` in that folder if you installed with git.

## Usage

### `init` — project setup

It first surveys the project (traces of agents in use, installed CLIs, language, build/test commands, CI, git remote, whether there's a UI), then asks with the platform's question UI (or a numbered list) and sets things up.

1. **Platforms** — Claude Code / Codex / Gemini CLI / other
2. **Roles** — worker / reviewer / researcher / verifier
3. **Delegation methods** — subagent / child chat session / other agent CLI / manual handoff
4. **Model level** — Balanced (recommended) / Quality / Economy

   | Level | Claude Code default |
   |---|---|
   | Balanced | worker·reviewer `sonnet`/high, verifier `haiku`/high, researcher `haiku`/medium |
   | Quality | One step up (worker·reviewer `opus`) |
   | Economy | One step lower effort |

   For other platforms it suggests the same levels from the installed CLI's model list, and asks if unsure.
5. Follow-up questions depending on your answers — method and executor per role (e.g. reviewer on Codex), the rule for opening child chats, how the lead turns on (only on `assign` / always), **where to record "always"** (project-personal `CLAUDE.local.md` / project `CLAUDE.md` / global `~/.claude/CLAUDE.md`), work rules (branch → PR → merge after approval, parallel worktrees, async verification, team sharing), verification targets, build/test commands.

Files created (`<dir>` = each chosen platform's folder: `.claude`, `.codex`, ...):

| File | Contents |
|---|---|
| `<dir>/team-lead/rules.md` | Operating rules (one identical copy per platform) |
| `.claude/agents/*.md`, `.codex/agents/*.toml` | Subagent definitions per role (only for platforms you use) |
| A line in the instruction file for the chosen scope | Only if you chose "always" |
| `.claude/skills/team-lead/`, `.agents/skills/team-lead/` | Only if you chose "Share with team" |

- Global settings, agents, and other home-directory files are not touched. The only exception is the global instruction file, when you choose the global scope. Nothing is committed.
- Re-running shows current values as defaults and changes only what differs. Legacy layouts (`.team-lead/`, `.claude/team-lead.md`) are migrated to the new layout.
- New subagents may only show up **from a new session**.

### `assign` — make this session the team lead

Reads this platform's `<dir>/team-lead/rules.md`, makes the current session the team lead, and shows the roster of executors available on this platform. If an executor set in the rules isn't available (e.g. you assign in Codex but only Claude subagents are defined), it falls back subagent → CLI → handoff and tells you. Add a task after it to start right away.

```
/team-lead assign Add social login to the login screen
```

It also works in projects without init, using the default rules and the platform's default subagent.

## Structure

```
SKILL.md                  # Entry point: init / assign dispatch
references/init.md        # init procedure
references/assign.md      # assign procedure
references/platforms.md   # Per-platform locations, instruction-file scopes, definition formats, models, delegation methods
templates/rules.md        # Operating rules template
templates/roles/*.md      # Role templates (worker, reviewer, researcher, verifier)
```

To change rules or roles, edit the files created in the project, or edit the templates and re-run `init`. To support a new platform, add rows to `references/platforms.md`.

## License

MIT
