---
id: {{CREATED_ID}}
type: system
tags: [index, decisions, adr]
status: active
created: {{CREATED_DATE}}
updated: {{CREATED_DATE}}
summary: 이 프로젝트 한정 ADR 인덱스
---

# Decisions Index

> 이 프로젝트 한정 결정만. 도구·스타일 등 평생급 결정은 메인 볼트 `30_Decisions/`로.

## 모든 결정

```dataview
TABLE status, tags, updated
FROM "40_Decisions"
WHERE file.name != "_decisions_index"
SORT updated DESC
```

## Active

```dataview
TABLE summary, updated
FROM "40_Decisions"
WHERE status = "active" AND file.name != "_decisions_index"
SORT updated DESC
```

## Archived

```dataview
TABLE summary, updated
FROM "40_Decisions"
WHERE status = "archived" AND file.name != "_decisions_index"
SORT updated DESC
```
