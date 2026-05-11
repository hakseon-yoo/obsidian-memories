---
id: 20260511-000021
type: system
tags: [index, resources]
status: active
created: 2026-05-11
updated: 2026-05-11
summary: 외부 자료 요약 인덱스
---

# Resources Index

> 외부 자료의 본인 언어 요약. 출처 URL(`source`) 필수.

## 최근 추가/수정

```dataview
TABLE source, tags, updated
FROM "20_Resources"
WHERE file.name != "_resources_index"
SORT updated DESC
LIMIT 30
```

## Draft

```dataview
TABLE source, created
FROM "20_Resources"
WHERE status = "draft" AND file.name != "_resources_index"
SORT created DESC
```

## 태그별

```dataview
TABLE rows.file.link AS notes
FROM "20_Resources"
WHERE file.name != "_resources_index" AND status = "active"
FLATTEN tags AS tag
GROUP BY tag
SORT tag ASC
```
