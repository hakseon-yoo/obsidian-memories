---
id: 20260511-000020
type: system
tags: [index, knowledge]
status: active
created: 2026-05-11
updated: 2026-05-11
summary: 영구 지식 카테고리 인덱스
---

# Knowledge Index

> Zettelkasten. 한 노트 = 한 개념. 시간 지나도 유효한 것만.

## 최근 추가/수정

```dataview
TABLE tags, status, updated
FROM "10_Knowledge"
WHERE file.name != "_knowledge_index"
SORT updated DESC
LIMIT 30
```

## Draft

```dataview
TABLE tags, created
FROM "10_Knowledge"
WHERE status = "draft" AND file.name != "_knowledge_index"
SORT created DESC
```

## 태그별

```dataview
TABLE rows.file.link AS notes
FROM "10_Knowledge"
WHERE file.name != "_knowledge_index" AND status = "active"
FLATTEN tags AS tag
GROUP BY tag
SORT tag ASC
```
