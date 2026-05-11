# Obsidian × Claude Code 통합 세팅 — Claude Code 핸드오프

> **Claude Code에게**: 이 문서는 claude.ai의 사전 설계 세션에서 만들어진 인수인계 문서다. 처음부터 끝까지 읽고, "Pending Decisions" 섹션의 질문 두 개를 사용자에게 먼저 묻고, 답을 받은 뒤에 "Implementation Tasks"를 순서대로 수행하라.

---

## 1. 목표

Obsidian을 Claude의 영구 메모리로 활용한다. 두 종류의 vault를 만든다:

1. **메인 볼트** (글로벌·영구) — 모든 프로젝트가 공유하는 지식·룰·용어집
2. **프로젝트 볼트** (프로젝트별) — 각 프로젝트 작업 노트

마지막에 새 프로젝트 시작 시 한 줄 명령으로 프로젝트 볼트가 자동 생성되도록 셸 스크립트까지 만든다.

---

## 2. 핵심 원칙

- **단일 진입점 → 카테고리 인덱스 → 실제 노트**: Claude는 vault 전체를 스캔하지 않는다. `_index.md` → `_xxx_index.md` → 개별 노트 순으로만 라우팅한다.
- **Frontmatter 표준화**: 모든 노트에 메타데이터를 박는다. 위반 금지.
- **Dataview로 인덱스 자동 생성**: 수동 인덱스 관리 안 함.
- **Templater로 표준 강제**: 새 노트는 반드시 템플릿 경유.
- **메인 볼트 우선, 프로젝트 볼트 그 다음**: Claude는 항상 글로벌 컨텍스트 먼저 본 뒤 프로젝트 컨텍스트를 본다.

---

## 3. Architecture: 2-Vault System

```
~/claude-memory/              ← 메인 볼트 (글로벌)
                                평생 누적되는 지식·룰
                                모든 프로젝트가 참조

your-project/vault/           ← 프로젝트 볼트 (프로젝트 내부)
                                이 프로젝트 한정 작업물
                                프로젝트와 함께 git에 들어감
```

연결 방식:
1. 프로젝트 루트의 `CLAUDE.md`에 "글로벌 메모리 경로: `~/claude-memory/`" 명시
2. Claude Code 세션 시작 → 프로젝트 `CLAUDE.md` → 메인 볼트 `_index.md` → 프로젝트 볼트 `_index.md` 순으로 컨텍스트 로딩
3. 영구 지식은 메인으로, 프로젝트 한정 작업은 프로젝트 볼트에

---

## 4. Pending Decisions (사용자에게 먼저 물을 것)

**Q1. 메인 볼트의 위치는 어디로?**

추천 선택지:
- `~/claude-memory/` — 짧고 직관적 (기본 추천)
- `~/Documents/Obsidian/main/` — Obsidian 통합 관리 시
- `~/Dropbox/claude-memory/` 또는 iCloud 경로 — 멀티 디바이스 사용 시

**Q2. 프로젝트 볼트의 폴더명은 어떻게?**

추천 선택지:
- `./vault/` — 가장 명시적 (개인 프로젝트 추천)
- `./docs/` — 팀원에게 자연스럽게 보임 (팀 프로젝트 추천)
- `./.claude-vault/` — 숨김 폴더로 코드 트리에서 안 보이게

이 둘을 받기 전엔 파일 생성 시작하지 말 것.

---

## 5. 메인 볼트 구조

```
~/claude-memory/  (Q1에서 받은 경로)
├── _index.md                       # 전역 진입점, Dataview 인덱스
├── CLAUDE.md                       # Claude 글로벌 지침
├── 00_System/
│   ├── rules.md                    # 사용자의 코딩 스타일, 작업 원칙
│   ├── schema.md                   # frontmatter 표준
│   └── glossary.md                 # 보편 용어
├── 10_Knowledge/
│   └── _knowledge_index.md         # 영구 지식 인덱스
├── 20_Resources/
│   └── _resources_index.md         # 외부 자료 인덱스
├── 30_Decisions/
│   └── _decisions_index.md         # 평생급 결정 (도구·스타일 선택 등)
├── 40_Projects/
│   └── _projects_registry.md       # 프로젝트 레지스트리 (메타만)
├── 90_Archive/
│   └── .gitkeep
└── _templates/
    ├── template-knowledge.md
    ├── template-resource.md
    ├── template-decision.md
    └── template-log.md
```

**각 폴더의 의미**:
- `00_System/`: vault 운영 규칙. Claude가 항상 참조.
- `10_Knowledge/`: 시간 지나도 유효한 지식 (Zettelkasten). 한 노트 = 한 개념.
- `20_Resources/`: 외부 자료의 본인 언어 요약. 출처 URL 필수.
- `30_Decisions/`: 도구 선정·스타일 결정 같은 평생급 ADR. 프로젝트 한정 결정은 여기 아님.
- `40_Projects/_projects_registry.md`: 각 프로젝트의 한 줄 요약·시작일·종료일·프로젝트 볼트 경로 링크.

---

## 6. 프로젝트 볼트 구조 (보일러플레이트)

이 구조는 **템플릿으로 보관**해두었다가 `claude-init` 스크립트가 새 프로젝트 시작 시 복사하도록 한다. 보관 위치 추천: `~/.local/share/claude-project-vault-template/`

```
your-project/vault/  (Q2에서 받은 폴더명)
├── _index.md                       # 이 프로젝트 진입점
├── CLAUDE.md                       # 메인 볼트 참조 룰 포함
├── 00_Project/
│   ├── overview.md                 # 프로젝트가 무엇인지
│   ├── rules.md                    # 이 프로젝트 한정 룰
│   └── glossary.md                 # 이 프로젝트 한정 용어
├── 10_Specs/
│   └── _specs_index.md             # 요구사항·설계·아키텍처 인덱스
├── 20_Logs/
│   └── _logs_index.md              # 작업·회의 로그 인덱스
├── 30_References/
│   └── _references_index.md        # 이 프로젝트용 외부 자료
├── 40_Decisions/
│   └── _decisions_index.md         # 이 프로젝트 한정 ADR
├── 90_Archive/
│   └── .gitkeep
└── _templates/
    ├── template-spec.md
    ├── template-log.md
    ├── template-decision.md
    └── template-reference.md
```

**프로젝트 볼트에 `10_Knowledge/`가 없는 이유**: 영구 지식은 메인 볼트로 승격되어야 하므로. 프로젝트 볼트는 작업 중 발생한 휘발성·문맥 의존적 노트만 담는다.

---

## 7. Frontmatter Schema (양쪽 vault 공통)

### 필수 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `id` | string | `YYYYMMDD-HHmmss` |
| `type` | enum | `system` / `rule` / `knowledge` / `resource` / `decision` / `log` / `spec` / `project` |
| `tags` | list | 소문자-하이픈 |
| `status` | enum | `draft` / `active` / `archived` |
| `created` | date | `YYYY-MM-DD` |
| `updated` | date | `YYYY-MM-DD` |
| `summary` | string | 1줄, 25자 내외 |

### 선택 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `aliases` | list | 별칭 |
| `related` | list | `[[링크]]` 리스트 |
| `source` | string | 출처 URL |
| `project` | string | 프로젝트 노트 링크 (메인 볼트의 `_projects_registry` 엔트리) |

---

## 8. 핵심 파일 콘텐츠 가이드

### 메인 볼트 `CLAUDE.md`

다음을 반드시 포함:
- 이 vault가 Claude의 글로벌 영구 메모리임을 명시
- 진입 규칙: `_index.md` 먼저, 그 다음 카테고리 인덱스
- 노트 작성 규칙: schema.md 준수, 템플릿 경유
- 금지: vault 전체 스캔, frontmatter 누락 노트 생성, archive 수정

### 메인 볼트 `_index.md`

다음 Dataview 쿼리를 포함:
- 전체 최근 활동 (15개)
- draft 상태 노트 (미정리)
- 진행 중 프로젝트 (40_Projects에서 status=active)

### 프로젝트 볼트 `CLAUDE.md`

핵심 룰:
```
1. 세션 시작 시 ~/claude-memory/_index.md를 먼저 읽어 글로벌 컨텍스트 로딩
2. 그 다음 이 vault의 _index.md를 읽어 프로젝트 컨텍스트 로딩
3. 영구 지식은 ~/claude-memory/10_Knowledge/에 작성, 이 vault에는 작성 금지
4. 프로젝트 한정 결정은 이 vault의 40_Decisions/에 ADR로
5. 평생급 결정(도구·스타일)은 ~/claude-memory/30_Decisions/에
```

### 프로젝트 볼트 `00_Project/overview.md`

새 프로젝트마다 사용자가 채워야 할 부분. 템플릿으로:
- 프로젝트명·시작일
- 목적·범위
- 주요 이해관계자
- 성공 기준

---

## 9. Templater 템플릿 형식

모든 템플릿은 다음 frontmatter를 포함:

```markdown
---
id: <% tp.date.now("YYYYMMDD-HHmmss") %>
type: [타입에 따라]
tags: []
status: draft
created: <% tp.date.now("YYYY-MM-DD") %>
updated: <% tp.date.now("YYYY-MM-DD") %>
summary: 
---

# <% tp.file.title %>

[타입에 맞는 섹션들]
```

타입별 본문 섹션:
- **knowledge**: Context / Content / References
- **resource**: 요약 / 핵심 내용 / 인용·발췌 / 관련
- **decision**: Context / Decision / Consequences
- **log**: 한 일 / 배운 것 / 다음에 할 것
- **spec**: 배경 / 요구사항 / 설계 / 미해결 사항

---

## 10. Implementation Tasks (순서대로)

1. **Pending Decisions 확인** — Q1, Q2를 사용자에게 묻고 답 받기.
2. **메인 볼트 디렉토리 생성** — Q1 경로에 섹션 5의 전체 트리 생성.
3. **메인 볼트 파일 작성** — `_index.md`, `CLAUDE.md`, `00_System/*`, 각 카테고리의 `_xxx_index.md` 작성. Dataview 쿼리는 섹션 8 참조.
4. **메인 볼트 템플릿 작성** — `_templates/` 안에 섹션 9의 형식대로.
5. **프로젝트 볼트 템플릿 디렉토리 생성** — `~/.local/share/claude-project-vault-template/`에 섹션 6의 트리 생성.
6. **프로젝트 볼트 파일 작성** — 동일 방식으로 `_index.md`, `CLAUDE.md` (메인 볼트 참조 룰 포함), `00_Project/*`, 카테고리 인덱스, 템플릿.
7. **`claude-init` 셸 스크립트 작성** — `~/.local/bin/claude-init` 또는 사용자의 셸 함수 파일(`~/.zshrc` / `~/.bashrc`)에 추가. 동작:
   - 인자: `<project-name>` (필수), `--path <경로>` (선택, 기본은 현재 디렉토리)
   - 프로젝트 디렉토리 생성
   - 프로젝트 볼트 템플릿을 Q2 폴더명으로 복사
   - 프로젝트 루트에 `CLAUDE.md` 작성 (메인 볼트 경로 + 프로젝트 볼트 경로 명시)
   - 메인 볼트의 `40_Projects/_projects_registry.md`에 신규 프로젝트 엔트리 추가 (id, 이름, 시작일, 경로)
   - 완료 메시지 출력 + 다음 단계 안내
8. **테스트** — 더미 프로젝트로 `claude-init test-project` 실행, 모든 파일이 제대로 생성되는지 확인.
9. **(선택) Obsidian MCP 서버 설치** — 사용자가 원하면 `mcp-obsidian` 같은 MCP 서버를 Claude Code에 등록. 메인 볼트에 read 권한 부여.

각 단계 종료 시마다 사용자에게 결과 보고하고 다음 단계로 넘어갈지 확인.

---

## 11. 첫 메시지 가이드

Claude Code는 이 문서를 다 읽은 뒤 사용자에게 다음과 같이 시작할 것:

> 이전 claude.ai 세션의 설계를 인수받았습니다. 2-volt 시스템(메인 볼트 + 프로젝트 볼트 보일러플레이트 + claude-init 스크립트)을 만들 예정입니다.
>
> 시작 전 두 가지만 확인해주세요:
> 1. 메인 볼트 경로? (추천: `~/claude-memory/`)
> 2. 프로젝트 볼트 폴더명? (추천: 개인이면 `./vault/`, 팀이면 `./docs/`)

답을 받으면 섹션 10의 Task 2부터 진행.
