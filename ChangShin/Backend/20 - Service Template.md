---
title: Service Template (from Boilerplate)
parent: "[[00 - Backend (Index)]]"
project: 창신
created: 2026-04-23
status: draft
tags:
  - backend
  - template
  - nestjs
  - 창신
---

# Service Template · 보일러플레이트 기반 서비스 구성

> 상위 문서: [[00 - Backend (Index)]]
> 이전: [[10 - Auth Strategy]]

> [!summary] 한 줄로 말하면
> 보일러플레이트의 `apps/` 아래 앱 하나(예: `user-api`)를 **기본 템플릿**으로 삼아 Auth / AX1 / AX2 / AX3 네 개 앱을 분화한다. 공통 가드·설정은 `libs/`로 분리.

---

## 1. 보일러플레이트 앱 구조 (참고)

```
apps/user-api/
├── src/
│   ├── app.ts                            # NestJS bootstrap
│   ├── user-api.controller.ts            # 루트 컨트롤러 (헬스체크 등)
│   ├── user-api.module.ts                # 루트 모듈
│   ├── api-docs/                         # Swagger YAML + Markdown
│   ├── controllers/
│   │   ├── user-http.module.ts           # 컨트롤러 모듈 묶음
│   │   ├── auth/
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── auth-http.module.ts
│   │   │   └── dto/
│   │   │       ├── req/
│   │   │       └── res/
│   │   ├── faqs/
│   │   └── files/
│   └── guards/
│       ├── authentication.guard.ts
│       └── authorization.guard.ts
├── eslint.config.mjs
├── jest.config.ts
├── nest-cli.json
├── package.json
└── .lintstagedrc.json
```

---

## 2. 서비스별 역할 분담

### 2-1. Auth (`apps/auth/`)

- **기반**: `user-api`의 `controllers/auth/` **전체**를 복사 후 확장
- **추가·수정**:
  - `/auth/ax1/login`, `/auth/ax2/login`, `/auth/ax3/login` 분기
  - JWT 발급 시 `aud` 주입 (자세한 것: [[10 - Auth Strategy]])
  - `/auth/.well-known/jwks.json` 엔드포인트 추가
  - Refresh 토큰 저장소(Redis) 설정
- **제거**: `faqs`, `files` 등 AX 서비스가 필요로 하지 않는 컨트롤러

### 2-2. AX1 API (`apps/ax1-api/`)

- **기반**: `user-api`의 구조 복사
- **남김**: `authentication.guard.ts` (수정해서 `aud=ax1` 검증)
- **제거**: `controllers/auth/` (Auth 서버가 담당하므로 삭제)
- **추가**: AX1 도메인 컨트롤러 (도메인은 별도 결정)
- **설정**: 공유 DB 중 `ax1.*` 스키마 접근 권한만

### 2-3. AX2 API (`apps/ax2-api/`)

- AX1과 동일한 패턴, `aud=ax2`, `ax2.*` 스키마
- [[AX-2 지능형 스케줄러/00 - AX-2 쉬운 설명서 (Index)|AX-2 모듈 1~4]] 도메인이 여기에 구현됨

### 2-4. AX3 API (`apps/ax3-api/`)

- 동일 패턴, `aud=ax3`, `ax3.*`

---

## 3. 공유 코드 배치 (`libs/`)

```
libs/
├── auth/                       # JWT 검증 공통 (AX1/2/3 모두 사용)
│   ├── src/
│   │   ├── jwt-audience.guard.ts
│   │   ├── jwks-client.service.ts
│   │   └── auth.module.ts
│   └── tsconfig.lib.json
├── common/                     # 공통 유틸 · 상수 · 에러 클래스
├── database/                   # TypeORM datasource · 공통 엔티티 (users, orgs)
└── api-docs/                   # Swagger 공통 설정
```

### 3-1. libs/auth 사용 예 (AX1)

```typescript
// apps/ax1-api/src/ax1-api.module.ts
@Module({
  imports: [
    AuthModule.forService({ aud: 'ax1' }),
    // ...
  ],
})
export class Ax1ApiModule {}
```

### 3-2. libs/database 예

- 공유 DB의 `common.*` 스키마 엔티티 (예: `User`, `Organization`) 정의
- 각 앱은 `libs/database`를 import + 자기 스키마 엔티티를 `apps/<svc>/src/entities/` 에 추가

---

## 4. 새 앱 생성 절차

NestJS CLI로 monorepo에 새 앱 추가:

```bash
# 보일러플레이트 루트에서
nest generate app auth
nest generate app ax1-api
nest generate app ax2-api
nest generate app ax3-api
```

이후:

1. `apps/<svc>/` 의 기본 구조를 `user-api` 참고해 정리
2. `nest-cli.json`에 앱 등록 확인
3. `package.json` 스크립트 추가 (`start:dev:auth`, `start:dev:ax1` 등)
4. 불필요 컨트롤러 제거
5. `libs/auth`, `libs/database` 등록

---

## 5. 환경 변수 규칙 (초안)

각 앱 공통:

```
NODE_ENV=                 # development | staging | production
PORT=                     # 앱 포트 (내부)

DB_HOST=
DB_PORT=
DB_NAME=
DB_USERNAME=              # 앱별 분리된 DB 계정
DB_PASSWORD=
DB_SCHEMA=                # auth / ax1 / ax2 / ax3 / common
```

Auth 서버만:

```
JWT_PRIVATE_KEY_PEM=      # 서명용 비밀키
JWT_KID=                  # 키 ID (JWKS)
JWT_ACCESS_TTL=3600       # 초 단위
JWT_REFRESH_TTL=1209600
REDIS_URL=
```

AX 서버만:

```
JWT_EXPECTED_AUD=ax1      # ax1 | ax2 | ax3 (앱별)
JWT_ISSUER=https://auth.changshin.io
JWKS_URL=https://auth.changshin.io/auth/.well-known/jwks.json
```

> 시크릿은 Secrets Manager + External Secrets로 주입 ([[Infrastructure/20 - AWS Deployment#2. 사용할 AWS 서비스 매핑]])

---

## 6. 포트 · 도메인 규칙 (초안)

| 서비스 | 로컬 포트 | 도메인 (후보) |
|--------|----------|-------------|
| Auth | 3000 | `auth.changshin.io` |
| AX1 | 3001 | `ax1.changshin.io` |
| AX2 | 3002 | `ax2.changshin.io` |
| AX3 | 3003 | `ax3.changshin.io` |

또는 path 기반:
- `https://api.changshin.io/auth/*`
- `https://api.changshin.io/ax1/*`

자세한 Ingress 설계: [[Infrastructure/10 - Architecture#3-1. 라우팅]]

---

## 7. Dockerfile · 빌드 전략

### 공통 Dockerfile (monorepo 기준)

```dockerfile
# 빌드용 인자
ARG APP=auth

FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build -- --project $APP

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist/apps/$APP ./dist
COPY --from=builder /app/node_modules ./node_modules
ENV APP=${APP}
CMD ["node", "dist/main.js"]
```

ECR에는 앱별 리포지토리로 push:

- `<account>.dkr.ecr.<region>.amazonaws.com/changshin/auth`
- `.../changshin/ax1-api`
- `.../changshin/ax2-api`
- `.../changshin/ax3-api`

---

## 8. 체크리스트 · 앱 하나 신규 생성 시

- [ ] `nest generate app <svc>` 로 앱 골격 생성
- [ ] `user-api` 구조 참고해 `controllers/`, `guards/`, `api-docs/` 디렉토리 정비
- [ ] AX API 앱이면 `controllers/auth/` 제거
- [ ] `libs/auth`, `libs/database` 등록
- [ ] `apps/<svc>/.env.example` 작성
- [ ] Dockerfile 빌드 인자 추가
- [ ] ECR 레포지토리 생성 (Terraform `modules/ecr`)
- [ ] K8s 매니페스트 작성 (Deployment · Service · Ingress · ExternalSecret)
- [ ] Swagger doc (`api-docs/`) 시드
- [ ] CI 파이프라인에 빌드·테스트 추가

---

## 열린 질문

- [ ] 공유 라이브러리 배포 방식 (NestJS monorepo `libs/` vs 별도 npm private registry)
- [ ] TypeORM Migration 관리: 서비스별 실행 vs Auth에 집중
- [ ] 헬스체크 엔드포인트 규격 (`/healthz` · `/readyz`)
- [ ] API 버저닝 (`/v1/...`) 시점

---

> 처음으로: [[00 - Backend (Index)]]
