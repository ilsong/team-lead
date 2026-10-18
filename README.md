# team-lead

메인 세션을 **팀장(오케스트레이터)** 으로 두고, 세부 작업은 서브에이전트·하위 채팅·다른 에이전트 CLI에 맡기는 운영 방식을 프로젝트마다 셋업하는 [Agent Skill](https://agentskills.io)입니다. Claude Code, Codex를 비롯해 스킬을 읽는 에이전트에서 씁니다.

```
team-lead init     # 이 프로젝트에 역할·위임 방법·운영 규칙을 셋업 (묻고 정함)
team-lead assign   # 지금 채팅 세션을 팀장으로 지정
```

| 에이전트 | 부르는 법 |
|---|---|
| Claude Code | `/team-lead init`, `/team-lead assign` |
| Codex | `$team-lead init`, `$team-lead assign` |
| 그 밖 | "team-lead 스킬로 init 해줘"처럼 이름과 모드를 말하기 |

## 왜 쓰나

혼자 모든 걸 하는 세션은 대화가 길어질수록 맥락이 차고, 비싼 모델로 단순 구현까지 하게 됩니다. team-lead는 역할을 나눕니다.

| 누가 | 하는 일 |
|---|---|
| 팀장(메인 세션) | 설계, 작업 분해, 병렬 배분, 결과 검토, 통합(PR·머지 승인 요청) |
| `worker` | 구현·단위 테스트 |
| `reviewer` | 코드 검토(읽기 전용) |
| `researcher` | 코드·문서 조사(읽기 전용, 싼 모델) |
| `verifier` | 실행 검증(테스트, 브라우저, 에뮬레이터 등) — 백그라운드로 돌리고 기다리지 않음 |

- 팀장은 서로 의존하지 않는 작업을 찾아 **한꺼번에 병렬로** 배분하고, 파일을 쓰는 병렬 작업은 worktree로 격리합니다.
- 역할마다 모델·추론 수준이 고정되어 매번 지정하지 않아도 일정한 비용·품질로 돕니다.
- 머지는 항상 사용자 승인 후. 실행자의 보고는 diff·테스트로 확인한 뒤 사용자에게 전달합니다.

## 위임 방법

init에서 고르고, 역할마다 다르게 정할 수 있습니다.

| 방법 | 무엇 | 예 |
|---|---|---|
| 서브에이전트(기본) | 같은 세션 안에서 위임. 정리할 것이 없음 | Claude Code `.claude/agents/`, Codex `.codex/agents/` |
| 하위 채팅 세션 | 앱에서 별도 채팅을 열어 위임. 여는 규칙(매번 승인 / 자유)을 정함 | Claude 데스크톱 세션, Codex 앱 스레드 |
| 다른 에이전트 CLI | 헤드리스로 다른 에이전트를 실행. 서로 다른 에이전트끼리 협업 | Claude 팀장 → `codex exec`로 검토 |
| 수동 핸드오프 | 팀장이 브리프 파일을 쓰고 사람이 다른 도구에서 실행 | 어디서나 |

## 설치

**개인(내 모든 프로젝트에서 쓰기)** — [`skills` CLI](https://github.com/vercel-labs/skills)로 쓰는 에이전트를 골라 설치합니다.

```bash
npx skills add ilsong/team-lead -g
```

에이전트를 지정하려면 `-a claude-code -a codex`처럼 붙입니다. git으로 직접 넣어도 됩니다.

```bash
git clone https://github.com/ilsong/team-lead.git ~/.claude/skills/team-lead   # Claude Code
git clone https://github.com/ilsong/team-lead.git ~/.agents/skills/team-lead   # Codex 등
```

설치해도 아무 프로젝트도 바뀌지 않습니다. `init`을 실행한 프로젝트만 셋업됩니다.

**팀 공유(레포를 받은 팀원 누구나)** — `init`에서 "팀원 공유"를 고르면 스킬 사본이 프로젝트의 `.claude/skills/team-lead/`(Claude Code)·`.agents/skills/team-lead/`(Codex 등)에 들어갑니다. 만들어진 파일을 커밋하면 팀원은 따로 설치하지 않아도 됩니다. 직접 넣으려면 프로젝트 루트에서:

```bash
npx skills add ilsong/team-lead --copy
```

**갱신**: 설치한 명령을 다시 실행하거나, git으로 설치했으면 그 폴더에서 `git pull`.

## 사용법

### `init` — 프로젝트 셋업

프로젝트를 먼저 살펴본 뒤(쓰는 에이전트 흔적, 설치된 CLI, 언어, 빌드·테스트 명령, CI, git 원격, UI 유무) 플랫폼의 질문 UI로(없으면 번호 목록으로) 묻고 셋업합니다.

1. **플랫폼** — Claude Code / Codex / Gemini CLI / 그 밖
2. **역할** — worker / reviewer / researcher / verifier
3. **위임 방법** — 서브에이전트 / 하위 채팅 세션 / 다른 에이전트 CLI / 수동 핸드오프
4. **모델 수준** — 균형(추천) / 품질 / 절약

   | 수준 | Claude Code 기본 |
   |---|---|
   | 균형 | worker·reviewer `sonnet`/high, verifier `haiku`/high, researcher `haiku`/medium |
   | 품질 | 한 단계 위(worker·reviewer `opus`) |
   | 절약 | 추론 한 단계 아래 |

   다른 플랫폼은 설치된 CLI의 모델 목록에서 같은 수준으로 제안하고, 모르면 묻습니다.
5. 고른 것에 따라 더 묻습니다 — 역할별 방법·실행자(예: reviewer는 Codex), 하위 채팅을 여는 규칙, 팀장 켜는 방식(`assign` 할 때만 / 항상), 작업 규칙(브랜치→PR→승인 후 머지, worktree 병렬, 검증 비동기, 팀원 공유), 검증 대상, 빌드·테스트 명령.

만들어지는 파일(모두 프로젝트 안):

| 파일 | 내용 |
|---|---|
| `.team-lead/rules.md` | 운영 규칙 정본(에이전트 무관) |
| `.team-lead/roles/*.md` | 역할 지침 정본 |
| `.claude/agents/*.md`, `.codex/agents/*.toml` | 플랫폼별 서브에이전트 정의(쓰는 플랫폼만) |
| `CLAUDE.md`·`AGENTS.md`·`GEMINI.md`의 한 줄 | "항상" 방식을 골랐을 때만 |
| `.claude/skills/team-lead/`, `.agents/skills/team-lead/` | "팀원 공유"를 골랐을 때만 |

- 홈 디렉터리의 전역 설정·에이전트·지침 파일은 건드리지 않습니다. 커밋도 하지 않습니다.
- 다시 실행하면 현재 값을 기본값으로 보여 주고 바뀐 것만 고칩니다. 예전 형식(`.claude/team-lead.md`)은 새 형식으로 옮깁니다.
- 새 서브에이전트는 **새 세션부터** 목록에 잡힐 수 있습니다.

### `assign` — 이 세션을 팀장으로

`.team-lead/rules.md`를 읽어 지금 세션을 팀장으로 지정하고, 지금 플랫폼에서 쓸 수 있는 실행자 명단을 보여 줍니다. 규칙에 정한 실행자를 못 쓰면(예: Codex에서 assign했는데 Claude 서브에이전트만 정의됨) 서브에이전트 → CLI → 핸드오프 순으로 대체하고 알려 줍니다. 뒤에 작업을 붙이면 바로 시작합니다.

```
/team-lead assign 로그인 화면에 소셜 로그인 추가
```

init을 하지 않은 프로젝트에서도 기본 규칙과 플랫폼 기본 서브에이전트로 동작합니다.

## 구조

```
SKILL.md                  # 진입점: init / assign 분기
references/init.md        # init 절차
references/assign.md      # assign 절차
references/platforms.md   # 플랫폼별 파일 위치·정의 형식·모델·위임 방법
templates/rules.md        # 운영 규칙 템플릿
templates/roles/*.md      # 역할 템플릿 (worker, reviewer, researcher, verifier)
```

규칙이나 역할을 바꾸려면 프로젝트에 만들어진 파일을 직접 고치거나, 템플릿을 고친 뒤 `init`을 다시 실행하세요. 새 플랫폼을 지원하려면 `references/platforms.md`에 줄을 더하면 됩니다.

## 라이선스

MIT
