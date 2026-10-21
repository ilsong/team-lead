# Platforms and delegation methods

Reference tables for init and assign. For platforms not listed here, confirm the same items from that platform's current docs or help (`<cli> --help`), and ask the user about anything you can't confirm. Don't invent formats you don't know.

## 1. Locations per platform

| Platform | Folder `<dir>` | Skill location (project) | Native subagent definitions | Question tool |
|---|---|---|---|---|
| Claude Code | `.claude` | `.claude/skills/` | `.claude/agents/<name>.md` | `AskUserQuestion` |
| Codex | `.codex` | `.agents/skills/` | `.codex/agents/<name>.toml` | That tool if available, else numbered list |
| Gemini CLI | `.gemini` | `.agents/skills/` | Needs confirmation | Numbered list |
| Cursor, Copilot, OpenCode, Cline, etc. | `.agents` | `.agents/skills/` | Needs confirmation | Numbered list |

The rules file is `<dir>/team-lead/rules.md`; `cli` and `handoff` outputs also go under `<dir>/team-lead/`.

**Instruction files by scope** (where the "always on" line goes)

| Platform | Global | Project | Project-personal |
|---|---|---|---|
| Claude Code | `~/.claude/CLAUDE.md` | `CLAUDE.md` | `CLAUDE.local.md` |
| Codex | `~/.codex/AGENTS.md` | `AGENTS.md` | Not supported |
| Gemini CLI | `~/.gemini/GEMINI.md` | `GEMINI.md` | Not supported |
| Others | Check docs | `AGENTS.md` | Not supported |

Line to add, under a `## Workflow` heading:
- Claude Code project / project-personal: `@.claude/team-lead/rules.md`
- Other project files: "Sessions in this project act as team lead. Before working, read and follow `<dir>/team-lead/rules.md`."
- Global (`@` imports resolve relative to the file that contains them, so don't use them): "In a project that has `<dir>/team-lead/rules.md`, act as team lead: read and follow that file before working."

If there's no question tool, put the questions in one message with numbers and option letters (e.g. `1) a. … b. …`), have the user answer like `1a 2bc`, and wait for the answer.

## 2. Native subagent definition formats

Build from the role's `description` and body plus the chosen model and effort. The body carries the full role instructions (so the executor needs nothing else).

**Claude Code** — `.claude/agents/<role>.md`
```
---
name: <role>
description: <description>
model: <opus|sonnet|haiku or a full model ID>
effort: <low|medium|high|xhigh|max>
tools: <if access is read: Read, Grep, Glob, Bash (researcher adds WebFetch, WebSearch). Omit the line for write/run>
---
<body>
```

**Codex** — `.codex/agents/<role>.toml` (required: `name`, `description`, `developer_instructions`. Omitted fields inherit from the parent session)
```
name = "<role>"
description = "<description>"
model = "<model>"
model_reasoning_effort = "<low|medium|high|xhigh>"
sandbox_mode = "<read-only if access is read, else workspace-write>"
developer_instructions = """
<body>
"""
```
Before writing, confirm the installed Codex accepts this format (version, docs); if it differs, follow the actual format.

## 3. Model level

Map the role's `tier` (high/mid/low) to platform models. The table is the default suggestion; shift one step up or down by the chosen level (Balanced/Quality/Economy).

| Platform | high | mid | low |
|---|---|---|---|
| Claude Code | `sonnet` / `high` | `haiku` / `high` | `haiku` / `medium` |
| Others | Strongest coding model in the installed CLI's model list or settings / `high` | Mid model / `high` | Cheapest model / `medium` |

- Quality: high = Claude `opus` (or the platform's top model); mid and low also one step up.
- Economy: lower effort by one step each.
- If unsure of a model name, ask the user or leave it empty so it inherits from the parent session.

## 4. Delegation methods

| Method | What | Where available | Notes |
|---|---|---|---|
| `subagent` | Call a native subagent within the same session | Claude Code (Agent tool), Codex (subagents) | Default. Nothing to clean up |
| `session` | Open a separate chat session (child chat) in the app | Claude desktop/web (session start tool), Codex app (new thread, cloud task), etc. | Needs cleanup afterward. Has an approval rule |
| `cli` | Run another agent CLI headless | Any platform with a shell | Enables delegation across different agents (e.g. Claude lead → Codex worker) |
| `handoff` | Write a brief file; the user runs it in another tool | Anywhere | No automation. A person carries it over |

**`cli` command examples** (check installation with `command -v`; recheck options with `--help`)
- Claude Code: `claude -p "<brief>" --model <model>`
- Codex: `codex exec -m <model> "<brief>"` (add `--sandbox read-only` for read-only)
- Gemini CLI: `gemini -p "<brief>" -m <model>`

Run rules: write tasks run in a worktree dedicated to that executor, in the background, with output saved to `<dir>/team-lead/runs/<date-role-summary>.md` (`<dir>` = the lead's platform). If the brief is long, write it to a file and pass the path. The brief includes the role instructions (the body of that role's definition file).

**`handoff` rules**: the lead writes a brief (including role instructions) to `<dir>/team-lead/briefs/<date-role-summary>.md` and tells the user which tool to run it in. The executor leaves its report at the same name with `.report.md` appended.

**Fallback order**: if a role's method isn't available on the current platform, fall back `subagent` (role instructions as a brief to this platform's default subagent) → `cli` → `handoff`, and tell the user. Never fall back to `session` automatically.

`runs/` and `briefs/` are created when first used. Whether to commit them is the user's call (init's wrap-up may suggest adding them to `.gitignore` but doesn't change it).
