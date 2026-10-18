---
name: team-lead
description: 메인 세션을 팀장(오케스트레이터)으로 두고 세부 작업은 서브에이전트·하위 채팅·다른 에이전트 CLI 등에 맡기는 운영 방식. Claude Code, Codex 등 여러 에이전트에서 쓴다. `init`은 현재 프로젝트에 역할과 운영 규칙을 셋업하고, `assign`은 지금 채팅 세션을 팀장으로 지정한다. 사용자가 이 스킬을 init 또는 assign과 함께 부를 때만 쓴다.
argument-hint: init | assign [첫 작업]
disable-model-invocation: true
---

# team-lead

사용자가 스킬 이름 뒤에 붙인 첫 단어로 모드를 고른다(Claude Code는 `$ARGUMENTS`, 다른 에이전트는 사용자 메시지에서 읽는다).

- `init` → 이 SKILL.md와 같은 폴더의 `references/init.md`를 읽고 그대로 따른다.
- `assign` → 같은 폴더의 `references/assign.md`를 읽고 그대로 따른다. 첫 단어 뒤에 남은 글이 있으면 팀장으로서 받은 첫 작업으로 다룬다.
- 그 밖(빈 인자 포함) → 두 모드를 한 줄씩 설명하고 어느 쪽인지 묻는다.

공통:
- 플랫폼(Claude Code, Codex 등)마다 다른 것은 `references/platforms.md`를 따른다.
- 사용자와 대화하는 언어로 답하고, 프로젝트에 쓰는 파일도 그 언어로 쓴다(템플릿은 한국어이므로 필요하면 옮긴다).
- 이 스킬은 **현재 프로젝트 안**만 바꾼다. 홈 디렉터리의 전역 설정·전역 에이전트·전역 지침 파일은 건드리지 않는다.
