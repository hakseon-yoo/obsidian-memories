---
title: Backend (Index)
project: 창신
created: 2026-04-23
updated: 2026-04-23
tags:
  - MOC
  - backend
  - nestjs
  - 창신
---

# Backend

> [!abstract] 허브 문서
> 창신 AX 시리즈의 서버 코드(Auth + AX1/2/3 API) 설계 문서 MOC.
> 사내 `weplanet-starter-nestjs` 보일러플레이트를 기반으로 네 개 서비스를 구성한다.

---

## 한눈에 보기

- **프레임워크**: NestJS (monorepo · `apps/` 구조)
- **서비스**: 4개 — Auth / AX1 / AX2 / AX3
- **인증 전략**: **JWT `aud` (audience) 클레임으로 서비스 분기**
  - Auth 서버가 `aud: "ax1" | "ax2" | "ax3"`를 발급
  - 각 AX API 서버는 자신의 `aud`만 수락
- **DB**: 공유 DB · 스키마 단위 분리 (자세한 내용: [[Infrastructure/10 - Architecture#2. 공유 데이터베이스 전략]])
- **인프라**: K8s 위에 각 서비스 독립 배포 ([[Infrastructure/20 - AWS Deployment]])

---

## 문서 구성

### 개요

- [[01 - Overview]] — 보일러플레이트 구조 · 4개 서비스 파생 계획

### 인증 · 인가

- [[10 - Auth Strategy]] — Auth 서버 역할 · `aud` 기반 JWT · 가드 동작

### 서비스 구성

- [[20 - Service Template]] — 보일러플레이트 → 각 서비스로 분화하는 규칙

<!--
향후 추가 가능:
- 11 - Authorization (Roles & Permissions)
- 21 - Shared Libraries (공통 코드)
- 22 - Domain Modeling
- 30 - Testing Strategy
- 31 - Error Handling & Logging
-->

---

## 관련 문서

- 인프라: [[Infrastructure/00 - Infrastructure (Index)]]
- AX-2 도메인: [[AX-2 지능형 스케줄러/00 - AX-2 쉬운 설명서 (Index)]]
- 인프라 결정 로그: [[Infrastructure/31 - Decision Log]]

---

## 현재 상태

- 초기 설계 단계 (2026-04-23)
- 보일러플레이트 확보 · Auth 엔드포인트 기본 동작 확인 필요
- 다음 단계: `aud` 발급·검증 로직 구현 설계 확정
