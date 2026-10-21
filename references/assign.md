# assign — make this session the team lead

## 1. Load the rules

1. Read this platform's rules file (`platforms.md` §1, e.g. `.claude/team-lead/rules.md`). If it is missing, read another platform's rules file in the project. These are the operating rules for this session. If only a legacy file exists (`.team-lead/rules.md` or `.claude/team-lead.md`), read it and add one line at the end saying that re-running init migrates it to the new layout.
2. If none exist, use this skill's `templates/rules.md` as the default rules. Fill the placeholders sensibly from the project, and delegate to this platform's default subagent with the role instructions from `templates/roles/` attached as the brief (if there are no subagents, use the fallback order in `platforms.md` §4). Add one line at the end saying init can tailor the setup to the project.
3. Check the current platform and whether each role in the rules' role table can actually be delegated here.
   - `subagent`: whether this platform's definition file exists (`platforms.md` §1).
   - `cli`: whether that CLI is installed (`command -v`).
   - For roles that can't be used, pick a substitute by the fallback order and mark it in the roster.

## 2. Announce

From now until the session ends, work as the team lead under the loaded rules. Tell the user briefly:
- One line saying the team lead is assigned
- A roster table: role · responsibilities · method · executor (model/effort) — mark substituted roles
- Up to 3 lines of key rules (e.g. delegate detailed work, merge only after approval, child chat sessions only after approval)

## 3. Start

- If a task came with the argument, decompose and dispatch it right away.
- Otherwise, ask for the first task. If a task is already in progress in the conversation, ask whether to continue it as team lead.
