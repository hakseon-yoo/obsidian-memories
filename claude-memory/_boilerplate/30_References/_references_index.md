---
id: {{CREATED_ID}}
type: system
tags: [index, references]
status: active
created: {{CREATED_DATE}}
updated: {{CREATED_DATE}}
summary: 이 프로젝트용 외부 자료 인덱스
---

# References Index

> 이 프로젝트 작업에 쓰인 외부 자료. 보편적·재사용 가치 있는 자료는 **메인 볼트** `20_Resources/`로 승격.

## 최근 추가/수정

```dataview
TABLE source, tags, updated
FROM "30_References"
WHERE file.name != "_references_index"
SORT updated DESC
LIMIT 30
```

## 태그별

```dataview
TABLE rows.file.link AS notes
FROM "30_References"
WHERE file.name != "_references_index" AND status = "active"
FLATTEN tags AS tag
GROUP BY tag
SORT tag ASC
```
