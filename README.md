# team-lead

[Claude Code](https://claude.com/claude-code)에서 **메인 세션을 팀장(오케스트레이터)으로 두고, 세부 작업은 서브에이전트에 맡기는** 운영 방식을 프로젝트마다 셋업하는 스킬입니다.

```
/team-lead init     # 이 프로젝트에 에이전트와 운영 규칙을 셋업
/team-lead assign   # 지금 채팅 세션을 팀장으로 지정
```

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
- 각 에이전트의 모델·추론 수준은 정의 파일에 고정되어, 매번 지정하지 않아도 일정한 비용·품질로 돕니다.
- 머지는 항상 사용자 승인 후. 서브에이전트 보고는 diff·테스트로 확인한 뒤 사용자에게 전달합니다.

## 설치

**개인(내 모든 프로젝트에서 `/team-lead` 쓰기)**

```bash
git clone https://github.com/ilsong/claude-team-lead.git ~/.claude/skills/team-lead
```

설치해도 아무 프로젝트도 바뀌지 않습니다. `/team-lead init`을 실행한 프로젝트만 셋업됩니다.

**팀 공유(레포를 받은 팀원 누구나)**

`/team-lead init`에서 "팀원 공유"를 고르면 스킬 사본이 프로젝트의 `.claude/skills/team-lead/`에 들어갑니다. 만들어진 파일을 커밋하면 팀원은 따로 설치하지 않아도 됩니다.

직접 넣으려면:

```bash
git clone --depth 1 https://github.com/ilsong/claude-team-lead.git .claude/skills/team-lead && rm -rf .claude/skills/team-lead/.git
```

**갱신**

```bash
git -C ~/.claude/skills/team-lead pull
```

## 사용법

### `/team-lead init` — 프로젝트 셋업

프로젝트를 먼저 살펴본 뒤(언어, 빌드·테스트 명령, CI, git 원격, UI 유무) 몇 가지를 묻고 셋업합니다.

1. **에이전트 구성** — worker / reviewer / researcher / verifier 중 고르기
2. **모델 수준**

   | 수준 | worker·reviewer | researcher | verifier |
   |---|---|---|---|
   | 균형(추천) | sonnet / high | haiku / medium | haiku / high |
   | 품질 | opus / high | sonnet / medium | sonnet / high |
   | 절약 | sonnet / medium | haiku / low | haiku / medium |

3. **팀장 켜는 방식** — `assign` 할 때만(추천) / 이 프로젝트의 모든 세션
4. **작업 규칙** — 브랜치→PR→승인 후 머지, worktree 병렬, 검증 비동기, 팀원 공유
5. verifier를 고르면 검증 대상·방법(에뮬레이터 시리얼, URL, 테스트 명령 등)을 추가로 묻습니다.

만들어지는 파일(모두 프로젝트 안):

| 파일 | 내용 |
|---|---|
| `.claude/team-lead.md` | 이 프로젝트의 팀장 운영 규칙 |
| `.claude/agents/*.md` | 고른 에이전트 정의 |
| `CLAUDE.md`에 `@.claude/team-lead.md` | "모든 세션" 방식을 골랐을 때만 |
| `.claude/skills/team-lead/` | "팀원 공유"를 골랐을 때만 |

- `~/.claude/` 아래 전역 설정·전역 에이전트·전역 CLAUDE.md는 건드리지 않습니다. 커밋도 하지 않습니다.
- 다시 실행하면 현재 값을 기본값으로 보여 주고 바뀐 것만 고칩니다.
- 새 에이전트는 **새 세션부터** 목록에 잡힐 수 있습니다.

### `/team-lead assign` — 이 세션을 팀장으로

`.claude/team-lead.md`를 읽어 지금 세션을 팀장으로 지정하고, 쓸 수 있는 에이전트 명단을 보여 줍니다. 뒤에 작업을 붙이면 바로 시작합니다.

```
/team-lead assign 로그인 화면에 소셜 로그인 추가
```

init을 하지 않은 프로젝트에서도 기본 규칙과 내장 `general-purpose` 에이전트로 동작합니다.

## 구조

```
SKILL.md                  # 진입점: 인자로 init / assign 분기
references/init.md        # init 절차
references/assign.md      # assign 절차
templates/team-lead.md    # 운영 규칙 템플릿
templates/agents/*.md     # 에이전트 템플릿 (worker, reviewer, researcher, verifier)
```

규칙이나 에이전트를 바꾸고 싶으면 프로젝트에 만들어진 파일을 직접 고치거나, 이 레포의 템플릿을 고친 뒤 `/team-lead init`을 다시 실행하세요.

## 라이선스

MIT
