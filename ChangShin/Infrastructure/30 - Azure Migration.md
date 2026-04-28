---
title: Azure Migration Plan
parent: "[[00 - Infrastructure (Index)]]"
project: 창신
created: 2026-04-23
updated: 2026-04-28
status: draft
tags:
  - infrastructure
  - azure
  - migration
  - 창신
---

# Azure Migration Plan

> 상위 문서: [[00 - Infrastructure (Index)]]
> 이전: [[20 - AWS Deployment]]

> [!summary] 한 줄로 말하면
> **K8s 위에 단일 통합 API를 올렸기 때문에 매니페스트는 대부분 재사용.** 이전 비용은 Terraform 리라이트와 데이터 마이그레이션, 네트워크/보안/DNS 컷오버에 집중.

> [!warning] 2026-04-28 변경 반영
> 단일 통합 서비스로 변경됨에 따라 ([[31 - Decision Log#D-019|D-019]]) 이전 시 옮길 워크로드는 **API 1개 + Batch 1개**로 단순화됨.

---

## 1. 이전 전략

### 1-1. 접근 방식

- **Lift-and-shift가 아닌 Lift-and-adapt**: K8s 레이어는 그대로, 플랫폼 레이어(관리형 서비스)는 Azure 네이티브로 교체
- 어플리케이션 코드는 **가능한 한 변경 없음**

### 1-2. 전제

- 이미 [[10 - Architecture#4. 서비스 간 통신|서비스 간 통신]]·[[10 - Architecture#2. 공유 데이터베이스 전략|DB 접근]]에서 클라우드 중립 원칙을 지킨 상태
- 클라우드 특화 서비스는 최소 (Ex: DynamoDB, Step Functions, SQS를 **굳이 안 쓰는** 설계)

---

## 2. 서비스 매핑 (AWS → Azure)

| 영역 | AWS | Azure |
|------|-----|-------|
| **K8s** | EKS | **AKS** (Azure Kubernetes Service) |
| **네트워크** | VPC | VNet |
| **서브넷** | Public/Private/Data Subnet | VNet Subnets + NSG |
| **로드밸런서** | ALB | Azure Load Balancer / **Application Gateway** |
| **DB** | RDS (PostgreSQL/MySQL) | **Azure Database for PostgreSQL / MySQL** |
| **캐시** | ElastiCache | **Azure Cache for Redis** |
| **컨테이너 레지스트리** | ECR | **ACR** (Azure Container Registry) |
| **DNS** | Route53 | **Azure DNS** |
| **TLS 인증서** | ACM | **Azure Key Vault + App Gateway** |
| **시크릿** | Secrets Manager + External Secrets | **Key Vault** + External Secrets |
| **GitOps** | Flux | Flux (그대로) |
| **오토스케일링** | Karpenter | **KEDA + Cluster Autoscaler** (또는 Karpenter AKS 지원 확인) |
| **관측성** | CloudWatch | **Azure Monitor / Log Analytics** |
| **CI/CD** | CodePipeline | **Azure DevOps Pipelines** / GitHub Actions |
| **Bastion** | EC2 Bastion | **Azure Bastion** |
| **IAM** | IAM + IRSA | **Azure AD + Workload Identity** |

---

## 3. 이전 단계 (권장 순서)

### Phase A · 사전 준비 (이전 시작 전 지속 점검)

- [ ] K8s 매니페스트에 AWS 특화 설정 최소화
- [ ] External Secrets 사용 (Secret 추상화)
- [ ] 환경 변수/ConfigMap으로 클라우드 리전 분리
- [ ] 관측성을 OpenTelemetry로 (벤더 중립)

### Phase B · Azure 환경 구축

- [ ] Azure 구독 · Resource Group 생성
- [ ] Terraform Azure Provider로 인프라 코드 작성 (기존 AWS 모듈을 참조해 1:1 매핑)
- [ ] AKS 클러스터 · ACR · Azure DB · Key Vault · Azure DNS 세팅
- [ ] Flux를 Azure AKS에 설치, 같은 매니페스트 저장소 연결
- [ ] dev 환경 먼저 이전해 검증

### Phase C · 데이터 마이그레이션

- [ ] RDS → Azure DB 스키마 이전 (pg_dump/mysqldump 또는 DMS)
- [ ] 애플리케이션 기동 테스트 (DB 연결 문자열만 교체)
- [ ] 이전 시점 **데이터 동기화 전략** 결정 (one-shot vs 이중 쓰기 vs CDC)
- [ ] 마이그레이션 리허설 1회 이상

### Phase D · 컷오버

- [ ] Azure 환경 트래픽 테스트 (내부)
- [ ] DNS TTL 단축 (예: 300s)
- [ ] 점진적 트래픽 이동 (Route53 weighted → Azure DNS)
- [ ] 롤백 계획 수립
- [ ] AWS 리소스 decommission (일정 유예 기간 후)

---

## 4. 주의사항 · 리스크

> [!warning] 핵심 리스크
> - **관리형 서비스 엔진 호환성**: PostgreSQL/MySQL은 호환성 높지만 Aurora 전용 기능 사용 시 재작업 필요
> - **IAM 모델 차이**: AWS IRSA vs Azure Workload Identity — 코드 변경은 적지만 인프라 설정 다름
> - **네트워크 레이턴시**: 리전·고객 위치별로 측정 필요 (한국/일본 리전 기준)
> - **비용 산정**: 유사 스펙이라도 가격 구조 다름. 이전 전 TCO 비교 필수

> [!tip] 이전을 쉽게 만드는 습관
> - 로컬 개발은 **docker-compose / minikube / kind** 사용 (어디서든 동일)
> - Terraform 모듈을 **클라우드 공통 입력/출력**으로 감싸기
> - 시크릿은 **환경 변수 이름**으로만 참조, 출처(Secrets Manager / Key Vault) 분리

---

## 5. 이전 트리거 조건 (참고)

다음 중 하나 이상 충족 시 이전 검토:

- 창신 내부 Azure 표준화 정책 확정
- AWS 비용 대비 Azure 할인 혜택이 유의미
- Azure 전용 기능(예: Microsoft Fabric, Azure OpenAI) 필요성
- 고객(ODM/브랜드사) 요구사항

---

## 열린 질문

- [ ] 이전 예상 시점 (1년 내 / 3년 내)
- [ ] 일부만 Azure 이전 가능성 (하이브리드) 고려 여부
- [ ] AWS·Azure 모두 운영하는 기간 허용치
- [ ] 데이터 거버넌스(데이터 위치, 리전) 제약

---

> 다음: [[31 - Decision Log]]
