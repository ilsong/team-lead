# init — project setup

Goal: create the files needed to run a team lead in the current project (the git root of the working directory, or the working directory if there is none). Home-directory files (`~/.claude/`, `~/.codex/`, `~/.agents/`, etc.) stay untouched, except the global instruction file when the user picks that scope. Follow `platforms.md` in the same folder for per-platform formats. `<dir>` below is each platform's folder (`platforms.md` §1: `.claude`, `.codex`, ...).

What it creates:

| File | When | Contents |
|---|---|---|
| `<dir>/team-lead/rules.md` | For each chosen platform | Operating rules (roster, delegation methods, build/test, branches, verification) |
| `.claude/agents/<role>.md` | Claude Code chosen | Claude subagent definitions (one per role) |
| `.codex/agents/<role>.toml` | Codex chosen | Codex subagent definitions (one per role) |
| `<dir>/team-lead/roles/<role>.md` | Platforms without native subagent definitions | Role instructions |
| A line in the instruction file | "Always on" chosen | Link that makes every session read the rules. The file depends on the chosen scope (global / project / project-personal) |
| `.claude/skills/team-lead/`, `.agents/skills/team-lead/` | "Share with team" chosen | Copy of this skill |

## 1. Survey (before asking)

Check quietly and use the results as defaults for the questions.
- Existing setup: `<dir>/team-lead/` (if present this is a **re-run**: show current values as defaults and change only what differs), `.claude/agents/`, `.codex/agents/`, and lines in instruction files that point to the rules (all scopes in `platforms.md` §1).
- Legacy layouts: `.team-lead/` (previous version) or `.claude/team-lead.md` (oldest). If present, migrate in §3.
- Traces of platforms in use: `CLAUDE.md`·`.claude/`, `AGENTS.md`·`.codex/`, `GEMINI.md`·`.gemini/`, `.cursor/`, etc. Installed CLIs: `command -v claude codex gemini`.
- The platform running this init (yourself).
- Language, build tools, test commands (`package.json` scripts, `Makefile`, `gradlew`, `pyproject.toml`, `Cargo.toml`, etc.), CI config, `git remote`, whether `gh` is available, default branch.
- Whether there is a UI (web, mobile, desktop) → grounds for recommending the verifier role.

## 2. Ask

Ask with the platform's question tool, or as a numbered list if it has none (`platforms.md` §1). Put the recommended option first and mark it "(Recommended)". Don't ask what the survey already made clear; use it as the default. At most 4 questions at a time; use several rounds if needed.

**Round 1**
1. **Platforms** (multi) — agents used as lead and executors in this project. Pre-check from traces and installed CLIs; just ask for confirmation. E.g. Claude Code / Codex / Gemini CLI / other (enter a name).
2. **Roles** (multi)
   - `worker` — design, implementation, unit tests (always recommended)
   - `reviewer` — code review (read-only)
   - `researcher` — code/doc research, broad search (read-only, cheap model)
   - `verifier` — runtime verification (full tests, browser, emulator, etc.). Recommend if there is a UI or device to verify
3. **Delegation methods** (multi, `platforms.md` §4)
   - Subagent (Recommended) — delegate within the same session, nothing to clean up
   - Child chat session — open a separate chat in the app
   - Other agent CLI — e.g. a Claude lead runs workers or reviewers through `codex exec`
   - Manual handoff — write a brief file; the user runs it in another tool
4. **Model level** — Balanced (Recommended) / Quality / Economy (`platforms.md` §3)

**Round 2 (depending on answers)**
- If two or more methods were chosen: **method and executor per role**. Default: every role = subagent (same platform as the lead). If other agent CLIs were chosen, which role runs on which CLI (e.g. reviewer = Codex — a different model reviewing reduces blind spots).
- If child chat sessions were chosen: **opening rule** — ask the user every time (Recommended) / free at the lead's discretion. Also what they are for (e.g. long independent tasks, tasks the user wants to watch).
- **How the lead turns on** — only on `assign` (Recommended) / always (every session in this project is the lead).
- If always: **where to record it** (`platforms.md` §1) — project-personal (Recommended, e.g. `CLAUDE.local.md`: only you, only this project) / project (e.g. `CLAUDE.md`: everyone working on this repo) / global (e.g. `~/.claude/CLAUDE.md`: all your projects that have this setup). Leave out scopes the chosen platform doesn't support.
- **Work rules** (multi; only chosen ones go into the rules file) — branch → PR → merge after approval / isolate parallel write tasks in worktrees / run verification in the background without waiting / share with team (copy the skill into the project).
- If `verifier` was chosen: verification targets and methods (emulator name/serial, browser URL, test command, etc.), whether it is a shared resource that must run one at a time, where reports go.
- If build/test commands weren't found, ask.
- If a file with the same name already exists: overwrite / skip / new name.

## 3. Write

- **Migrate legacy layout**: move the contents of `.team-lead/rules.md` or `.claude/team-lead.md` into the new rules file(s), and `.team-lead/runs/`·`.team-lead/briefs/` into the lead platform's `<dir>/team-lead/`. Change instruction-file lines that point to the old paths to the new path. Delete the old files only after the user confirms.
- **Rules**: copy `templates/rules.md` to each chosen platform's `<dir>/team-lead/rules.md`. The copies are identical except `{{ROLE_FILES}}`; on re-run keep them in sync.
  - `{{ROLE_ROWS}}`: only the roles actually created, `| role | responsibilities | method | platform · model / effort |`.
  - `{{ROLE_FILES}}`: where that platform's role instructions live (`.claude/agents/<role>.md`, `.codex/agents/<role>.toml`, or `<dir>/team-lead/roles/<role>.md`).
  - `{{METHODS}}`: one paragraph per chosen method — when to use it, the command or tool, approval rules (bold the chosen rule for child chat sessions), and a project-specific condensed version of the run rules and fallback order from `platforms.md` §4. If child chat sessions weren't chosen, add one line: "Open separate chat sessions only when the user asks."
  - Fill the remaining placeholders and delete sections for rules that weren't chosen.
- **Roles**: fill `{{…}}` in this skill's `templates/roles/<role>.md`, delete lines the project doesn't need, then write them in each chosen platform's format.
  - Platforms with native subagents: definition files in the `platforms.md` §2 format for **every** role (also used as the fallback when a `cli` role's CLI isn't available, and as the brief for `cli`/`handoff`).
  - Platforms without: `<dir>/team-lead/roles/<role>.md` (keep the template as is, including frontmatter).
- **Always on**: add the line from `platforms.md` §1 to the instruction file for the chosen scope (create the file if missing). Don't add it twice. In assign mode don't touch it; when switching always → assign or changing scope, remove the line from the old file.
- **Share with team**: copy this skill folder's `SKILL.md`, `references/`, `templates/`, `README.md`, `LICENSE` to the chosen platforms' skill locations (`platforms.md` §1) as `team-lead/` (excluding `.git`).
- Don't change settings files (`.claude/settings.json`, `.codex/config.toml`, etc.), `.gitignore`, or global files other than the chosen global instruction file. If a feature needs a setting to turn on (e.g. a Codex version with subagents off), just mention it in the wrap-up.

## 4. Wrap up

- Show a table of created/changed files and each role's method, executor, model/effort.
- Don't commit. In one line, say to commit these files to share with the team (and mention if a created path is caught by `.gitignore`). If project-personal scope was chosen and that file isn't in `.gitignore`, say to add it. If `cli` or `handoff` was chosen, say to decide whether to exclude `<dir>/team-lead/runs/` and `<dir>/team-lead/briefs/` from commits.
- Say subagent definitions may only be picked up **from a new session**.
- Next step: `team-lead assign` in a new session (or just give a task if always on). Invocation differs by platform (Claude Code `/team-lead assign`, Codex `$team-lead assign`).
