# Claude Code에 Vercel·Supabase MCP 연동하기

## MCP란?

MCP(Model Context Protocol)는 Claude Code가 Vercel, Supabase 같은 외부 서비스의 정보와 기능을 사용할 수 있게 해 주는 연결 방식이다.

- **Vercel MCP**: 프로젝트, 배포, 로그 확인
- **Supabase MCP**: DB, 로그, 마이그레이션, Edge Functions 관리

## 설정 범위

| 범위 | 설명 |
|---|---|
| `local` | 현재 프로젝트에서 나만 사용하며 기본값이다. |
| `project` | `.mcp.json`에 저장하여 팀과 공유한다. |
| `user` | 내 모든 프로젝트에서 사용한다. |

아래 예시는 팀에서 공유할 수 있도록 `project` 범위를 사용한다.

## Vercel 연결

프로젝트 루트에서 실행한다.

```bash
claude mcp add --transport http --scope project vercel https://mcp.vercel.com
```

Claude Code를 실행하고 `/mcp`에서 `vercel`을 선택한 뒤 브라우저 인증을 진행한다.

```bash
claude
```

```text
/mcp
```

연결 후 다음과 같이 요청할 수 있다.

```text
최근 Vercel 배포 목록과 실패 로그를 확인해 줘.
```

## Supabase 연결

Supabase Dashboard에서 **Project Ref**를 확인한 뒤 `YOUR_PROJECT_REF`를 실제 값으로 변경한다.

처음에는 특정 프로젝트에 읽기 전용으로 연결하는 것이 안전하다.

```bash
claude mcp add --transport http --scope project supabase "https://mcp.supabase.com/mcp?project_ref=YOUR_PROJECT_REF&read_only=true&features=database,debugging,docs"
```

Claude Code의 `/mcp`에서 `supabase`를 선택하고 브라우저 인증을 진행한다.

연결 후 다음과 같이 요청할 수 있다.

```text
Supabase의 테이블 목록과 컬럼을 조회해 줘. 변경은 하지 마.
```

DB 변경이나 Edge Function 배포가 필요하면 `read_only=true`를 제거하고 필요한 기능을 추가한다.

```bash
claude mcp remove --scope project supabase

claude mcp add --transport http --scope project supabase "https://mcp.supabase.com/mcp?project_ref=YOUR_PROJECT_REF&features=database,debugging,development,functions,docs"
```

## 확인 및 삭제

```bash
# 연결 목록
claude mcp list

# 상세 정보
claude mcp get vercel
claude mcp get supabase

# 연결 삭제
claude mcp remove --scope project vercel
claude mcp remove --scope project supabase
```

## 핵심 주의사항

- 공식 주소인지 확인한다.
  - Vercel: `https://mcp.vercel.com`
  - Supabase: `https://mcp.supabase.com/mcp`
- Supabase는 운영 프로젝트보다 개발 프로젝트에 연결한다.
- 가능하면 `project_ref`와 `read_only=true`를 사용한다.
- SQL 변경, 마이그레이션, 배포 작업은 실행 전에 내용을 확인한다.
- API 키나 액세스 토큰을 `.mcp.json`과 Git에 저장하지 않는다.

## 공식 문서

- [Claude Code MCP](https://code.claude.com/docs/en/mcp)
- [Vercel MCP](https://vercel.com/docs/agent-resources/vercel-mcp)
- [Supabase MCP](https://supabase.com/docs/guides/ai-tools/mcp)
