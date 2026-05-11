---
id: 20260511-000022
type: system
tags: [index, decisions, adr]
status: active
created: 2026-05-11
updated: 2026-05-11
summary: 평생급 결정(ADR) 인덱스
---

# Decisions Index

> 도구 선정·스타일 등 **평생급** 결정만. 프로젝트 한정 결정은 해당 프로젝트 볼트의 `40_Decisions/`로.

## 모든 결정

```dataview
TABLE status, tags, updated
FROM "30_Decisions"
WHERE file.name != "_decisions_index"
SORT updated DESC
```

## Active만

```dataview
TABLE summary, updated
FROM "30_Decisions"
WHERE status = "active" AND file.name != "_decisions_index"
SORT updated DESC
```

## Archived

```dataview
TABLE summary, updated
FROM "30_Decisions"
WHERE status = "archived" AND file.name != "_decisions_index"
SORT updated DESC
```
