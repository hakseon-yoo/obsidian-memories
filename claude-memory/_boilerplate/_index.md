---
id: {{CREATED_ID}}
type: system
tags: [index, root, project]
status: active
created: {{CREATED_DATE}}
updated: {{CREATED_DATE}}
summary: {{PROJECT_NAME}} 프로젝트 볼트 진입점
---

# {{PROJECT_NAME}} — Project Vault Index

이 vault는 **{{PROJECT_NAME}} 프로젝트 한정** 작업물을 담습니다.
글로벌 영구 메모리는 메인 볼트(`/Users/yoohakseon/Documents/Obsidian Vault/claude-memory/`)에 있습니다.

## Routing

1. 메인 볼트 `_index.md` 먼저 (글로벌 컨텍스트)
2. 이 문서 (프로젝트 컨텍스트)
3. 카테고리 인덱스 → 개별 노트

## Categories

- [[00_Project/overview|Project Overview]]
- [[00_Project/rules|Project Rules]]
- [[00_Project/glossary|Project Glossary]]
- [[10_Specs/_specs_index|Specs Index]]
- [[20_Logs/_logs_index|Logs Index]]
- [[30_References/_references_index|References Index]]
- [[40_Decisions/_decisions_index|Decisions Index]]

---

## 최근 활동

```dataview
TABLE type, status, updated
FROM "10_Specs" OR "20_Logs" OR "30_References" OR "40_Decisions"
WHERE !contains(file.name, "_index")
SORT updated DESC
LIMIT 15
```

## Draft

```dataview
TABLE type, created
FROM "10_Specs" OR "20_Logs" OR "30_References" OR "40_Decisions"
WHERE status = "draft"
SORT created DESC
```

## 미해결 사항 (specs 중)

```dataview
TABLE summary, updated
FROM "10_Specs"
WHERE type = "spec" AND status = "active"
SORT updated DESC
```
