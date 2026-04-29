---
title: Deployment State
parent: "[[00 - Infrastructure (Index)]]"
project: 창신
created: 2026-04-24
updated: 2026-04-28
tags:
  - infrastructure
  - state
  - 창신
---

# Deployment State (2026-04-28 기준)

> 상위 문서: [[00 - Infrastructure (Index)]]

> [!warning] 2026-04-28 아키텍처 변경
> 4서비스 분리 → **단일 통합 API 서비스(`changshin-api`)**로 변경 ([[31 - Decision Log#D-019|D-019]]).
> 기존 4리포(`changshin-auth-api`, `changshin-ax2-api`, `changshin-ax3-api`)는 `changshin/changshin-legacy/` 로 이전. 빌드/배포 중단. 코드 자산은 `changshin-api`의 도메인 모듈로 흡수 진행 중.

> [!summary]
> AWS(single account, dev env) · EKS + Flux + observability 스택 운영 중. **단일 `changshin-api` 리포 + `apps/ax-api` Deployment**가 dev 클러스터에서 동작 중. 다음은 도메인 모듈(Auth · AX-2) 구현.

---

## 1. 구축 완료 체크리스트

### Terraform (changshin-iac)

| 단계 | 상태 | 비고 |
|---|---|---|
| `init.sh`로 `base.yml` 생성 | ✅ | `project_name=changshin`, `region=ap-northeast-2`, `domain=weplanet.co.kr` |
| Backend S3 버킷 | ✅ | `changshin-terraform-backend` |
| Base 리소스 | ✅ | ECR 5개 + OIDC Provider(GitLab) + S3 temp + ACM/Route53 참조 |
| `env/dev` 환경 | ✅ | VPC + EKS + RDS(PostgreSQL) + ElastiCache(Valkey) + ALB + EC2 Bastion |
| EKS kubeconfig | ✅ | 클러스터명: `changshin-dev` |
| Metrics Server | ✅ | |
| Flux OCI 아티팩트 push | ✅ | `changshin-iac` ECR, 태그: `6ea3fbc`/`development`/`latest` |
| Flux 동기화 (operators, monitoring, infra, node-pools) | ✅ | 10개 HelmRelease 전부 Ready |

### 앱 리포 (단일 통합 — D-019 적용 후)

| 리포 | 역할 | 상태 |
|---|---|---|
| **`changshin/changshin-api`** | 통합 API 단일 리포 (`apps/ax-api` + `apps/batch`) | ✅ **K8s에서 동작 중** (2026-04-28) |
| `changshin/changshin-legacy/changshin-auth-api` | (구) Auth 분리 리포 | 🗄️ legacy 보관 |
| `changshin/changshin-legacy/changshin-ax2-api` | (구) AX2 분리 리포 | 🗄️ legacy 보관 |
| `changshin/changshin-legacy/changshin-ax3-api` | (구) AX3 분리 리포 | 🗄️ legacy 보관 |

### 통합 API 배포 상태 (2026-04-28)

| 항목 | 상태 |
|---|---|
| `changshin-api` 리포 생성 + `apps/ax-api`, `apps/batch` 구조 정착 | ✅ |
| Auth 분리 리포의 `apps/auth/` 자산 → `changshin-api` 통합 (모듈로 흡수 진행) | ⏳ 진행 중 |
| `.gitlab-ci.yml`: `ecr_repo=changshin-{?}`, `ecr_deploy=changshin-{?}-deploy`, OIDC role | ⏳ ECR 통폐합 후속 결정 |
| K8s 매니페스트(`k8s/clusters/dev/ax-api/`, `k8s/clusters/dev/batch/`) | ✅ |
| Flux GitOps 사이클 | ✅ |
| `auth-jwt` K8s Secret (RSA 키, 수동) | ✅ (재활용) |
| Pod Identity association | ✅ |
| Ingress + ALB(internet-facing) + Route53 alias 레코드 | ✅ |
| `https://changshin-api.dev.weplanet.co.kr` 외부 접근 | ✅ |

> [!info] 도메인 모듈 통합 진행 메모
> 기존 `changshin-auth-api`(분리 시기)의 코드 자산(register/login/refresh/password/social, Swagger, gitlab-ci 등)을 `changshin-api`의 Auth 모듈로 흡수. 보일러 잔재 정리(verifications/admin 등)와 함께 진행.

---

## 2. 핵심 참조값

| 항목 | 값 |
|---|---|
| AWS Account ID | `687582709363` |
| Region | `ap-northeast-2` |
| EKS Cluster | `changshin-dev` |
| VPC CIDR | env.sh 기본값 (사용 중인 CIDR는 `terraform output` 또는 AWS 콘솔에서 확인) |
| Route53 Hosted Zone | `Z2ELFWHU9W34XL` (`weplanet.co.kr`) |
| ACM (ap-northeast-2) | `arn:aws:acm:ap-northeast-2:687582709363:certificate/d90d0636-ef31-425f-b1e1-cef68d699523` |
| ACM (us-east-1, CloudFront용) | `arn:aws:acm:us-east-1:687582709363:certificate/21bb6d33-c3a2-4b92-a821-d33d68601115` |
| OIDC Provider | `arn:aws:iam::687582709363:oidc-provider/git.weplanet.co.kr` |
| CI Role ARN | `arn:aws:iam::687582709363:role/changshin-oidc-ci` |
| OIDC Audience | `flux-deploy` |
| ECR (현재) | `…/changshin-auth`, `…/changshin-auth-deploy` 등 4세트 — D-019 후속 통폐합 예정 |
| ECR (iac/Flux OCI) | `687582709363.dkr.ecr.ap-northeast-2.amazonaws.com/changshin-iac` |
| S3 temp | `changshin-temp` |
| Terraform state S3 | `changshin-terraform-backend` |
| K8s Namespace (앱 워크로드) | `changshin` (D-017) |
| 임시 외부 호스트 | `changshin-api.dev.weplanet.co.kr` (D-015) |
| ALB DNS (auto-managed) | `k8s-default-1acb9e2c57-1088456801.ap-northeast-2.elb.amazonaws.com` |
| RDS endpoint | `changshin-dev-changshin.cgigcuul8yn2.ap-northeast-2.rds.amazonaws.com` |
| RDS Secrets Manager key | `changshin/dev/rds-changshin` (host/port/username/password/dbname property) |
| 수동 관리 K8s Secret | `auth-jwt`(RSA 키), `auth-api-docs`(swagger basic auth) — both in `changshin` ns |

---

## 3. 운영 명령 모음

### AWS 인증 (MFA 필수)
```bash
aws-vault exec weplanet.ethan -- <command>
# ex: aws-vault exec weplanet.ethan -- kubectl get pods -A
```

### kubeconfig
```bash
aws-vault exec weplanet.ethan -- aws eks update-kubeconfig \
  --region ap-northeast-2 --name changshin-dev --alias changshin-dev
```

### Flux OCI 재푸시 (iac 리포)
```bash
cd changshin-iac && aws-vault exec weplanet.ethan -- ./scripts/flux-push.sh
```
스크립트가 `development`/`latest`/`<git sha>` 3개 태그 모두 push.

### Flux 상태 확인 / 재동기화
```bash
aws-vault exec weplanet.ethan -- flux get all -A
aws-vault exec weplanet.ethan -- flux reconcile source oci -n flux-system flux-system
```

### Terraform plan/apply (dev 환경)
```bash
cd changshin-iac/aws/env/dev && aws-vault exec weplanet.ethan -- terraform plan
cd changshin-iac/aws/env/dev && aws-vault exec weplanet.ethan -- terraform apply
```

---

## 4. 수정한 스타터 결함 (다음 사람 참고)

| 파일 | 원본 문제 | 수정 내용 |
|---|---|---|
| `aws/env/backend/main.tf` | `module "base"`에 정의되지 않은 `region` 인자 전달 → `Unsupported argument` | `region` 인자 제거 + `provider "aws" { region = ... }` 블록 추가 |
| `docs/ops/00-quickstart.md` 6단계 | `aws ecr get-login-password \| flux push artifact ...` 파이프 방식 → 403 Forbidden (flux가 stdin에서 자격증명을 읽지 않음) | `scripts/flux-push.sh` 추가. `--creds "AWS:$PASSWORD"` 플래그 사용 |

추후 보일러플레이트에 피드백 제출 권장.

---

## 5. OIDC Provider 공용 리소스 주의

OIDC Provider `git.weplanet.co.kr`는 **AWS 계정당 1개만 존재 가능**한 공유 리소스. 현재 `changshin-iac`의 terraform state로 import되어 있으며, 이에 따라 **태그 `Environment=base`, `Project=weplanet`이 제거됨**. 원래 소유자(weplanet 내부 프로젝트)가 apply하면 태그 drift 발생 가능. 중장기적으로 전용 "bootstrap" terraform 또는 사내 관례 합의 필요.

---

## 6. 남은 작업 (우선순위 순)

### 완료 (2026-04-24 ~ 04-27)

- [x] **Auth 보일러 정리** — apps 4개 삭제, `user-api → auth` 리네임, Dockerfile/nest-cli/tsconfig 정리, admin-api 의존 제거
- [x] **Auth 배포 파이프라인** — `.gitlab-ci.yml`(ECR, OIDC, 브랜치 매핑), K8s 매니페스트 세트 (`deploy/service/ingress/secrets-manager`)
- [x] **Auth 결정 3가지** — Ingress host(임시) [[31 - Decision Log#D-015]], JWT 키 저장(K8s Secret 직접) [[31 - Decision Log#D-018]], 내부 src 리네임(Auth 도메인 구현과 함께)
- [x] **iac에 service 등록** — `aws/env/dev/config.yml` `eks.services` 활성화, namespace=`changshin` [[31 - Decision Log#D-017]], ECR `*-deploy` 4개 추가 [[31 - Decision Log#D-016]]
- [x] **iac 첫 적용** — `terraform apply` (Pod Identity, IAM Role, Flux OCIRepository/Kustomization 생성)
- [x] **첫 Pod 외부 노출 검증** — Route53 alias + ACM `*.dev.weplanet.co.kr` 와일드카드 + ALB → `https://changshin-api.dev.weplanet.co.kr/auth/health` 200 OK
- [x] **Swagger 외부 접근** — `globalPrefix='auth'` 적용, swagger documentPath `/auth/api-docs`로 이동, basic auth 자격증명을 `auth-api-docs` Secret으로 강화
- [x] **SKU 이미지 S3 + DB 1차 반영** (2026-04-29) — `weplanet-files/images/skus/<uuid>.png` 287개 업로드, `Sku.imageUrl` 풀 cloudfront URL UPDATE rows=289, 매칭 실패 2 (후속 정규화 대상). 보일러 컨벤션(legacy `files.controller`) 정렬 [[31 - Decision Log#D-022]]

### 단기 / 중기 / 장기 작업 — Action Board 로 이전

> [!tip] 단일 보드로 이전됨
> 미완 작업은 [[00 - Action Board]] 에서 단일 소스로 관리. 본 문서는 **현재 운영 상태와 참조값**에 집중.
>
> - **현재 사이클 P0**: [[00 - Action Board#🔥 현재 사이클 (본인 P0 — 직접 진행)]]
> - **백로그**: [[00 - Action Board#📥 백로그 (다음 사이클 / 결정·답변 도착 시 진행)]]
> - **장기**: [[00 - Action Board#📅 장기 / Phase 4]]

### 다음 세션 재개 포인트

워킹 디렉터리: `/Users/yoohakseon/Documents/GitLab/changshin/changshin-api`
- 단일 통합 리포. `apps/ax-api`(API) + `apps/batch`(cron). 클러스터에 Pod 정상 동작 중.
- 시작점: **D-019 후속 정리** + **Auth 모듈 통합(legacy → ax-api)** + **AX-2 모듈 골격 잡기** — 자세한 항목은 [[00 - Action Board]]
- 참고: legacy 자산은 `/Users/yoohakseon/Documents/GitLab/changshin/changshin-legacy/` (필요한 코드는 cherry-pick)

---

## 관련 문서
- [[00 - Infrastructure (Index)]]
- [[31 - Decision Log]] — D-001 ~ D-018
