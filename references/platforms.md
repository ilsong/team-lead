# 플랫폼과 위임 방법

init·assign이 참고하는 표. 여기에 없는 플랫폼은 그 플랫폼의 현재 문서·도움말(`<cli> --help`)로 같은 항목을 확인하고, 확인하지 못한 것은 사용자에게 묻는다. 모르는 형식을 지어내지 않는다.

## 1. 플랫폼별 위치

| 플랫폼 | 프로젝트 지침 파일 | 스킬 위치(프로젝트) | 네이티브 서브에이전트 정의 | 질문 도구 |
|---|---|---|---|---|
| Claude Code | `CLAUDE.md` (`@경로` 가져오기 가능) | `.claude/skills/` | `.claude/agents/<이름>.md` | `AskUserQuestion` |
| Codex | `AGENTS.md` (가져오기 없음, 경로를 적어 읽게 함) | `.agents/skills/` | `.codex/agents/<이름>.toml` | 있으면 그 도구, 없으면 번호 목록 |
| Gemini CLI | `GEMINI.md` | `.agents/skills/` | 확인 필요 | 번호 목록 |
| Cursor·Copilot·OpenCode·Cline 등 | `AGENTS.md` | `.agents/skills/` | 확인 필요 | 번호 목록 |

질문 도구가 없으면 질문을 번호와 선택지 기호로 한 메시지에 모아 내고(예: `1) a. … b. …`), `1a 2bc`처럼 답하게 한 뒤 답을 기다린다.

## 2. 네이티브 서브에이전트 정의 형식

역할 정본(`.team-lead/roles/<역할>.md`)의 `description`·본문과 고른 모델·추론으로 만든다. 본문은 정본을 그대로 옮긴다(실행자가 정본 파일을 못 읽어도 되게).

**Claude Code** — `.claude/agents/<역할>.md`
```
---
name: <역할>
description: <description>
model: <opus|sonnet|haiku 또는 전체 모델 ID>
effort: <low|medium|high|xhigh|max>
tools: <access가 read면 Read, Grep, Glob, Bash (researcher는 + WebFetch, WebSearch). write·run이면 줄 생략>
---
<본문>
```

**Codex** — `.codex/agents/<역할>.toml` (필수: `name`, `description`, `developer_instructions`. 나머지는 생략하면 부모 세션 값을 따른다)
```
name = "<역할>"
description = "<description>"
model = "<모델>"
model_reasoning_effort = "<low|medium|high|xhigh>"
sandbox_mode = "<access가 read면 read-only, 아니면 workspace-write>"
developer_instructions = """
<본문>
"""
```
쓰기 전에 설치된 Codex가 이 형식을 받는지(버전·문서) 확인하고, 다르면 실제 형식을 따른다.

## 3. 모델 수준

역할 정본의 `tier`(상/중/하)를 플랫폼 모델로 옮긴다. 아래는 기본 제안이고, 사용자가 고른 수준(균형/품질/절약)에 따라 한 칸씩 올리거나 내린다.

| 플랫폼 | 상 | 중 | 하 |
|---|---|---|---|
| Claude Code | `sonnet` / `high` | `haiku` / `high` | `haiku` / `medium` |
| 그 밖 | 설치된 CLI의 모델 목록·설정에서 가장 강한 코딩 모델 / `high` | 중간 모델 / `high` | 가장 싼 모델 / `medium` |

- 품질: 상 = Claude `opus`(또는 그 플랫폼 최상위), 중·하도 한 단계씩 위.
- 절약: 추론을 한 단계씩 낮춘다.
- 모델 이름을 확신할 수 없으면 사용자에게 묻거나 비워 두어 부모 세션 값을 따르게 한다.

## 4. 위임 방법

| 방법 | 무엇 | 쓸 수 있는 곳 | 비고 |
|---|---|---|---|
| `subagent` | 같은 세션 안에서 네이티브 서브에이전트 호출 | Claude Code(Agent 도구), Codex(서브에이전트) | 기본값. 정리할 것이 없다 |
| `session` | 앱에서 별도 채팅 세션(하위 채팅)을 연다 | Claude 데스크톱·웹(세션 시작 도구), Codex 앱(새 스레드·클라우드 작업) 등 | 끝난 뒤 정리가 필요하다. 승인 규칙을 둔다 |
| `cli` | 다른 에이전트 CLI를 헤드리스로 실행 | 셸이 있는 모든 플랫폼 | 서로 다른 에이전트 간 위임(예: Claude 팀장 → Codex 작업자)이 된다 |
| `handoff` | 브리프 파일을 쓰고 사용자가 다른 도구에서 실행 | 어디서나 | 자동화 없음. 사람이 옮긴다 |

**`cli` 명령 예** (설치 여부는 `command -v`로 확인. 옵션은 `--help`로 다시 확인)
- Claude Code: `claude -p "<브리프>" --model <모델>`
- Codex: `codex exec -m <모델> "<브리프>"` (읽기 전용이면 `--sandbox read-only`)
- Gemini CLI: `gemini -p "<브리프>" -m <모델>`

실행 규칙: 쓰기 작업은 그 실행자 전용 worktree에서, 백그라운드로 돌리고 출력은 `.team-lead/runs/<날짜-역할-요약>.md`에 남긴다. 브리프가 길면 파일로 쓰고 경로를 넘긴다.

**`handoff` 규칙**: 팀장이 `.team-lead/briefs/<날짜-역할-요약>.md`에 브리프(역할 지침 포함)를 쓰고 사용자에게 어느 도구에서 실행할지 알린다. 실행자는 같은 이름에 `.report.md`를 붙여 보고를 남긴다.

**대체 순서**: 역할에 정한 방법을 지금 플랫폼에서 못 쓰면 `subagent`(이 플랫폼 기본 서브에이전트에 역할 지침을 브리프로) → `cli` → `handoff` 순으로 내려가고 사용자에게 알린다. `session`으로는 자동으로 내려가지 않는다.

`.team-lead/runs/`·`.team-lead/briefs/`는 쓸 때 만든다. 커밋할지는 사용자가 정한다(init 마무리에서 `.gitignore` 추가를 권할 수 있으나 직접 바꾸지 않는다).
