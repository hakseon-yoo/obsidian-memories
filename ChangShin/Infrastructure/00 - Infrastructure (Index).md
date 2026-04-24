---
title: Infrastructure (Index)
project: 창신
created: 2026-04-23
updated: 2026-04-23
tags:
  - MOC
  - infrastructure
  - 창신
---

# Infrastructure

> [!abstract] 허브 문서
> 창신 AX 시리즈(AX1 · AX2 · AX3) 인프라 문서를 모으는 MOC.
> **K8s 기반 마이크로서비스**로 구성하고 **AWS(EKS)로 시작해 Azure(AKS)로 이전**할 예정이다.

---

## 한눈에 보기

- **서비스**: 4개 마이크로서비스 (Auth + AX1/2/3 API)
- **데이터베이스**: 공유 DB (논리 분리)
- **오케스트레이션**: Kubernetes
- **1단계**: AWS EKS · RDS · ElastiCache · ALB · Flux GitOps
- **2단계**: Azure AKS로 이전
- **IaC**: Terraform (사내 `weplanet-starter-iac` 보일러플레이트 활용)

---

## 문서 구성

### 개요 · 아키텍처

- [[01 - Overview]] — 프로젝트 맥락 · 서버 구성 · 배포 플랫폼 · 문서 로드맵
- [[10 - Architecture]] — 시스템 아키텍처 · 서비스 경계 · 트래픽 흐름 · 공유 DB 전략

### 배포

- [[20 - AWS Deployment]] — EKS 기반 AWS 배포 설계 · 보일러플레이트 모듈 매핑
- [[30 - Azure Migration]] — Azure(AKS) 이전 전략 · 서비스 매핑 · 이전 단계

### 결정 기록

- [[31 - Decision Log]] — 주요 기술 선택과 근거 (D-001 ~)

### 운영 현황

- [[32 - Deployment State]] — 구축 완료/진행 상태 · 핵심 참조값 · 남은 작업

<!--
향후 추가 가능:
- 02 - Requirements & Constraints
- 11 - Services (Auth/AX1/AX2/AX3 세부)
- 12 - Database (스키마 상세)
- 21 - CI-CD
- 22 - Security & Access
- 23 - Observability
-->

---

## 관련 프로젝트 · 문서

- [[AX-2 지능형 스케줄러/00 - AX-2 쉬운 설명서 (Index)]] — AX-2 서비스 (AX2 API가 담당할 도메인)

---

## 현재 상태

- 초기 설계 문서 작성 단계 (2026-04-23)
- 주요 결정: D-001 ~ D-014 확정 (자세한 내용은 [[31 - Decision Log]])
- **2026-04-24 기준 구축 상태**: AWS dev 환경 + EKS + Flux + observability 스택 완료. 앱 리포 3개(auth/ax2/ax3) 보일러플레이트 push 완료. 자세한 상태는 [[32 - Deployment State]] 참고.
- **다음 단계**: `changshin-auth-api`에서 Auth 서비스 구현 (D-007/D-009)
