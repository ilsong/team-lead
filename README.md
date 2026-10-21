# team-lead

An [Agent Skill](https://agentskills.io) that makes your main agent session work as a **team lead**: it designs, breaks down, dispatches, and reviews, while subagents, child chat sessions, or other agent CLIs do the detailed work. Works with Claude Code, Codex, and any agent that reads skills.

```
team-lead init     # Add team lead instructions and role agents (asks where)
team-lead assign   # Make the current chat session the team lead
```

| Agent | How to invoke |
|---|---|
| Claude Code | `/team-lead init`, `/team-lead assign` |
| Codex | `$team-lead init`, `$team-lead assign` |
| Others | Say the name and mode, e.g. "run the team-lead skill with init" |

## Roles

| Role | Does |
|---|---|
| Team lead (main session) | Design, task breakdown, parallel dispatch, review, integration |
| `worker` | Implementation and unit tests |
| `reviewer` | Code review (read-only) |
| `researcher` | Code/doc research (read-only, cheap model) |
| `verifier` | Runs tests, browser, emulator checks |

Delegation methods, chosen per role: subagent (default), child chat session, another agent CLI (e.g. Claude lead → `codex exec` reviewer), or manual handoff (the brief is shown in chat for you to run elsewhere). No method writes record files.

## Install

```bash
npx skills add ilsong/team-lead -g
```

Add `-a claude-code -a codex` to pick agents. Or clone:

```bash
git clone https://github.com/ilsong/team-lead.git ~/.claude/skills/team-lead   # Claude Code
git clone https://github.com/ilsong/team-lead.git ~/.agents/skills/team-lead   # Codex and others
```

Installing changes nothing until you run `init`.

## `init`

Asks:
1. Platforms — Claude Code / Codex / Gemini CLI / other
2. Roles — worker / reviewer / researcher / verifier
3. Delegation methods, and which role uses which
4. Model level — Balanced / Quality / Economy (Claude Code Balanced: worker·reviewer `sonnet`, verifier·researcher `haiku`)
5. When the lead is on — only after `assign` / every session
6. Where to record it, per platform:

   | Scope | Claude Code | Codex | Gemini CLI |
   |---|---|---|---|
   | Project-personal | `CLAUDE.local.md` | — | — |
   | Project | `CLAUDE.md` | `AGENTS.md` | `GEMINI.md` |
   | Global | `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` | `~/.gemini/GEMINI.md` |

Then, for each chosen platform, it appends a short team lead block to that instruction file and creates one agent definition per role (`.claude/agents/`, `.codex/agents/`, or the global folders for global scope). Nothing else changes, and nothing is committed. Re-running replaces the block.

## `assign`

Reads the team lead block, checks which roles are usable on this platform (falls back subagent → CLI → handoff), shows the roster, and starts working as lead. Add a task to start right away:

```
/team-lead assign Add social login to the login screen
```

Without init, it uses the default block and the platform's default subagent.

## Structure

```
SKILL.md                  # Entry point: init / assign
references/init.md        # init procedure
references/assign.md      # assign procedure
references/platforms.md   # File locations, agent formats, models, delegation methods
templates/block.md        # Team lead block appended to instruction files
templates/roles/*.md      # Role templates
```

## License

MIT
