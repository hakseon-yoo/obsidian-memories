---
title: Action Board (놓치고 가는 작업 단일 보드)
project: 창신
created: 2026-04-29
updated: 2026-04-30
tags:
  - MOC
  - action-board
  - 창신
---

# Action Board · 작업 단일 보드

> [!abstract] 이 문서의 쓰임
> 흩어져 있던 "열린 질문 / 미결정 / 남은 작업" 을 **한 보드**에 모은 단일 소스.
> 각 항목은 출처 문서로 위키링크. 각 문서의 "열린 질문" 섹션은 이 보드를 가리키는 링크만 유지한다.

> [!tip] 사용 규칙
> - 새 작업이 생기면 → 적절한 카테고리에 항목 추가 + 출처 문서에서 이 보드로 링크
> - 항목이 결정되면 → `✅ 최근 완료` 로 이동하거나 [[Infrastructure/31 - Decision Log]] 의 `D-###` 로 승격
> - **상태 표기**: `🔥` 진행중 · `❌` 미정 · `⏳` 부분 진행/임시 · `❓` 클라이언트 답변 대기 · `⛔` 본인 책임 외 · `✅` 완료
> - 본문 표는 **요약 + 링크 위주**. 자세한 맥락은 출처 문서에서.

---

## 🔥 현재 사이클 (본인 P0 — 직접 진행)

### A. `changshin-api` 통합 마무리 (D-019 후속)

| 항목 | 상태 | 출처 |
|------|------|------|
| ~~legacy admin-api → ax-api 흡수~~ → **완료 (2026-04-29)**: `Administrator`/`AdministratorPassword` 엔티티 + `AdministratorService`/`AdministratorModule` (`shared/administrator/`), `AdminJwtStrategy`, `AuthController`/`AuthService` (`controllers/auth/`), `AdministratorsController` (`controllers/administrators/`), `AdminHttpModule` 신규 묶음, `ax-api.module.ts` imports 추가. 타입체크 통과 | ✅ | 일반 사용자 흐름(`users`/`verifications` 등)은 #34 답변 후 별도 결정 |
| ~~`aud` 클레임 처리 방향 결정~~ → **위플래닛 표준 `user / admin` 채택** ([[Infrastructure/31 - Decision Log#D-024]]) | ✅ | 2026-04-29 결정 |
| ~~Ingress path 단순화 여부~~ → **D-011 path 분리 유지로 결정** ([[Infrastructure/31 - Decision Log#D-023]]) | ✅ | 2026-04-29 회의 후속으로 닫힘 |
| ~~ECR 리포 통폐합~~ → **점검 결과 이미 1세트(api+api-deploy)로 시작. batch 는 [[Infrastructure/31 - Decision Log#D-025\|D-025]] 로 별도 1세트 추가** | ✅ | 2026-04-29. 사용자 `terraform apply` 필요 (changshin-iac base + dev) |
| ~~DB 스키마 마이그레이션~~ → **D-002 schema 분리 폐기, 모두 `public` 사용** ([[Infrastructure/31 - Decision Log#D-026\|D-026]]) | ✅ | 2026-04-30 결정. 작업 불필요 |

### B. AX-2 도메인 모듈 구현 시작

| 항목 | 상태 | 출처 |
|------|------|------|
| Module 1 · 납기일 예측 모듈 골격 | 🔥 | [[AX-2 지능형 스케줄러/02 - Module 1 · 납기일 예측]] |
| Module 2 · 생산 계획 관리 모듈 골격 | 🔥 | [[AX-2 지능형 스케줄러/03 - Module 2 · 생산 계획 관리]] |
| Module 3 · 물류 · 배차 (Phase 3) | 📅 | [[AX-2 지능형 스케줄러/04 - Module 3 · 물류 · 배차]] |
| Module 4 · 구매 · 정산 (Phase 3) | 📅 | [[AX-2 지능형 스케줄러/05 - Module 4 · 구매 · 정산]] |

### C. IaC · CI 자동화

| 항목 | 상태 | 출처 |
|------|------|------|
| `changshin-iac` 의 `.gitlab-ci.yml` 작성 (terraform plan/apply 자동화) | ❌ | [[Infrastructure/32 - Deployment State#단기 (현재 사이클)]] |

---

## 📅 회의 발생 액션 (회의 후속)

> 출처: [[Meetings/2026-04-29 - 화장품 용기 데이터 프로젝트 범위 조정]] · [[Meetings/2026-04-30 - 클라이언트 답변 - 수주 시나리오 · 납기 로직]]

| 항목 | 상태 | 비고 |
|------|------|------|
| ~~AX1·AX2·AX3 URL 구조 결정~~ → **FE 화면만 path 분리, BE 는 D-011 그대로** ([[Infrastructure/31 - Decision Log#D-023]]) | ✅ | 2026-04-29 결정 |
| **로그인·마스터 계정·권한 IA** — D-024 admin auth BE API 는 다 흡수됨([[Infrastructure/31 - Decision Log#D-024\|D-024]]). 첫 master 발급(bootstrap migration), 임시 비밀번호 변경 강제 흐름, 권한 그룹 확장 등 운영 IA 는 **필요 시점에 도입**. 사용자 그룹 #34 답변 / 실제 admin 화면 개발 트리거 발생 시 진행 | 📅 | 2026-04-30 보류. 검토 결과는 [[Meetings/2026-04-29 - 화장품 용기 데이터 프로젝트 범위 조정]] 와 D-024 영향 항목 참조 |
| **5월 말 vs 6월 말 타임라인 정합성 점검** — 본인 P0(6월말) ↔ 회의(5월말 1차) | ❌ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#0. P0 빠른 참조 (착수 전 결정 필수)\|#64]] |
| **NDA 체결 후 3D 도면 수령** — SKU 미매칭 41건 보강 활용 가능 | ❓ | 팀 스파르타 ↔ 창신 NDA 진행 후 |
| **SKU 매핑 정합성** — 본인 진행한 [[Infrastructure/31 - Decision Log#D-021\|D-021]]/[[Infrastructure/31 - Decision Log#D-022\|D-022]] 결과물이 회의 SKU 매핑 흐름과 연결되는지 확인 | ❌ | 정보 동기화 |
| **수주 시나리오 입력 화면 필드 설계** — 기제품/신제품 2-way 분기 반영, Module 1 entity 설계 직전에 마무리 | 🔥 | [[Meetings/2026-04-30 - 클라이언트 답변 - 수주 시나리오 · 납기 로직]] |
| **수주 확정 후 프로세스 온라인 미팅 일정 조율** — 서주현 팀장 (모니터 딜로이트) | 🔥 | 클라이언트가 카톡 답변 대신 미팅으로 설명 약속 |
| **기제품 부자재·공정별 평균 소요 변수 인터뷰 결과 수령** — 서주현 팀장이 생산관리팀장 인터뷰 후 공유 예정 | ❓ | Module 1 알고리즘 본체 구현 트리거 |

> 본 회의의 외부 액션(스파르타/고객사) 은 Action Board 에 등록하지 않고 [[Meetings/2026-04-29 - 화장품 용기 데이터 프로젝트 범위 조정#3. Action Items\|회의록 자체]] 에서 추적.

---

## ❓ 클라이언트 답변 대기 (P0 — 답 없으면 코드 작성 차단)

> 마스터: [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트]]

### B-1. ERP 연동 (3건)

| 항목 | 상태 | 출처 |
|------|------|------|
| 현재 ERP 정체 (제품·버전·벤더) | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#2. 데이터 · 외부 시스템\|#26]] |
| ERP 연동 방식 가능성 (REST/DB/배치/큐) | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#2. 데이터 · 외부 시스템\|#27]] |
| ERP 데이터 실시간성 (폴링 주기·푸시 가능?) | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#2. 데이터 · 외부 시스템\|#28]] |

### C-1. 사용자 · 인증 (3건)

| 항목 | 상태 | 출처 |
|------|------|------|
| 사용자 그룹 정체 (창신/외주처/ODM/고객사) | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#3. 사용자 · 인증 · 권한\|#34]] |
| 외주처 직원 계정 발급 방식 | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#3. 사용자 · 인증 · 권한\|#35]] |
| 창신 사내 SSO 존재 (OIDC/SAML) | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#3. 사용자 · 인증 · 권한\|#36]] |

### A-1. AX-2 도메인 · 비즈니스 (P0 9건)

| 항목 | 상태 | 출처 |
|------|------|------|
| 납기일 예측의 타겟 납기일 세팅 로직 | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#1-1. Module 1 — 납기일 예측\|#1]] |
| 공정별 "며칠 전 컨펌 요청" D-day 기준 | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#1-2. Module 2 — 생산 계획 관리\|#6]] |
| WIP 품질 판정 워크플로우 (이중 컨펌 / 사진 필수?) | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#1-2. Module 2 — 생산 계획 관리\|#7]] |
| 외주처 PDA 활용 가능 여부 + 대안 | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#1-3. Module 3 — 물류 · 배차\|#11]] |
| ODM 창고 미예약 사고 통계 | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#1-3. Module 3 — 물류 · 배차\|#12]] |
| 재생산 공제 로직 상세 | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#1-4. Module 4 — 구매 · 정산\|#17]] |
| 품질 귀책 판단 주체·표준 절차 | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#1-4. Module 4 — 구매 · 정산\|#18]] |
| 입고 시 불량/수량 부족 ERP 미입력 흐름 | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#1-5. 공통 · 전반\|#22]] |
| 금형 샷 수 관리 위치·계측 방법 | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#1-5. 공통 · 전반\|#23]] |
| **(SKU 마스터 클렌징 책임 부서·완료 시점)** — 1차 완료, 후속 41건 정리 책임만 미정 | ⏳ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#1-1. Module 1 — 납기일 예측\|#2]] |

### D-1. 운영 · 규모 · SLA · 보안 (P0 4건)

| 항목 | 상태 | 출처 |
|------|------|------|
| 예상 동시 사용자 수 (피크/평균) | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#4. 운영 · 규모 · SLA\|#41]] |
| 운영 시간대 (24/7 vs 평일 09-18, 야간 배치) | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#4. 운영 · 규모 · SLA\|#42]] |
| 데이터 거주 요구 (한국/일본 리전?) | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#5. 보안 · 컴플라이언스\|#48]] |
| 개인정보 취급 항목 식별 | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#5. 보안 · 컴플라이언스\|#49]] |

### D-2. 인프라 · 도메인 (임시 결정, 영구화 필요)

| 항목 | 상태 | 출처 |
|------|------|------|
| AWS 계정 창신 보유 여부 | ⏳ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#6. 인프라 · 배포 환경\|#54]] · [[Infrastructure/31 - Decision Log#D-012]] |
| 도메인 보유 여부 (`changshin.io` 등) | ⏳ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#6. 인프라 · 배포 환경\|#55]] · [[Infrastructure/31 - Decision Log#D-015]] |
| 클라이언트 도메인 확정 시 인증서 + host 일괄 교체 | 📅 | [[Infrastructure/31 - Decision Log#미결정 · 논의 필요]] |
| 현장 파일럿 대상 (어느 외주처·SKU?) | ❓ | [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#8. 일정 · 우선순위 · 범위\|#66]] |

### ⛔ 본인 책임 외 (AX1·AX3 담당자 위임)

- AX1 도메인 영역, AX3 도메인 영역, AX1·AX3 착수 시기 — [[AX-2 지능형 스케줄러/10 - 프로젝트 착수 질의 리스트#7. AX 시리즈 도메인 명확화|7번 섹션]]

---

## 📥 백로그 (다음 사이클 / 결정·답변 도착 시 진행)

| 항목 | 상태 | 출처 |
|------|------|------|
| **AX1·AX3 도메인 모듈 구현** (도메인 정의 후) | 📅 | [[Infrastructure/32 - Deployment State#중기]] |
| **80 포트 listener 미동작** 진단 (HTTPS-only? 80→443 redirect?) | 📅 | [[Infrastructure/31 - Decision Log#미결정 · 논의 필요]] |
| **OIDC Provider 소유권 정리** — 공용 리소스 거버넌스 | 📅 | [[Infrastructure/32 - Deployment State#5. OIDC Provider 공용 리소스 주의]] |
| **Backend state 공유 방법** — `aws/env/backend/terraform.tfstate` 로컬 only | 📅 | [[Infrastructure/32 - Deployment State#중기]] |
| **Multi-AZ · HA 수준** (개발/스테이징/운영 차등) | 📅 | [[Infrastructure/01 - Overview#열린 질문]] |
| **환경 분리 정책** (dev/stage/prod 모두 필요?) | 📅 | [[Infrastructure/01 - Overview#열린 질문]] |
| **Saga 필요 유스케이스** 존재 여부 | 📅 | [[Infrastructure/10 - Architecture#열린 질문]] |
| **API 버저닝**(`/v1/...`) 도입 시점 | 📅 | [[Backend/20 - Service Template#열린 질문]] |
| **MFA 도입 시점** | 📅 | [[Backend/10 - Auth Strategy#열린 질문]] |
| **Refresh 토큰 저장소** (DB vs Redis), 수명 결정 | 📅 | [[Backend/10 - Auth Strategy#열린 질문]] |

---

## 🧹 데이터 클렌징 후속 (D-021/D-022)

| 항목 | 상태 | 출처 |
|------|------|------|
| 미매칭 자재코드 41건 정규화 (변형 코드 + CSV 중복) | 📅 | [[Infrastructure/31 - Decision Log#D-022]] · `data/unmatched-report.csv` |
| `CSF60-DU` · `CSF100-DU` 챔버 구조 확인 (단일 vs 이중) | 📅 | [[Infrastructure/31 - Decision Log#미결정 · 논의 필요]] |
| 시리즈 표기 흔들림 정리 (`장식A` 별도 시리즈 vs `A` 변형?) | 📅 | [[Infrastructure/31 - Decision Log#미결정 · 논의 필요]] |

---

## 📅 장기 / Phase 4

| 항목 | 상태 | 출처 |
|------|------|------|
| Azure 이전 (Phase 2) — 트리거 조건 결정 | 📅 | [[Infrastructure/30 - Azure Migration]] |
| ML 기반 납기 예측 도입 | 📅 | [[AX-2 지능형 스케줄러/07 - 제안 로드맵]] |
| 금형 메인터넌스 사이클 자동화 | 📅 | [[AX-2 지능형 스케줄러/06 - 공통 · 보조 영역]] |

---

## ✅ 최근 완료 (참고용 · 시간순)

| 일자 | 항목 |
|------|------|
| 2026-04-30 | **클라이언트 답변 수령 (수주 시나리오 · 납기 로직)** — 수주 화면은 **기제품/신제품 2-way 분기**, 입력은 자재코드+수량. 신제품 금형 평균 45일(capa 1순위 변수). 기제품 변수는 생산관리팀장 인터뷰 후 추가 공유. 외주처 생산계획 미수신 전제 → 시스템 내 기수주 점유율 기반 지연 추정 ([[Meetings/2026-04-30 - 클라이언트 답변 - 수주 시나리오 · 납기 로직]]) |
| 2026-04-30 | **D-027 K8s namespace = 환경별** — D-017 superseded. dev/stage/prod 환경 prefix 채택. 실제 운영 패턴(`dev` namespace)을 명시 결정으로 흡수 ([[Infrastructure/31 - Decision Log#D-027]]) |
| 2026-04-30 | **D-026 PostgreSQL schema 분리 폐기** — D-002 partially superseded. 모든 테이블 `public` 단일 schema 사용. 단일 서비스 + 단일 DB 계정 운영에서 schema 분리 실익 약함. entity 갱신·마이그레이션 작업 불필요 ([[Infrastructure/31 - Decision Log#D-026]]) |
| 2026-04-29 | **D-025 batch 워크로드 활성화 (taabshop 패턴)** — `changshin-batch` ECR 1개(image 보관용) + 단일 Flux Kustomization(api) 이 root kustomization 으로 ax-api+batch 모두 apply. batch 의 `job-flux` 잡 폐기. 1차 시도(batch 별도 Flux Kustomization) 점검 결과 ax-api 만 적용되는 문제 → taabshop 패턴 정합으로 정정 ([[Infrastructure/31 - Decision Log#D-025]]) |
| 2026-04-29 | **legacy admin-api → ax-api 흡수 완료** — Administrator 도메인(`shared/administrator/`) + AdminJwtStrategy + admin auth/administrators 컨트롤러 + AdminHttpModule. 타입체크 통과. D-024 적용 |
| 2026-04-29 | **D-024 JWT `aud` = 위플래닛 표준 `user / admin`** — D-019 후속 "`aud` 처리 방향" 닫음. D-007 의 서비스 분기 의미를 클라이언트 종류 식별로 재정의 ([[Infrastructure/31 - Decision Log#D-024]]) |
| 2026-04-29 | **D-023 AX1·AX2·AX3 URL 분리** — FE 화면만 path 분리, BE 는 D-011 그대로 유지. D-019 후속 "Ingress path 단순화" 닫음 ([[Infrastructure/31 - Decision Log#D-023]]) |
| 2026-04-29 | **SKU 이미지 S3 + DB 1차 반영** — 287/289 매칭 ([[Infrastructure/31 - Decision Log#D-022]]) |
| 2026-04-29 | **SKU 마스터 도메인 설계** + DB 시드 330건 ([[Infrastructure/31 - Decision Log#D-021]]) |
| 2026-04-28 | **D-020 도메인 패키지 트랙별 분리** (`ax/ax2/ax3/shared/` sub-barrel) |
| 2026-04-28 | **D-019 단일 통합 API 서비스로 변경** — 4서비스 → 1서비스 ([[Infrastructure/31 - Decision Log#D-019]]) |
| 2026-04-29 | **AX-2 스코프 확정** — MVP 6월말, 워크프로세스 흐름 순서, 웹 only + 반응형 |
| 2026-04-27 | **Auth Pod 첫 외부 노출** — `https://changshin-api.dev.weplanet.co.kr/auth/health` 200 |
| 2026-04-24 | **AWS dev 환경 + EKS + Flux + observability** 스택 완료 |
| 2026-04-23 | **D-001 ~ D-018** 초기 결정 일괄 ([[Infrastructure/31 - Decision Log]]) |

---

## 🔁 운영 메모

- **이 보드의 단일 소스 원칙**: 흩어진 미완 항목은 모두 이 보드로 모음. 출처 문서엔 짧은 요약 + 이 보드 링크만.
- **갱신 주기**: 새 결정/작업 시 즉시. 정기 리뷰는 주 1회 (월요일 권장).
- **Decision Log 와의 관계**: 결정으로 닫히는 항목은 `D-###` 부여 후 [[Infrastructure/31 - Decision Log]] 로 이동, 본 보드의 ✅ 항목으로 정리.

---

> 처음으로: [[CLAUDE]]
