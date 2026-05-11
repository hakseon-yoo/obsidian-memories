---
id: 20260511-000023
type: system
tags: [index, projects, registry]
status: active
created: 2026-05-11
updated: 2026-05-11
summary: 프로젝트 레지스트리 (메타만)
---

# Projects Registry

> 각 프로젝트의 한 줄 요약·시작일·종료일·프로젝트 볼트 경로.
> 프로젝트 본문 작업은 프로젝트 볼트에. 여기는 메타만.

## Active

```dataview
TABLE summary, created, file.frontmatter.path AS path
FROM "40_Projects"
WHERE type = "project" AND status = "active"
SORT created DESC
```

## Draft

```dataview
TABLE summary, created
FROM "40_Projects"
WHERE type = "project" AND status = "draft"
SORT created DESC
```

## Archived

```dataview
TABLE summary, created, updated
FROM "40_Projects"
WHERE type = "project" AND status = "archived"
SORT updated DESC
```

---

## 신규 프로젝트 엔트리 형식

`claude-init` 스크립트가 이 폴더에 다음 형식으로 노트를 추가합니다:

```yaml
---
id: <YYYYMMDD-HHmmss>
type: project
tags: [project]
status: active
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
summary: <한 줄 요약>
path: <프로젝트 절대 경로>
vault_path: <프로젝트 볼트 절대 경로>
---

# <프로젝트명>

<간단 설명>
```
