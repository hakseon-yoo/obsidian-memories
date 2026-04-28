---
title: Infrastructure (Index)
project: 창신
created: 2026-04-23
updated: 2026-04-28
tags:
  - MOC
  - infrastructure
  - 창신
---

# Infrastructure

> [!abstract] 허브 문서
> 창신 AX 시리즈(AX1 · AX2 · AX3) 인프라 문서를 모으는 MOC.
> **K8s 기반 단일 통합 API**로 구성하고 **AWS(EKS)로 시작해 Azure(AKS)로 이전** 가능.

> [!warning] 2026-04-28 아키텍처 변경
> 4서비스 분리 → **단일 통합 API 서비스**로 변경 ([[31 - Decision Log#D-019|D-019]]). 모든 인프라 문서는 새 구조 기준으로 갱신됨.

---

## 한눈에 보기

- **서비스**: **단일 통합 API (`ax-api`) + Batch (`batch`)** — 도메인은 모듈로 분리 (Auth/AX1/AX2/AX3)
- **저장소**: GitLab `changshin/changshin-api` 1개
- **데이터베이스**: PostgreSQL · 공유 DB · 스키마 단위 논리 분리
- **오케스트레이션**: Kubernetes (EKS · namespace `changshin`)
- **1단계**: AWS EKS · RDS · ElastiCache · ALB · Flux GitOps (dev 환경 운영 중)
- **2단계**: Azure AKS로 이전 (예정)
- **IaC**: Terraform (`changshin-iac` 리포)

---

## 문서 구성

### 개요 · 아키텍처

- [[01 - Overview]] — 프로젝트 맥락 · 단일 서비스 구성 · 배포 플랫폼 · 문서 로드맵
- [[10 - Architecture]] — 시스템 아키텍처 · 모듈 경계 · 트래픽 흐름 · 공유 DB 전략

### 배포

- [[20 - AWS Deployment]] — EKS 기반 AWS 배포 설계 · `changshin-iac` 모듈 매핑
- [[30 - Azure Migration]] — Azure(AKS) 이전 전략 · 서비스 매핑 · 이전 단계

### 결정 기록

- [[31 - Decision Log]] — 주요 기술 선택과 근거 (D-001 ~ D-019)

### 운영 현황

- [[32 - Deployment State]] — 구축 완료/진행 상태 · 핵심 참조값 · 남은 작업

<!--
향후 추가 가능:
- 02 - Requirements & Constraints
- 11 - Modules (Auth/AX1/AX2/AX3 모듈 세부)
- 12 - Database (스키마 상세)
- 21 - CI-CD
- 22 - Security & Access
- 23 - Observability
-->

---

## 관련 프로젝트 · 문서

- [[AX-2 지능형 스케줄러/00 - AX-2 쉬운 설명서 (Index)]] — AX-2 도메인 (`ax-api`의 AX2 모듈에 구현)
- [[Backend/00 - Backend (Index)]] — 서버 코드 허브 (NestJS monorepo 기반)

---

## 현재 상태

- **2026-04-28**: 4서비스 분리 → 단일 통합 서비스로 [[31 - Decision Log#D-019|아키텍처 변경]] 반영
- **2026-04-27 기준 구축 상태**: AWS dev 환경 + EKS + Flux + observability 스택 완료. 첫 Pod이 `https://changshin-api.dev.weplanet.co.kr` 에서 응답 중. 자세한 상태는 [[32 - Deployment State]]
- **주요 결정**: D-001 ~ D-019 확정 ([[31 - Decision Log]])
- **다음 단계**: `changshin-api`의 도메인 모듈 구현 (Auth · AX-2 우선)
