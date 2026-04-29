---
title: Backend Overview
parent: "[[00 - Backend (Index)]]"
project: 창신
created: 2026-04-23
updated: 2026-04-28
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
> `weplanet-starter-nestjs` 보일러플레이트 기반의 **단일 NestJS monorepo (`changshin-api`)**. `apps/ax-api`(API) + `apps/batch`(배치)의 두 워크로드가 같은 코드베이스를 공유하며, 도메인은 모듈 디렉터리로 분리.

> [!warning] 2026-04-28 변경
> 4 분리 서비스 (Auth + AX1/2/3) → 단일 통합 서비스로 변경됨 ([[Infrastructure/31 - Decision Log#D-019|D-019]]).

---

## 1. 보일러플레이트 (출발점)

**경로**: `/Users/yoohakseon/Documents/GitLab/weplanet/weplanet-starter-nestjs`

**특징**:

- NestJS **monorepo** (`apps/` 아래 여러 앱)
- 샘플 앱: `admin-api`, `user-api`, `batch`, `llm-api`, `mcp-server`
- 공통 패턴:
  - `controllers/` — 컨트롤러 + `dto/req/`, `dto/res/` 분리
  - `guards/` — `authentication.guard`, `authorization.guard`
  - `api-docs/` — Swagger YAML + Markdown
- `.claude/rules/` (`api-design`, `auth-security`, `tech-stack`, `typeorm-schema`) · `.claude/skills/` (`add-controller`, `add-domain`, `docs-reviewer` 등) 자산 포함

> [!tip] Claude rules/skills 재활용
> `changshin-api` 리포에는 보일러플레이트의 `.claude/` 자산이 그대로 복사되어, 해당 리포에서 Claude Code를 사용할 때 자동 적용된다.

---

## 2. 실제 리포 (`changshin-api`)

**경로**: `/Users/yoohakseon/Documents/GitLab/changshin/changshin-api`

**현재 구조**:

```
changshin-api/
├── apps/
│   ├── ax-api/                    # 통합 HTTP API (단일 서비스)
│   │   └── src/
│   │       ├── ax-api.module.ts
│   │       ├── ax-api.controller.ts  # 헬스체크 등 루트
│   │       ├── controllers/          # 도메인 컨트롤러들
│   │       ├── guards/               # authentication / authorization
│   │       ├── strategies/           # JWT strategy
│   │       └── api-docs/             # Swagger
│   ├── batch/                     # cron · 배치
│   │   └── src/
│   │       └── cron/                 # scheduler 들
│   └── ax2-api/                   # (legacy stub, 정리 예정)
├── packages/                      # 공유 패키지 (data/decorators/config 등)
└── k8s/clusters/                  # K8s 매니페스트 (D-013 하이브리드)
```

`changshin-legacy/` 폴더에 분리 시기의 4리포가 보관되어 있다 (`changshin-auth-api`, `changshin-ax2-api`, `changshin-ax3-api`).

---

## 3. 도메인 모듈 배치 (목표 구조)

`apps/ax-api/src/modules/` 또는 `apps/ax-api/src/controllers/` 아래에 도메인 모듈을 분리한다.

```
apps/ax-api/src/
├── ax-api.module.ts                  # 루트 모듈 (모든 도메인 모듈 import)
├── modules/                          # 또는 controllers/
│   ├── auth/                         # Auth 모듈
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── entities/                 # auth.* 스키마 엔티티
│   │   ├── dto/req/, dto/res/
│   │   └── migrations/
│   ├── ax1/                          # AX1 모듈
│   ├── ax2/                          # AX2 모듈 (M1~M4)
│   │   ├── due-date-prediction/        # M1 납기 예측
│   │   ├── production-planning/        # M2 생산 계획
│   │   ├── logistics/                  # M3 물류
│   │   └── settlement/                 # M4 구매·정산
│   └── ax3/                          # AX3 모듈
├── guards/                           # 공통 가드 (모든 모듈 공유)
├── strategies/                       # JWT strategy
└── api-docs/                         # Swagger
```

> [!info] 모듈 경계 보존
> 같은 프로세스에서 동작하지만, **DB 스키마 · 엔티티 · 마이그레이션 · DTO 모두 도메인별로 독립**. 향후 분리(MSA 회귀) 비용을 낮춘다.

---

## 4. 도메인 ↔ 외부 표면 매핑

| 도메인 | NestJS 모듈 위치 | DB 스키마 | API 라우팅 prefix(잠정) |
|--------|----------------|----------|------------------------|
| Auth | `apps/ax-api/src/modules/auth/` | `auth.*` | `/auth/*` |
| AX1 | `apps/ax-api/src/modules/ax1/` | `ax1.*` | `/api/ax1/*` |
| AX2 | `apps/ax-api/src/modules/ax2/` | `ax2.*` | `/api/ax2/*` |
| AX3 | `apps/ax-api/src/modules/ax3/` | `ax3.*` | `/api/ax3/*` |
| Common | `packages/` 또는 `apps/ax-api/src/common/` | `common.*` | (없음, 다른 모듈에서 사용) |

> Ingress path 단순화 여부는 [[Infrastructure/31 - Decision Log#미결정 · 논의 필요|D-019 후속]] 결정 대상.

---

## 5. 빌드 · 실행

### 로컬 개발

```bash
cd changshin-api
pnpm install

# API 실행
pnpm dev:ax-api

# 배치 실행
pnpm dev:batch
```

### 컨테이너 이미지 (K8s 배포용)

- 앱당 별도 `Dockerfile` (`apps/ax-api/Dockerfile`, `apps/batch/Dockerfile`)
- ECR push (현재는 4세트 ECR 잔재 존재 → D-019 후속 통폐합 예정)

### K8s 배포

자세한 내용: [[Infrastructure/20 - AWS Deployment#4. K8s 리소스 구성]]

---

## 6. 공통 기술 스택

> 보일러플레이트의 `.claude/rules/tech-stack.md`에 상세 스택 명시. 실제 코딩 시 해당 규칙을 따른다.

| 영역 | 채택 |
|------|------|
| 런타임 | Node.js + NestJS |
| 타입 체크 | TypeScript |
| 패키지 매니저 | **pnpm** (monorepo workspace) |
| DB ORM | TypeORM |
| API 문서 | Swagger (`api-docs/` YAML + MD) |
| 테스트 | Jest (`*.spec.ts`) |
| 린트 | ESLint (`eslint.config.mjs`) |
| 포맷터 | Prettier |
| CI | GitLab CI (`.gitlab-ci.yml`) |
| 배포 | Flux (GitOps) — [[Infrastructure/31 - Decision Log#D-005|D-005]] |

---

## 7. 문서화 원칙

- **API 명세는 Swagger에 단일 소스**. 별도 Vault 문서에 OpenAPI 재기술 금지.
- **도메인 설계·정책**은 Vault에 작성(이 폴더 + AX-2 폴더) → 코드 변경 시 링크로 추적.
- 보일러플레이트의 `docs-reviewer` 스킬을 활용해 PR 시 문서 일관성 체크.

---

## 8. 도메인 ↔ 서버 매핑

| 도메인 | 서버 |
|--------|------|
| AX-2 ([[AX-2 지능형 스케줄러/00 - AX-2 쉬운 설명서 (Index)\|쉬운 설명서]]) M1~M4 | **`changshin-api`의 `apps/ax-api` AX2 모듈** |
| AX1 | **`changshin-api`의 `apps/ax-api` AX1 모듈** (도메인 정의 후 구현) |
| AX3 | **`changshin-api`의 `apps/ax-api` AX3 모듈** (도메인 정의 후 구현) |
| Auth | **`changshin-api`의 `apps/ax-api` Auth 모듈** (legacy 자산 통합 진행 중) |
| 배치 (스케줄러 · 알림 · ERP 동기화) | **`changshin-api`의 `apps/batch`** |

---

## 열린 질문

> 모든 미완 항목은 [[00 - Action Board]] 에서 관리. 본 문서 관련:
> - `aud` 처리 / Ingress path 단순화 → [[00 - Action Board#A. changshin-api 통합 마무리 (D-019 후속)]]
> - 도메인 모듈 디렉터리 컨벤션 / 직접 import 허용 → [[00 - Action Board#📥 백로그 (다음 사이클 / 결정·답변 도착 시 진행)]]
> - AX1·AX3 도메인 정의 → [[00 - Action Board#⛔ 본인 책임 외 (AX1·AX3 담당자 위임)]]

---

> 다음: [[10 - Auth Strategy]]
