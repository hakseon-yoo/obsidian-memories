---
id: {{CREATED_ID}}
type: system
tags: [index, specs]
status: active
created: {{CREATED_DATE}}
updated: {{CREATED_DATE}}
summary: 요구사항·설계·아키텍처 인덱스
---

# Specs Index

## 최근 추가/수정

```dataview
TABLE tags, status, updated
FROM "10_Specs"
WHERE file.name != "_specs_index"
SORT updated DESC
LIMIT 30
```

## Active

```dataview
TABLE summary, updated
FROM "10_Specs"
WHERE type = "spec" AND status = "active" AND file.name != "_specs_index"
SORT updated DESC
```

## Draft

```dataview
TABLE summary, created
FROM "10_Specs"
WHERE status = "draft" AND file.name != "_specs_index"
SORT created DESC
```
