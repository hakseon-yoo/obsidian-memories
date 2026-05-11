---
id: {{CREATED_ID}}
type: system
tags: [index, logs]
status: active
created: {{CREATED_DATE}}
updated: {{CREATED_DATE}}
summary: 작업·회의 로그 인덱스
---

# Logs Index

## 최근 로그

```dataview
TABLE tags, created
FROM "20_Logs"
WHERE file.name != "_logs_index"
SORT created DESC
LIMIT 30
```

## 이번 주

```dataview
TABLE summary, created
FROM "20_Logs"
WHERE file.name != "_logs_index" AND created >= date(today) - dur(7 days)
SORT created DESC
```

## 태그별

```dataview
TABLE rows.file.link AS notes
FROM "20_Logs"
WHERE file.name != "_logs_index"
FLATTEN tags AS tag
GROUP BY tag
SORT tag ASC
```
