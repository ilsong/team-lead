# /team-lead init — 프로젝트 셋업

목표: 현재 프로젝트(작업 디렉터리의 git 루트, 없으면 작업 디렉터리)에 팀장 운영에 필요한 파일을 만든다. 전역 설정은 건드리지 않는다.

만드는 것:

| 파일 | 내용 |
|---|---|
| `.claude/team-lead.md` | 이 프로젝트의 팀장 운영 규칙(에이전트 명단·빌드/테스트·브랜치·검증 규칙) |
| `.claude/agents/<이름>.md` | 고른 에이전트 정의(모델·추론 수준 고정) |
| `CLAUDE.md` 한 줄 | 자동 모드를 고르면 `@.claude/team-lead.md` 줄을 넣어 모든 세션이 팀장이 된다 |
| `.claude/skills/team-lead/` | (고르면) 이 스킬 사본. 팀원이 레포만 받아도 `/team-lead`를 쓴다 |

## 1. 살펴보기 (묻기 전에)

조용히 확인하고 질문의 기본값에 반영한다.
- `CLAUDE.md`, `.claude/team-lead.md`(있으면 **재실행**: 현재 값을 기본값으로 보여 주고 바뀐 것만 고친다), `.claude/agents/`, `.claude/settings.json`.
- 언어·빌드 도구·테스트 명령(`package.json` scripts, `Makefile`, `gradlew`, `pyproject.toml`, `Cargo.toml` 등), CI 설정, `git remote`, `gh` 사용 가능 여부, 기본 브랜치.
- UI가 있는지(웹·모바일·데스크톱) → 검증 에이전트 추천 근거.

## 2. 묻기

`AskUserQuestion`으로 묻는다(한 번에 최대 4문항, 필요하면 두 번). 추천안을 첫 선택지에 두고 "(추천)"을 붙인다. 살펴본 결과로 이미 분명한 것은 묻지 않고 기본값으로 둔다.

**1차**
1. **에이전트 구성**(복수 선택)
   - `worker` — 설계·구현·단위 테스트(항상 추천)
   - `reviewer` — 코드 검토(읽기 전용)
   - `researcher` — 코드·문서 조사, 넓은 탐색(읽기 전용, 싼 모델)
   - `verifier` — 실행 검증(테스트 전체, 브라우저, 에뮬레이터, 시뮬레이터 등). UI나 기기 검증이 있는 프로젝트면 추천
2. **모델 수준**
   - 균형(추천): worker·reviewer = `sonnet`/`high`, researcher = `haiku`/`medium`, verifier = `haiku`/`high`
   - 품질: worker·reviewer = `opus`/`high`, researcher = `sonnet`/`medium`, verifier = `sonnet`/`high`
   - 절약: worker·reviewer = `sonnet`/`medium`, researcher = `haiku`/`low`, verifier = `haiku`/`medium`
3. **팀장 켜는 방식**
   - assign 할 때만(추천): `/team-lead assign`을 실행한 세션만 팀장
   - 항상: `CLAUDE.md`에 `@.claude/team-lead.md`를 넣어 이 프로젝트의 모든 세션이 팀장
4. **작업 규칙**(복수 선택, 해당하는 것만 규칙 파일에 넣는다)
   - 브랜치 → PR → 사용자 승인 후 머지(기본 브랜치 직접 푸시 금지)
   - 쓰기 작업 병렬은 worktree로 격리
   - 검증은 백그라운드로 맡기고 기다리지 않음(머지는 빌드·단위 테스트 통과면 진행, 검증 오류는 후속 수정)
   - 팀원 공유: 이 스킬을 `.claude/skills/team-lead/`에 복사

**2차(필요할 때만)**
- `verifier`를 골랐으면: 검증 대상과 방법(예: 에뮬레이터 이름·시리얼, 브라우저 URL, 테스트 명령), 동시에 하나만 돌려야 하는 공유 자원인지, 보고서 위치.
- 빌드·테스트 명령을 못 찾았으면 묻는다.
- 같은 이름의 에이전트 파일이 이미 있으면: 덮어쓰기 / 건너뛰기 / 새 이름.

## 3. 쓰기

- 에이전트: 이 스킬 폴더의 `templates/agents/<이름>.md`를 바탕으로 `.claude/agents/<이름>.md`를 만든다. frontmatter의 `model`·`effort`를 고른 수준으로 바꾸고, `{{…}}` 자리표시를 채운다. 프로젝트에 필요 없는 줄은 지운다.
- 규칙: `templates/team-lead.md`를 바탕으로 `.claude/team-lead.md`를 만든다. 자리표시를 채우고, 고르지 않은 규칙 절은 지운다. 에이전트 명단 표는 실제로 만든 에이전트만 적는다.
- 자동 모드면 `CLAUDE.md`(없으면 만든다)에 아래를 넣는다. 이미 있으면 다시 넣지 않는다.
  ```
  ## 작업 방식
  @.claude/team-lead.md
  ```
  assign 모드면 `CLAUDE.md`는 건드리지 않는다. 재실행으로 자동 → assign으로 바꾸면 그 줄을 지운다.
- 팀원 공유를 골랐으면 이 스킬 폴더의 `SKILL.md`, `references/`, `templates/`, `README.md`, `LICENSE`를 `.claude/skills/team-lead/`로 복사한다(`.git` 제외).
- `.claude/settings.json`, `.gitignore`, 전역 파일은 바꾸지 않는다.

## 4. 마무리

- 만든·바꾼 파일 목록과 각 에이전트의 모델·추론 수준을 표로 보여 준다.
- 커밋은 하지 않는다. 팀원과 공유하려면 이 파일들을 커밋하라고 한 줄로 안내한다(`.claude/agents/`가 `.gitignore`에 걸려 있으면 그 사실도 알린다).
- 에이전트 정의는 **새 세션부터** 목록에 잡힐 수 있다고 알린다.
- 다음 단계: 새 세션에서 `/team-lead assign`(자동 모드면 그냥 작업 지시).
