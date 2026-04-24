---
title: Backend Overview
parent: "[[00 - Backend (Index)]]"
project: 창신
created: 2026-04-23
status: draft
tags:
  - backend
  - nestjs
  - overview
  - 창신
---

# Backend Overview

> 상위 문서: [[00 - Backend (Index)]]

> [!summary] 한 줄로 말하면
> `weplanet-starter-nestjs` 보일러플레이트를 기반으로 **Auth + AX1/2/3 = 총 4개 서비스**를 구성. monorepo 구조를 그대로 활용해 `apps/` 아래에 각 서비스를 배치.

---

## 1. 보일러플레이트

**경로**: `/Users/yoohakseon/Documents/GitLab/weplanet/weplanet-starter-nestjs`

**특징**:

- NestJS **monorepo** (`apps/` 아래 여러 앱)
- 이미 포함된 샘플 앱:
  - `admin-api` — 관리자 API
  - `user-api` — 일반 사용자 API
  - `batch` — 배치 작업
  - `llm-api` — LLM API
  - `mcp-server` — MCP 서버
- 공통 패턴:
  - `controllers/` — 컨트롤러 + `dto/req/`, `dto/res/` 분리
  - `guards/` — `authentication.guard`, `authorization.guard`
  - `api-docs/` — Swagger YAML + Markdown 조합
- **Claude rules** 포함 (`.claude/rules/`):
  - `api-design.md` · `auth-security.md` · `tech-stack.md` · `typeorm-schema.md`
- **Claude skills** 포함 (`.claude/skills/`):
  - `add-controller`, `add-domain`, `docs-reviewer`, `skill-creator`, `skill-reviewer`

> [!tip] Claude rules/skills 재활용
> 보일러플레이트의 `.claude/` 구조는 이 Vault와 별개로 **프로젝트 저장소에 그대로 존재**한다. 새 서비스 저장소에서 Claude Code를 사용하면 해당 규칙/스킬이 자동 적용된다.

---

## 2. 서비스 파생 계획

보일러플레이트의 `admin-api` 또는 `user-api`를 템플릿 삼아 각 서비스로 분화.

| 서비스 | 역할 | 보일러플레이트의 Auth 사용 |
|--------|------|---------------------------|
| **Auth** | 인증·인가 중앙화, JWT 발급 (`aud` 포함) | **메인 로직** — Auth 컨트롤러/서비스를 그대로 활용 |
| **AX1 API** | AX1 비즈니스 로직 | 발급 X · JWT 검증만 (`aud=ax1` 수락) |
| **AX2 API** | **[[AX-2 지능형 스케줄러/00 - AX-2 쉬운 설명서 (Index)\|AX-2 지능형 스케줄러]] 전체 도메인 구현** — 납기 예측·생산 계획·물류·정산 4개 모듈 | 발급 X · JWT 검증만 (`aud=ax2`) |
| **AX3 API** | AX3 비즈니스 로직 | 발급 X · JWT 검증만 (`aud=ax3`) |

### 서비스 ↔ 도메인 매핑

| 서비스 | 구현할 도메인 문서 |
|--------|-------------------|
| AX2 API | [[AX-2 지능형 스케줄러/02 - Module 1 · 납기일 예측\|M1 · 납기일 예측]] · [[AX-2 지능형 스케줄러/03 - Module 2 · 생산 계획 관리\|M2 · 생산 계획]] · [[AX-2 지능형 스케줄러/04 - Module 3 · 물류 · 배차\|M3 · 물류]] · [[AX-2 지능형 스케줄러/05 - Module 4 · 구매 · 정산\|M4 · 구매/정산]] |
| AX1 API | (도메인 문서 작성 예정) |
| AX3 API | (도메인 문서 작성 예정) |
| Auth | 자체 도메인 (사용자 · 인증) |

---

## 3. 저장소 전략 (초안)

### 옵션 A: 서비스별 저장소 분리

```
changshin-auth/
changshin-ax1-api/
changshin-ax2-api/
changshin-ax3-api/
```

- 장점: CI/CD · 배포 · 권한이 서비스별 독립
- 단점: 공통 코드(공유 DB 엔티티 등) 관리 부담

### 옵션 B: 단일 monorepo (보일러플레이트 구조 확장)

```
changshin-backend/
└── apps/
    ├── auth/
    ├── ax1-api/
    ├── ax2-api/
    └── ax3-api/
```

- 장점: 공통 라이브러리·타입·DB 엔티티를 `libs/`에 두고 공유 용이
- 단점: CI 파이프라인에서 변경된 앱만 빌드하는 설정 필요

> [!question] 결정 필요
> 옵션 A/B 중 선택. 기본 권장은 **옵션 B(monorepo)** — 공유 DB 엔티티 관리·타입 공유가 편하다.
> 확정은 [[Infrastructure/31 - Decision Log]]에 추가.

---

## 4. 공통 라이브러리 구조 (옵션 B 기준)

```
changshin-backend/
├── apps/
│   ├── auth/
│   ├── ax1-api/
│   ├── ax2-api/
│   └── ax3-api/
└── libs/
    ├── common/              # 공통 유틸
    ├── auth/                # JWT 검증 · 가드 공통 모듈
    ├── database/            # TypeORM 설정 · 공유 엔티티
    └── api-docs/            # Swagger 공통 설정
```

- `libs/auth` 는 각 AX 서비스가 import해서 **JWT 검증·가드**만 재사용
- `libs/database` 는 공유 DB 연결 + `common.*` 스키마 엔티티
- 각 앱은 자기 스키마 엔티티(`ax1.*` 등)를 `apps/<svc>/src/entities/`에 유지

---

## 5. 빌드 · 실행 · 배포

### 로컬 개발

```bash
# monorepo 루트에서
npm install

# 개별 앱 실행
npm run start:dev -- --project auth
npm run start:dev -- --project ax1-api
```

### 컨테이너 이미지 (K8s 배포용)

- 앱당 별도 `Dockerfile` (또는 공통 Dockerfile + `APP` 인자)
- ECR에 앱별 레포지토리 push
  - `changshin/auth`
  - `changshin/ax1-api`
  - `changshin/ax2-api`
  - `changshin/ax3-api`

### K8s 배포

자세한 내용: [[Infrastructure/20 - AWS Deployment#4. K8s 리소스 설계 초안]]

---

## 6. 공통 기술 스택

> 보일러플레이트의 `.claude/rules/tech-stack.md`에 상세 스택이 명시되어 있다. 실제 코딩 시 해당 규칙을 따른다.

| 영역 | 채택 |
|------|------|
| 런타임 | Node.js + NestJS |
| 타입 체크 | TypeScript |
| DB ORM | TypeORM (보일러플레이트 convention) |
| API 문서 | Swagger (`api-docs/` YAML + MD) |
| 테스트 | Jest (`*.spec.ts`) |
| 린트 | ESLint (`eslint.config.mjs`) |
| 포맷터 | Prettier |
| CI | GitLab CI (`.gitlab-ci.yml`) |

---

## 7. 문서화 원칙

- **API 명세는 Swagger에 단일 소스**. 별도 Vault 문서에 OpenAPI 재기술 금지.
- **도메인 설계·정책**은 Vault에 작성(이 폴더) → 코드 변경 시 링크로 추적.
- 보일러플레이트의 `docs-reviewer` 스킬을 활용해 PR 시 문서 일관성 체크.

---

## 열린 질문

- [ ] 저장소 전략: 옵션 A vs 옵션 B
- [ ] 공유 라이브러리 범위 (auth 공통만 / DB 포함 / 유틸까지)
- [ ] 앱별 Dockerfile 분리 vs 공통 + 빌드 인자
- [ ] 배치(batch), llm-api, mcp-server 중 창신 프로젝트에 활용할 것 여부

---

> 다음: [[10 - Auth Strategy]]
