---
title: Backend (Index)
project: 창신
created: 2026-04-23
updated: 2026-04-28
tags:
  - MOC
  - backend
  - nestjs
  - 창신
---

# Backend

> [!abstract] 허브 문서
> 창신 AX 시리즈의 서버 코드 설계 문서 MOC.
> 사내 `weplanet-starter-nestjs` 보일러플레이트 기반 **단일 통합 API (`changshin-api`)** 구조.

> [!warning] 2026-04-28 변경
> 4서비스(Auth + AX1/2/3) 분리 → **단일 API 서비스**로 변경됨 ([[Infrastructure/31 - Decision Log#D-019|D-019]]). 이 폴더의 모든 문서는 새 구조(모듈러 모놀리스) 기준으로 갱신됨.

---

## 한눈에 보기

- **프레임워크**: NestJS (monorepo · `apps/` 구조)
- **저장소**: GitLab `changshin/changshin-api` 1개
- **앱 구성**:
  - `apps/ax-api/` — HTTP API (모든 도메인 모듈 호스팅)
  - `apps/batch/` — cron · 배치 (스케줄러)
- **도메인 분리 방식**: 같은 서비스 내 **NestJS 모듈 디렉터리** 단위
  - `Auth` 모듈 — 사용자 · 인증 · 토큰
  - `AX1` 모듈 — AX1 도메인
  - `AX2` 모듈 — [[AX-2 지능형 스케줄러/00 - AX-2 쉬운 설명서 (Index)|AX-2 지능형 스케줄러]] (M1~M4)
  - `AX3` 모듈 — AX3 도메인
- **DB**: PostgreSQL · 스키마 단위 논리 분리 (자세히: [[Infrastructure/10 - Architecture#2. 공유 데이터베이스 전략]])
- **인증**: 자체 JWT (D-009) — 같은 프로세스 내 가드가 검증

---

## 문서 구성

### 개요

- [[01 - Overview]] — 보일러플레이트 구조 · 단일 통합 서비스 설계 · 도메인 모듈 배치

### 인증 · 인가

- [[10 - Auth Strategy]] — JWT 발급·검증 (단일 서비스 안에서) · `aud` 클레임 재정의 옵션

### 서비스 구성

- [[20 - Service Template]] — 보일러플레이트 → `changshin-api` 정리 규칙 · `apps/ax-api` 모듈 구조

<!--
향후 추가 가능:
- 11 - Authorization (Roles & Permissions)
- 21 - Shared Libraries (packages/)
- 22 - Domain Modeling
- 30 - Testing Strategy
- 31 - Error Handling & Logging
-->

---

## 관련 문서

- 인프라: [[Infrastructure/00 - Infrastructure (Index)]]
- 결정 로그: [[Infrastructure/31 - Decision Log]] (D-001 ~ D-019)
- 배포 현황: [[Infrastructure/32 - Deployment State]]
- AX-2 도메인: [[AX-2 지능형 스케줄러/00 - AX-2 쉬운 설명서 (Index)]]

---

## 현재 상태

- **2026-04-28**: 4서비스 분리 → 단일 통합으로 [[Infrastructure/31 - Decision Log#D-019|D-019]] 변경 반영
- 클러스터에 `apps/ax-api` Deployment 동작 중
- 다음 단계: legacy `changshin-auth-api` 자산을 `apps/ax-api/src/modules/auth/`로 흡수 → AX-2 도메인 모듈 골격 작성
