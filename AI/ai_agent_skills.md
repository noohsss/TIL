# Claude Code Skill 만들기

## Skill이란?

Skill은 Claude Code가 반복해서 사용할 **지식, 규칙 또는 작업 절차**를 `SKILL.md`에 저장한 것이다.

- 관련 요청이 들어오면 Claude가 자동으로 불러올 수 있다.
- `/<skill-name>` 형식으로 직접 실행할 수 있다.
- 반복해서 설명하는 작업 방식이나 체크리스트를 재사용할 때 유용하다.

## 저장 위치

| 범위 | 경로 |
| --- | --- |
| 현재 프로젝트 | `.claude/skills/<skill-name>/SKILL.md` |
| 모든 프로젝트에서 개인 사용 | `~/.claude/skills/<skill-name>/SKILL.md` |

팀과 공유하려면 프로젝트 경로를 사용하고 Git에 포함한다.

## 기본 구조

예를 들어 변경사항을 요약하는 Skill은 다음과 같이 만든다.

```text
.claude/
└── skills/
    └── summarize-changes/
        └── SKILL.md
```

```markdown
---
name: summarize-changes
description: 변경된 파일을 확인하고 핵심 내용과 위험 요소를 요약할 때 사용한다.
---

# 작업 절차

1. Git 변경사항을 확인한다.
2. 변경 목적을 파일별로 요약한다.
3. 테스트 누락이나 보안 위험을 확인한다.
4. 커밋 메시지 후보를 제안한다.
```

## 작성 원칙

- `description`에 Skill을 언제 사용해야 하는지 명확히 적는다.
- 하나의 Skill은 하나의 목적에 집중한다.
- 실행 순서와 완료 조건을 구체적으로 작성한다.
- 긴 참고자료, 예제, 스크립트는 별도 파일로 나누고 `SKILL.md`에서 연결한다.
- 삭제, 배포, 커밋처럼 영향이 큰 작업에는 확인 절차를 포함한다.

Claude Code를 실행한 뒤 자연어로 관련 작업을 요청하거나 다음과 같이 직접 테스트한다.

```text
/summarize-changes
```

[Claude Code Skills 공식 문서](https://code.claude.com/docs/en/slash-commands)
