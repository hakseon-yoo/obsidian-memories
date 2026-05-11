# {{PROJECT_NAME}} — Project Vault · Claude 지침

이 vault는 **{{PROJECT_NAME}} 프로젝트 한정** 작업 노트입니다.
**메인 볼트**(`/Users/yoohakseon/Documents/Obsidian Vault/claude-memory/`)가 글로벌 영구 메모리입니다.

## 핵심 룰

1. 세션 시작 시 `/Users/yoohakseon/Documents/Obsidian Vault/claude-memory/_index.md`를 먼저 읽어 **글로벌 컨텍스트**를 로딩한다.
2. 그 다음 이 vault의 `_index.md`를 읽어 **프로젝트 컨텍스트**를 로딩한다.
3. **영구 지식**은 메인 볼트의 `10_Knowledge/`에 작성한다. 이 vault에는 작성 금지.
4. **프로젝트 한정 결정**은 이 vault의 `40_Decisions/`에 ADR로 기록한다.
5. **평생급 결정**(도구·스타일)은 메인 볼트의 `30_Decisions/`에 작성한다.

## 분류 가이드

| 종류 | 위치 |
|---|---|
| 프로젝트 개요·룰·용어 | `00_Project/` |
| 요구사항·설계·아키텍처 | `10_Specs/` |
| 작업·회의·세션 로그 | `20_Logs/` |
| 이 프로젝트용 외부 자료 | `30_References/` |
| 프로젝트 한정 ADR | `40_Decisions/` |
| 평생급 지식·결정 | **메인 볼트** |

## 노트 작성 규칙

- 모든 노트는 메인 볼트의 [[/Users/yoohakseon/Documents/Obsidian Vault/claude-memory/00_System/schema|Frontmatter Schema]]를 준수한다.
- 새 노트는 반드시 `_templates/`의 템플릿을 경유한다.
- 휘발성·문맥 의존적 노트만 이 vault에. 영구성은 메인으로 승격.

## 금지

- vault 전체 스캔
- frontmatter 누락 노트 생성
- `90_Archive/` 수정
- 영구 지식을 이 vault에 작성 (→ 메인 볼트로 승격)
