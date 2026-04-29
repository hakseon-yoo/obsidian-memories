---
title: Service Template (Single API)
parent: "[[00 - Backend (Index)]]"
project: 창신
created: 2026-04-23
updated: 2026-04-28
status: draft
tags:
  - backend
  - template
  - nestjs
  - 창신
---

# Service Template · 통합 API 구성

> 상위 문서: [[00 - Backend (Index)]]
> 이전: [[10 - Auth Strategy]]

> [!summary] 한 줄로 말하면
> `changshin-api` 단일 NestJS monorepo에 **`apps/ax-api`(API) + `apps/batch`(배치)** 두 워크로드를 두고, 도메인은 `apps/ax-api/src/modules/{auth,ax1,ax2,ax3}/`로 분리한다.

> [!warning] 2026-04-28 변경
> 4개 분리 앱(auth/ax1-api/ax2-api/ax3-api) → 단일 통합 앱으로 변경 ([[Infrastructure/31 - Decision Log#D-019|D-019]]).

> [!note] 2026-04-28 추가 결정
> - 도메인 패키지(`@data/domain`) 를 트랙별 디렉토리(`ax/ax2/ax3/shared/`) 로 분리 ([[Infrastructure/31 - Decision Log#D-020|D-020]])
> - 트랙 간 공유 마스터(`shared/sku/`) 첫 입주: SKU 마스터 5 엔티티 ([[Infrastructure/31 - Decision Log#D-021|D-021]])
> - 본 문서의 `apps/ax-api/src/modules/{auth,ax1,ax2,ax3}/` 구조 제안은 **이전 안**이며, 실제 구조는 다음과 같음:
>   - `apps/ax-api/src/controllers/<resource>/` (HTTP 핸들러 + DTO 만)
>   - `packages/data/domain/src/{ax|ax2|ax3|shared}/<resource>/` (Service + Entity + Module)
>   - 컨벤션 상세: 리포의 `.claude/rules/coding-conventions.md`

---

## 1. 보일러플레이트 앱 구조 (참고)

`weplanet-starter-nestjs/apps/user-api/` 등을 참조해 도메인별 코드 배치 컨벤션을 적용:

```
apps/<app>/
├── src/
│   ├── app.ts                            # bootstrap
│   ├── <app>.controller.ts               # 루트 (헬스체크)
│   ├── <app>.module.ts                   # 루트 모듈
│   ├── api-docs/                         # Swagger
│   ├── controllers/                      # 또는 modules/
│   │   ├── <domain>/
│   │   │   ├── <domain>.controller.ts
│   │   │   ├── <domain>.service.ts
│   │   │   ├── <domain>-http.module.ts
│   │   │   └── dto/
│   │   │       ├── req/
│   │   │       └── res/
│   ├── guards/                           # authentication / authorization
│   └── strategies/                       # JWT
├── eslint.config.mjs
├── jest.config.ts
├── nest-cli.json
└── package.json
```

---

## 2. `changshin-api` 실제 구조 (현재 + 목표)

### 2-1. 현재 (2026-04-28)

```
changshin-api/
├── apps/
│   ├── ax-api/
│   │   └── src/
│   │       ├── ax-api.module.ts
│   │       ├── ax-api.controller.ts
│   │       ├── controllers/              # 현재는 enrichment / market 등 잔재
│   │       ├── guards/
│   │       ├── strategies/
│   │       ├── api-docs/
│   │       ├── app.ts
│   │       └── main.ts
│   ├── ax2-api/                          # AX-2 트랙 컨트롤러 (현재 비어 있음, D-019 후 활성화 예정)
│   └── batch/
│       └── src/cron/                     # enrichment / trend-promotion scheduler
├── packages/
│   ├── data/
│   │   ├── domain/src/                   # 트랙별 도메인 (D-020)
│   │   │   ├── ax/                       # AX-1 (현재 17개 도메인 — beauty-category, trend, ...)
│   │   │   ├── ax2/                      # AX-2 (대기)
│   │   │   ├── ax3/                      # AX-3 (대기)
│   │   │   └── shared/sku/               # 트랙 공유 (D-021)
│   │   ├── decorators/, config/, dto/, lib/, migrations/
│   └── infra/, system/                   # 인프라/시스템 공유 패키지
├── k8s/clusters/dev/                     # K8s 매니페스트
└── pnpm-workspace.yaml
```

### 2-2. 목표 (D-019 후속 정리 후)

```
changshin-api/
├── apps/
│   ├── ax-api/
│   │   └── src/
│   │       ├── ax-api.module.ts          # 모든 도메인 모듈 import
│   │       ├── ax-api.controller.ts      # 헬스체크
│   │       ├── modules/                  # 도메인 모듈 단위 분리
│   │       │   ├── auth/
│   │       │   ├── ax1/
│   │       │   ├── ax2/
│   │       │   │   ├── due-date-prediction/    # M1
│   │       │   │   ├── production-planning/    # M2
│   │       │   │   ├── logistics/               # M3
│   │       │   │   └── settlement/              # M4
│   │       │   └── ax3/
│   │       ├── common/                   # 공통 (예외, 상수, 헬퍼)
│   │       ├── guards/                   # 공통 가드
│   │       ├── strategies/
│   │       └── api-docs/
│   └── batch/
│       └── src/
│           ├── batch.module.ts
│           ├── cron/                     # 도메인별 scheduler
│           │   ├── ax2/                  # AX-2 배치 (납기 예측 batch 등)
│           │   └── ...
│           └── main.ts
├── packages/
│   ├── data/                             # 공유 엔티티·decorator·config
│   └── (필요 시) auth/                    # JWT 검증 공통 (단일 서비스라 우선순위 낮음)
├── k8s/clusters/{dev,stage,prod}/
└── pnpm-workspace.yaml
```

---

## 3. 도메인 모듈 작성 규칙

### 3-1. 디렉토리 컨벤션 (제안)

각 도메인 모듈은 자기만의:

- `<domain>.module.ts` (NestJS Module)
- `<domain>.controller.ts` (HTTP)
- `<domain>.service.ts`
- `entities/` (TypeORM 엔티티, 자기 스키마)
- `migrations/` (자기 스키마 전용 마이그레이션)
- `dto/req/`, `dto/res/`

### 3-2. 모듈 import 정책

- **권장**: 도메인 모듈 간 직접 import 금지 (공유 데이터는 `packages/` 또는 `common.*` 스키마로)
- **허용**: Auth 모듈은 모든 도메인에서 가드/데코레이터로 사용 가능
- 강제 수단: ESLint `no-restricted-imports` 규칙

### 3-3. DB 스키마 매핑

```typescript
// apps/ax-api/src/modules/ax2/entities/order.entity.ts
@Entity({ schema: 'ax2', name: 'orders' })
export class Order { ... }
```

---

## 4. 환경 변수 규칙 (단일 서비스 기준)

```
# 공통
NODE_ENV=               # development | staging | production
PORT=3000

# DB (단일 연결 풀 — 모든 스키마 접근)
DB_HOST=
DB_PORT=
DB_NAME=
DB_USERNAME=
DB_PASSWORD=

# Redis
REDIS_URL=

# JWT
JWT_PRIVATE_KEY_PEM=
JWT_PUBLIC_KEY_PEM=
JWT_ISSUER=https://changshin-api.dev.weplanet.co.kr
JWT_ACCESS_TTL=3600
JWT_REFRESH_TTL=1209600

# 외부 (옵션)
ERP_BASE_URL=
ERP_API_KEY=
```

> 시크릿은 **External Secrets로 Secrets Manager에서 자동 주입** + 일부는 K8s Secret 수동 (예: JWT 키 — [[Infrastructure/31 - Decision Log#D-018|D-018]])

---

## 5. 빌드 · 배포

### 5-1. 로컬

```bash
pnpm install
pnpm dev:ax-api          # API
pnpm dev:batch           # 배치 (별도 터미널)
```

### 5-2. Dockerfile

`apps/ax-api/Dockerfile`, `apps/batch/Dockerfile`을 각각 운영. 공통 멀티스테이지 빌드 권장.

### 5-3. ECR 레포

현재 4세트(`changshin-{auth,ax1,ax2,ax3}` + `*-deploy`) 잔재 → D-019 후속 통폐합:

- 통합안 A: `changshin-ax-api`(이미지) + `changshin-ax-api-deploy`(Flux artifact) — 1세트
- 통합안 B: 워크로드별 분리 — `changshin-ax-api`, `changshin-batch` (+ 각각 `-deploy`) — 2세트

> [!info] 결정 후 작업
> 통합안 결정 → `changshin-iac/aws/env/base.yml`의 `ecr.repositories` 갱신 → `.gitlab-ci.yml` 인자 갱신 → 기존 ECR 정리

### 5-4. K8s 배포

자세한 내용: [[Infrastructure/20 - AWS Deployment#4. K8s 리소스 구성]]

---

## 6. 포트 · 도메인

| 워크로드 | 로컬 포트 | 외부 노출 |
|---------|----------|---------|
| `ax-api` | 3000 | `https://changshin-api.dev.weplanet.co.kr/*` (path 기반) |
| `batch` | (HTTP 노출 없음) | 클러스터 내부 cron |

자세한 Ingress 설계: [[Infrastructure/10 - Architecture#3-1. 라우팅 (현행)]]

---

## 7. 체크리스트 · 새 도메인 모듈 추가

- [ ] `apps/ax-api/src/modules/<domain>/` 디렉토리 생성
- [ ] `<domain>.module.ts`·`<domain>.controller.ts`·`<domain>.service.ts` 작성
- [ ] 엔티티는 `entities/`에, 스키마는 `@Entity({ schema: '<domain>', ... })`로 명시
- [ ] 마이그레이션 파일 추가 (`<domain>/migrations/`)
- [ ] DTO `dto/req/`, `dto/res/` 작성
- [ ] 가드 적용: `@UseGuards(AuthenticationGuard, AuthorizationGuard)`
- [ ] Swagger 데코레이터 (`@ApiTags`, `@ApiOperation`)
- [ ] 테스트 (`*.spec.ts`)
- [ ] `ax-api.module.ts`에 import
- [ ] (필요 시) 배치도 `apps/batch/src/cron/<domain>/`에 추가

---

## 8. 도메인 ↔ 외부 상세 (현재 시점)

| 도메인 | NestJS 위치 | 외부 라우팅 | DB 스키마 | 상태 |
|--------|-----------|-----------|----------|------|
| Auth | `apps/ax-api/src/modules/auth/` | `/auth/*` | `auth.*` | legacy 자산 통합 진행 |
| AX1 | `apps/ax-api/src/modules/ax1/` | `/api/ax1/*` | `ax1.*` | 도메인 정의 대기 |
| AX2 | `apps/ax-api/src/modules/ax2/` | `/api/ax2/*` | `ax2.*` | 모듈 골격 작성 예정 |
| AX3 | `apps/ax-api/src/modules/ax3/` | `/api/ax3/*` | `ax3.*` | 도메인 정의 대기 |

---

## 열린 질문

> 모든 미완 항목은 [[00 - Action Board]] 에서 관리. 디렉터리 컨벤션 / `packages/` 분리 범위 / legacy stub 정리 / Migration 실행 주체 / API 버저닝 등은 [[00 - Action Board#📥 백로그 (다음 사이클 / 결정·답변 도착 시 진행)]] 참조.

---

> 처음으로: [[00 - Backend (Index)]]
