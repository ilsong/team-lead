---
name: team-lead
description: 메인 세션을 팀장(오케스트레이터)으로 두고 세부 작업은 서브에이전트에 맡기는 운영 방식. `init`은 현재 프로젝트에 에이전트와 운영 규칙을 셋업하고, `assign`은 지금 채팅 세션을 팀장으로 지정한다.
argument-hint: init | assign
disable-model-invocation: true
---

# team-lead

인자(`$ARGUMENTS`)의 첫 단어로 모드를 고른다.

- `init` → 이 SKILL.md와 같은 폴더의 `references/init.md`를 읽고 그대로 따른다.
- `assign` → 같은 폴더의 `references/assign.md`를 읽고 그대로 따른다. 첫 단어 뒤에 남은 글이 있으면 팀장으로서 받은 첫 작업으로 다룬다.
- 그 밖(빈 인자 포함) → 두 모드를 한 줄씩 설명하고 어느 쪽인지 묻는다.

공통:
- 사용자와 대화하는 언어로 답하고, 프로젝트에 쓰는 파일도 그 언어로 쓴다(템플릿은 한국어이므로 필요하면 옮긴다).
- 이 스킬은 **현재 프로젝트 안**만 바꾼다. `~/.claude/` 아래 전역 설정·전역 에이전트·전역 CLAUDE.md는 건드리지 않는다.
