# 프롬프트로 Claude Code Skill 만들기

## Skill이란?

Skill은 Claude Code에서 반복해서 사용할 **지식, 규칙 또는 작업 절차**를 저장한 기능이다.

`SKILL.md`를 직접 작성할 수도 있지만, Claude Code에 원하는 작업을 자연어로 설명하여 Skill 생성을 요청할 수 있다.

```text
요구사항 작성 → Claude가 Skill 생성 → 생성 경로 확인
```

## 1. Claude Code 실행

Skill을 적용할 프로젝트 루트에서 Claude Code를 실행한다.

```bash
claude
```

## 2. 프롬프트로 Skill 생성 요청

프롬프트에는 다음 내용을 구체적으로 작성한다.

- Skill의 목적
- 언제 사용해야 하는지
- 작업 순서
- 결과 형식
- 프로젝트용인지 개인용인지
- 실행하면 안 되는 작업 또는 확인이 필요한 작업

### 기본 프롬프트 예시

```text
현재 프로젝트에서 사용할 Claude Code Skill을 만들어 줘.

목적: Git 변경사항을 분석하고 핵심 내용을 요약한다.
사용 시점: 사용자가 변경사항 정리나 커밋 메시지를 요청할 때 사용한다.
작업 순서:
1. git status와 git diff를 확인한다.
2. 변경 내용을 파일별로 요약한다.
3. 테스트 누락과 위험 요소를 확인한다.
4. 커밋 메시지 후보를 제안한다.

결과는 변경 요약, 주의사항, 커밋 메시지 순서로 작성해 줘.
커밋과 푸시는 사용자의 명시적인 요청 없이는 실행하지 마.
프로젝트용 Skill로 생성하고 생성된 파일 경로를 알려 줘.
```

Claude Code는 요청을 분석한 뒤 일반적으로 다음 위치에 Skill을 생성한다.

```text
.claude/skills/<skill-name>/SKILL.md
```

모든 프로젝트에서 개인적으로 사용하려면 프롬프트에 다음 조건을 추가한다.

```text
이 Skill을 현재 프로젝트 전용이 아니라 모든 프로젝트에서 사용할 수 있는 개인용 Skill로 만들어 줘.
```

개인용 Skill은 일반적으로 다음 위치에 생성된다.

```text
~/.claude/skills/<skill-name>/SKILL.md
```

## 핵심 정리

- 반복해서 사용하는 작업 절차는 Skill로 만든다.
- Skill의 목적, 사용 시점, 순서, 결과 형식을 프롬프트에 명확히 적는다.
- 프로젝트용과 개인용 중 저장 범위를 지정한다.
- 커밋, 배포, 삭제 등 영향이 큰 작업에는 사용자 확인 조건을 넣는다.

[Claude Code Skills 공식 문서](https://code.claude.com/docs/en/slash-commands)
