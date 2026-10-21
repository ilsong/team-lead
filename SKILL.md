---
name: team-lead
description: Run the main session as a team lead (orchestrator) that hands detailed work to subagents, child chat sessions, or other agent CLIs. Works with Claude Code, Codex, and other agents. `init` adds team lead instructions and role agents at the scope the user picks (global, project, or project-personal); `assign` makes the current chat session the team lead. Use only when the user invokes this skill with init or assign.
argument-hint: init | assign [first task]
disable-model-invocation: true
---

# team-lead

Pick the mode from the first word after the skill name (Claude Code passes it in `$ARGUMENTS`; other agents read it from the user's message).

- `init` → read `references/init.md` in the same folder as this SKILL.md and follow it.
- `assign` → read `references/assign.md` in the same folder and follow it. Any text after the first word is the first task given to the team lead.
- Anything else (including no argument) → describe both modes in one line each and ask which one.

Common:
- Follow `references/platforms.md` for anything that differs by platform (Claude Code, Codex, etc.).
- Reply in the language the user is using, and write project files in that language too (templates are in English; translate when needed).
- Write only the instruction files and agent definitions the user chose in init. Touch home-directory files only when the user picks the global scope.
