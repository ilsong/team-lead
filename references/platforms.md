# Platforms and delegation methods

For platforms not listed here, confirm the same items from that platform's docs or `<cli> --help`, and ask the user about anything you can't confirm. Don't invent formats.

## 1. Locations

| Platform | Global instruction file | Project | Project-personal | Agent definitions (global / project) | Question tool |
|---|---|---|---|---|---|
| Claude Code | `~/.claude/CLAUDE.md` | `CLAUDE.md` | `CLAUDE.local.md` | `~/.claude/agents/` / `.claude/agents/` | `AskUserQuestion` |
| Codex | `~/.codex/AGENTS.md` | `AGENTS.md` | — | `~/.codex/agents/` / `.codex/agents/` | That tool if available, else numbered list |
| Gemini CLI | `~/.gemini/GEMINI.md` | `GEMINI.md` | — | Needs confirmation | Numbered list |
| Others | Check docs | `AGENTS.md` | — | Needs confirmation | Numbered list |

Without a question tool, put the questions in one message with numbers and option letters (e.g. `1) a. … b. …`), have the user answer like `1a 2bc`, and wait.

## 2. Agent definition formats

Built from the role template's `description` and body plus the chosen model and effort.

**Claude Code** — `agents/<role>.md`
```
---
name: <role>
description: <description>
model: <opus|sonnet|haiku or a full model ID>
effort: <low|medium|high|xhigh|max>
tools: <if access is read: Read, Grep, Glob, Bash (researcher adds WebFetch, WebSearch). Omit the line otherwise>
---
<body>
```

**Codex** — `agents/<role>.toml` (omitted fields inherit from the parent session)
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
Confirm the installed Codex accepts this format first; if it differs, follow the actual format.

## 3. Model level

Map the role's `tier` to models. Quality shifts one step up (high = `opus` or the platform's top model); Economy lowers effort one step.

| Platform | high | mid | low |
|---|---|---|---|
| Claude Code | `sonnet` / `high` | `haiku` / `high` | `haiku` / `medium` |
| Others | Strongest coding model / `high` | Mid model / `high` | Cheapest model / `medium` |

If unsure of a model name, ask or leave it empty to inherit from the parent session.

## 4. Delegation methods

None of these write record files.

| Method | How |
|---|---|
| `subagent` | Call the role's agent definition within the session (Claude Code Agent tool, Codex subagents) |
| `session` | Open a separate chat session in the app (Claude desktop/web, Codex app). Follows the opening rule chosen in init |
| `cli` | Run another agent headless, in the background for long tasks, and read its output directly. Pass the role's instructions in the brief. Claude Code `claude -p "<brief>" --model <m>`, Codex `codex exec -m <m> "<brief>"` (`--sandbox read-only` for read-only roles), Gemini `gemini -p "<brief>" -m <m>`. Check with `command -v` and `--help` |
| `handoff` | Show the brief (with role instructions) in chat; the user runs it in another tool and pastes the report back |
