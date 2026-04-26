---
title: Deployment State
parent: "[[00 - Infrastructure (Index)]]"
project: 창신
created: 2026-04-24
updated: 2026-04-24 (dev2)
tags:
  - infrastructure
  - state
  - 창신
---

# Deployment State (2026-04-24 기준)

> 상위 문서: [[00 - Infrastructure (Index)]]

> [!summary]
> AWS(single account, dev env) · EKS + Flux + observability 스택까지 구축 완료. 앱 서비스 리포 4개 중 3개 보일러플레이트 초기화 완료. Auth 서비스 구현이 다음 단계.

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

### 앱 리포 (4개 중 3개)

| 서비스 | GitLab 리포 | 초기화 상태 |
|---|---|---|
| Auth | `changshin/changshin-auth-api` | ✅ 보일러플레이트 push (커밋 `9163aa4`) |
| AX1 | `changshin/changshin-api` (기존) | ⏭️ 본 프로젝트 범위 외 |
| AX2 | `changshin/changshin-ax2-api` | ✅ 보일러플레이트 push (커밋 `a9857ce`) |
| AX3 | `changshin/changshin-ax3-api` | ✅ 보일러플레이트 push, placeholder (커밋 `ee90069`) |

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
| ECR (auth) | `687582709363.dkr.ecr.ap-northeast-2.amazonaws.com/changshin-auth` |
| ECR (ax1) | `687582709363.dkr.ecr.ap-northeast-2.amazonaws.com/changshin-ax1` |
| ECR (ax2) | `687582709363.dkr.ecr.ap-northeast-2.amazonaws.com/changshin-ax2` |
| ECR (ax3) | `687582709363.dkr.ecr.ap-northeast-2.amazonaws.com/changshin-ax3` |
| ECR (iac/Flux OCI) | `687582709363.dkr.ecr.ap-northeast-2.amazonaws.com/changshin-iac` |
| S3 temp | `changshin-temp` |
| Terraform state S3 | `changshin-terraform-backend` |

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

### 단기 (현재 사이클)

- [x] **Auth 보일러플레이트 정리 + 배포 파이프라인** (2026-04-24 완료)
  - [x] `apps/` 예시 4개 삭제(`admin-api`, `batch`, `llm-api`, `mcp-server`)
  - [x] `apps/user-api` → `apps/auth` 리네임 (git mv로 히스토리 보존)
  - [x] `apps/auth/{package.json, nest-cli.json, Dockerfile, tsconfig.json}` 경로/이름 수정
  - [x] `apps/auth/src/user-api.module.ts` + `api-docs/index.ts`에서 admin-api import 제거
  - [x] `.gitlab-ci.yml`: `ecr_repo=changshin-auth`, `oidc_role=changshin-oidc-ci`, 브랜치 `main→prod`, `dev→dev`
  - [x] K8s 매니페스트 (`k8s/clusters/dev/auth/`): deploy/service/ingress/parameter-store/secrets-manager 전부 교체
  - [x] `pnpm install` + `turbo build --filter=auth` 성공 검증
  - ⚠️ **변경사항 아직 커밋 안 됨** (working tree에 머물러 있음)
- [ ] **Auth 서비스 구현** ← 다음 세션 시작점
  - [ ] ⚠️ 결정 필요 3가지 (아래 "다음 세션 시작 전 결정" 참조)
  - [ ] D-007 `aud` 기반 JWT 발급 로직 (`/auth/{service}/login`, `/auth/.well-known/jwks.json`)
  - [ ] D-009 자체 JWT (users 테이블, bcrypt/argon2 비밀번호 해시, 비밀번호 재설정 등)
  - [ ] D-002 공유 DB 스키마 `auth.*` 마이그레이션
  - [ ] 내부 src 리네임 (선택): `user-api.module.ts` → `auth-app.module.ts`, `UserApiModule` → `AuthAppModule` 등
- [ ] **Auth 배포 선행 작업**
  - [ ] SSM Parameter Store에 JWT 키 pair 업로드 (`/changshin/dev/auth/jwt/{private-key,public-key}`)
  - [ ] Ingress host 최종 확정 (현재 placeholder)
- [ ] **AX2 서비스 구현** (Auth 검증 적용)
- [ ] **changshin-iac의 `.gitlab-ci.yml` 생성** (`./ci.sh`) — Terraform plan/apply 자동화

### 다음 세션 시작 전 결정 필요

1. **Ingress host** — 현재 `api.dev.changshin.weplanet.co.kr` placeholder. AX1/2/3과 ALB/host 공유 규칙 확정 필요. 후보:
   - A안: 단일 host + path 라우팅 (`api.dev.changshin.weplanet.co.kr/auth`, `.../ax1`, ...)
   - B안: 서비스별 host (`auth.dev.changshin...`, `ax1.dev.changshin...`) — D-011은 path 라우팅 명시했으므로 A안 유력
2. **SSM JWT 키 경로** — `/changshin/dev/auth/jwt/{private-key,public-key}` 제안. terraform으로 관리할지(iac 리포에 넣기) 수동 put 할지.
3. **내부 src 리네임** — Auth 도메인 구현과 같이 묶어서 할지, 선행으로 먼저 정리할지.

### 다음 세션 재개 포인트

워킹 디렉터리: `/Users/yoohakseon/Documents/GitLab/changshin/changshin-auth-api`
- 브랜치: `main`, working tree에 리네임/수정 미커밋 상태
- 커밋 전에: 위 3개 결정 정리 → 커밋 → Auth 도메인 구현 시작

### 중기

- [ ] **AX3 서비스 구현** (현재 placeholder)
- [ ] **OIDC Provider 소유권 정리** (5번 항목)
- [ ] **Backend state 공유 방법** (`aws/env/backend/terraform.tfstate`가 로컬에만 있음)

### 장기

- [ ] **Azure 이전 (Phase 2, [[30 - Azure Migration]])**

---

## 관련 문서
- [[00 - Infrastructure (Index)]]
- [[31 - Decision Log]] — D-001 ~ D-014
