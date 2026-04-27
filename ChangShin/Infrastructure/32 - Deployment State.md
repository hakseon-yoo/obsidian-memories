---
title: Deployment State
parent: "[[00 - Infrastructure (Index)]]"
project: 창신
created: 2026-04-24
updated: 2026-04-27
tags:
  - infrastructure
  - state
  - 창신
---

# Deployment State (2026-04-27 기준)

> 상위 문서: [[00 - Infrastructure (Index)]]

> [!summary]
> AWS(single account, dev env) · EKS + Flux + observability 스택 완료. **Auth 서비스 첫 Pod이 dev 클러스터에 떠서 외부 HTTPS로 health 응답 확인.** Swagger도 외부 노출. 다음은 Auth 도메인 로직 구현(D-007 aud 분기, D-009 자체 JWT) → AX1 띄우기.

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
| Auth | `changshin/changshin-auth-api` | ✅ **첫 Pod 외부 HTTPS 노출 완료** (2026-04-27) |
| AX1 | `changshin/changshin-api` (기존) | ⏭️ 본 프로젝트 범위 외 |
| AX2 | `changshin/changshin-ax2-api` | ✅ 보일러플레이트 push (커밋 `a9857ce`) |
| AX3 | `changshin/changshin-ax3-api` | ✅ 보일러플레이트 push, placeholder (커밋 `ee90069`) |

### Auth 서비스 배포 (2026-04-27)

| 항목 | 상태 |
|---|---|
| `apps/auth` 보일러 정리 (apps 4개 삭제, user-api → auth 리네임) | ✅ |
| `.gitlab-ci.yml`: `ecr_repo=changshin-auth`, `ecr_deploy=changshin-auth-deploy`, OIDC role, 브랜치 매핑 | ✅ |
| `globalPrefix='auth'` + AuthController prefix 정리 + `/auth/health` | ✅ |
| Swagger를 `/auth/api-docs` 아래로 이동 (외부 노출) | ✅ |
| K8s 매니페스트(`k8s/clusters/dev/auth/`): deploy/service/ingress/secrets-manager | ✅ |
| Flux GitOps 사이클 (push → CI → ECR → Flux → Pod) | ✅ |
| `auth-jwt` K8s Secret (RSA 키, 수동 생성) | ✅ |
| `auth-api-docs` K8s Secret (swagger basic auth, 수동 생성) | ✅ |
| Pod Identity association (`changshin-svc-auth` IAM Role) | ✅ |
| Ingress + ALB(internet-facing) + Route53 alias 레코드 | ✅ |
| `https://changshin-api.dev.weplanet.co.kr/auth/health` 200 | ✅ |
| `https://changshin-api.dev.weplanet.co.kr/auth/api-docs/swagger` 접근 | ✅ |

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
| ECR (auth-deploy, Flux artifact) | `687582709363.dkr.ecr.ap-northeast-2.amazonaws.com/changshin-auth-deploy` |
| ECR (ax1 / ax1-deploy) | `…/changshin-ax1`, `…/changshin-ax1-deploy` |
| ECR (ax2 / ax2-deploy) | `…/changshin-ax2`, `…/changshin-ax2-deploy` |
| ECR (ax3 / ax3-deploy) | `…/changshin-ax3`, `…/changshin-ax3-deploy` |
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

### 단기 (현재 사이클) ← **다음 세션 재개 포인트**

- [ ] **Auth 도메인 구현 본격** [[31 - Decision Log#D-007]] [[31 - Decision Log#D-009]]
  - [ ] D-007 `aud` 분기 로그인 (`/auth/{ax1|ax2|ax3}/login`) — 단일 JwtStrategy를 aud 동적으로 확장
  - [ ] `/auth/.well-known/jwks.json` — RSA 공개키 JWK 노출 (AX 서비스 검증 진입점)
  - [ ] D-009 자체 JWT — users 테이블, bcrypt/argon2 해시, 비밀번호 재설정·이메일 인증
  - [ ] D-002 `auth.*` 스키마 마이그레이션
  - [ ] 보일러 잔재 정리 — 소셜 로그인(D-009 자체 JWT 결정에 따라), files/faqs/notices/verifications/admin 등 Auth 본질과 무관한 컨트롤러
  - [ ] 내부 src 리네임 (`user-api.module.ts` → `auth-app.module.ts` 등) — 도메인 구현 중 자연스럽게 정리
- [ ] **AX1 서비스 띄우기** — Auth 도메인 어느 정도 완성된 후, 같은 패턴으로 (보일러 정리 → ECR/iac 등록 → Pod)
- [ ] **changshin-iac의 `.gitlab-ci.yml` 생성** — Terraform plan/apply 자동화 (현재는 로컬에서 수동 apply)

### 중기

- [ ] **AX2/AX3 서비스 구현** (Auth 검증 적용)
- [ ] **클라이언트 도메인 확정 시 인증서/host 교체** [[31 - Decision Log#D-015]]
- [ ] **80 listener 미동작 진단** — IngressClassParams는 80/443 listen 설정인데 실제 ALB SG는 443만 열려 있음. HTTPS-only로 갈지, 80→443 redirect 정식 적용할지 결정
- [ ] **OIDC Provider 소유권 정리** (5번 항목)
- [ ] **Backend state 공유 방법** (`aws/env/backend/terraform.tfstate`가 로컬에만 있음)

### 다음 세션 재개 포인트

워킹 디렉터리: `/Users/yoohakseon/Documents/GitLab/changshin/changshin-auth-api`
- 브랜치: `dev` (origin/dev 동기화), Auth Pod 클러스터에 정상 동작 중
- 시작점: **Auth 도메인 구현 (D-007 aud 분기, JWKS, D-009 자체 JWT)**
- 보일러의 기존 `apps/auth/src/controllers/auth/auth.controller.ts`·`auth.service.ts`가 이미 register/login/refresh/password/social 풀스택 구현 — `aud` 동적화 + JWKS 추가 + 소셜 부분 정리(또는 제거) 결정부터 시작하면 자연스러움

### 중기

- [ ] **AX3 서비스 구현** (현재 placeholder)
- [ ] **OIDC Provider 소유권 정리** (5번 항목)
- [ ] **Backend state 공유 방법** (`aws/env/backend/terraform.tfstate`가 로컬에만 있음)

### 장기

- [ ] **Azure 이전 (Phase 2, [[30 - Azure Migration]])**

---

## 관련 문서
- [[00 - Infrastructure (Index)]]
- [[31 - Decision Log]] — D-001 ~ D-018
