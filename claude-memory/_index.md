---
id: 20260511-000001
type: system
tags: [index, root]
status: active
created: 2026-05-11
updated: 2026-05-11
summary: 메인 볼트 전역 진입점
---

# Main Vault — Global Index

이 vault는 **Claude의 글로벌 영구 메모리**입니다. 모든 프로젝트가 이곳을 참조합니다.

## Routing

진입 순서를 반드시 지킬 것:

1. 이 문서 (`_index.md`)
2. 카테고리 인덱스 (`10_Knowledge/_knowledge_index.md` 등)
3. 개별 노트

> vault 전체 스캔 금지. 항상 인덱스 경유.

## Categories

- [[00_System/rules|System Rules]] — 사용자 코딩 스타일·작업 원칙
- [[00_System/schema|Frontmatter Schema]] — 노트 메타데이터 표준
- [[00_System/glossary|Glossary]] — 보편 용어
- [[10_Knowledge/_knowledge_index|Knowledge Index]] — 영구 지식 (Zettelkasten)
- [[20_Resources/_resources_index|Resources Index]] — 외부 자료 요약
- [[30_Decisions/_decisions_index|Decisions Index]] — 평생급 ADR
- [[40_Projects/_projects_registry|Projects Registry]] — 프로젝트 레지스트리

---

## 최근 활동 (전체 15개)

```dataview
TABLE type, status, updated
FROM "10_Knowledge" OR "20_Resources" OR "30_Decisions" OR "40_Projects"
WHERE file.name != "_knowledge_index"
  AND file.name != "_resources_index"
  AND file.name != "_decisions_index"
  AND file.name != "_projects_registry"
SORT updated DESC
LIMIT 15
```

## Draft (미정리 노트)

```dataview
TABLE type, created
FROM "10_Knowledge" OR "20_Resources" OR "30_Decisions" OR "40_Projects"
WHERE status = "draft"
SORT created DESC
```

## 진행 중인 프로젝트

```dataview
TABLE summary, created
FROM "40_Projects"
WHERE type = "project" AND status = "active"
SORT created DESC
```
