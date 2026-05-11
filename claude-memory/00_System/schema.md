---
id: 20260511-000011
type: system
tags: [system, schema, frontmatter]
status: active
created: 2026-05-11
updated: 2026-05-11
summary: 모든 노트의 frontmatter 표준
---

# Frontmatter Schema

> 양쪽 vault(메인·프로젝트) 공통. 위반한 노트는 생성 금지.

## 필수 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `id` | string | `YYYYMMDD-HHmmss` |
| `type` | enum | `system` / `rule` / `knowledge` / `resource` / `decision` / `log` / `spec` / `project` |
| `tags` | list | 소문자-하이픈, 예: `[react, hooks]` |
| `status` | enum | `draft` / `active` / `archived` |
| `created` | date | `YYYY-MM-DD` |
| `updated` | date | `YYYY-MM-DD` |
| `summary` | string | 1줄, 25자 내외 |

## 선택 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `aliases` | list | 별칭 |
| `related` | list | `[[링크]]` 리스트 |
| `source` | string | 출처 URL (resource 타입에서 필수) |
| `project` | string | 프로젝트 노트 링크 (`40_Projects/_projects_registry` 엔트리) |

## 예시

```yaml
---
id: 20260511-143022
type: knowledge
tags: [react, hooks, useeffect]
status: active
created: 2026-05-11
updated: 2026-05-11
summary: useEffect 의존성 배열의 동작 원리
related:
  - [[20_Resources/react-docs-effects]]
---
```

## type별 의미

- **system**: vault 운영 룰·스키마
- **rule**: 사용자 작업 원칙
- **knowledge**: 영구 유효한 개념 (한 노트 = 한 개념)
- **resource**: 외부 자료의 본인 언어 요약
- **decision**: ADR — Context / Decision / Consequences
- **log**: 작업·회의·세션 로그
- **spec**: 요구사항·설계
- **project**: `40_Projects/_projects_registry` 엔트리
