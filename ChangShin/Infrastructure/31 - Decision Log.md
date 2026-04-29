---
title: Decision Log
parent: "[[00 - Infrastructure (Index)]]"
project: 창신
created: 2026-04-23
tags:
  - infrastructure
  - decision-log
  - 창신
---

# Decision Log

> 상위 문서: [[00 - Infrastructure (Index)]]
> 이전: [[30 - Azure Migration]]

> [!info] 쓰는 방법
> 주요 결정이 있을 때마다 **하나의 섹션**으로 추가. 각 결정은 `D-###` 번호 부여.
> 맥락이 변해 결정이 바뀌면 기존 항목을 수정하지 말고 **새 번호로 추가**하고 이전 항목에 "superseded by D-###" 주석을 남긴다.

### 템플릿

```markdown
## D-### 결정 제목

- **일자**: YYYY-MM-DD
- **상태**: proposed / accepted / superseded / rejected
- **결정**: (무엇을 정했는가)
- **맥락**: (왜 이 결정이 필요했나)
- **대안**: (검토한 다른 옵션)
- **근거**: (이유)
- **영향**: (이 결정으로 무엇이 바뀌는가)
```

---

## D-001 마이크로서비스 구조 채택

- **일자**: 2026-04-23
- **상태**: **superseded by [[#D-019]]** (2026-04-28 단일 서비스로 변경)
- **결정**: AX1, AX2, AX3, Auth를 **각각 독립 API 서버**로 구성. 총 4개 서비스.
- **맥락**: 창신 AX 시리즈가 3개 서비스로 구성되며, 서비스별 배포·스케일·팀 경계를 독립적으로 운영할 필요가 있음.
- **대안**:
  - 단일 모놀리식: 초기 속도는 빠르지만 향후 분리 비용 큼
  - 완전한 MSA (DB도 분리): 이상적이지만 초기 관리 부담 과다
- **근거**: 런타임은 분리하되 데이터는 공유해 **중간 지점** 선택 (D-002 참고)
- **영향**: K8s 네임스페이스·배포·CI 파이프라인을 서비스 단위로 구성

---

## D-002 공유 DB 채택 (논리 분리)

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: 4개 서비스가 **하나의 물리 DB 인스턴스를 공유**. 스키마(`auth.*`, `ax1.*`, `ax2.*`, `ax3.*`, `common.*`) 단위 논리 분리.
- **맥락**: 사용자·권한 등 공통 데이터가 많고, 초기 운영 부담을 줄여야 함.
- **대안**:
  - 서비스별 DB 완전 분리: 결합도↓, 운영·비용↑
  - 단일 스키마 공유: 결합도↑, 향후 분리 불가능
- **근거**: 초기엔 하나로 시작하되, **스키마 단위로 명확히 분리**해 향후 독립 DB 분리 비용을 낮춤.
- **영향**:
  - 서비스는 자기 스키마 + `common` 읽기만 허용
  - 다른 서비스 스키마 직접 접근 금지 (DB 계정 권한으로 강제)
  - ORM 설정·마이그레이션 스크립트를 서비스별로 분리

---

## D-003 Kubernetes 기반 (클라우드 중립)

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: 모든 워크로드를 **Kubernetes 위에 올림.** AWS는 EKS, Azure 이전 시 AKS.
- **맥락**: [[30 - Azure Migration|Azure 이전]]이 미래에 예정되어 있음. 클라우드 종속 최소화 필요.
- **대안**:
  - ECS/Fargate: AWS 전용 → 이전 시 재작업 부담
  - VM 기반: 오케스트레이션 부재, 스케일링·자동화 불리
- **근거**: K8s 매니페스트는 클라우드 이전 시 **거의 그대로 재사용** 가능
- **영향**: 팀이 K8s 운영 지식 학습 필요 (보일러플레이트가 완화)

---

## D-004 IaC로 Terraform 사용 (+ 사내 보일러플레이트)

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: 인프라는 **Terraform으로만 관리**. 사내 `weplanet-starter-iac` 보일러플레이트를 기반으로 확장.
- **맥락**: 이미 검증된 사내 보일러플레이트가 존재. 재사용이 효율적.
- **대안**: AWS CDK, Pulumi, 수동 콘솔 세팅
- **근거**: Terraform은 클라우드 중립(Azure Provider 존재). 보일러플레이트가 EKS·RDS·Flux·Karpenter 등 필요한 것을 전부 커버.
- **영향**: 이전 시 `aws/` 구조와 동일하게 `azure/` 구조 생성하면 재활용 용이

---

## D-005 GitOps with Flux

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: K8s 매니페스트 배포는 **Flux** 사용.
- **맥락**: 수동 `kubectl apply` 관리는 감사·롤백·멀티 환경에서 취약.
- **대안**: ArgoCD, Helm + CI 직접 배포
- **근거**: 보일러플레이트에 이미 Flux 모듈 존재. Git이 Source of Truth.
- **영향**: K8s 매니페스트 별도 저장소 필요

---

## D-006 NestJS 보일러플레이트 기반 서버 구성

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: 4개 서비스(Auth, AX1, AX2, AX3)를 **사내 `weplanet-starter-nestjs` 보일러플레이트 기반**으로 구성.
- **맥락**: 검증된 사내 보일러플레이트 존재. NestJS monorepo 구조와 `.claude/rules` · `.claude/skills` 규칙 자산 포함.
- **대안**: 프레임워크 재선정(Express · Fastify · Hono 등) — 학습·초기 세팅 비용 발생
- **근거**: 보일러플레이트에 Auth 엔드포인트 · 가드 · Swagger · TypeORM 컨벤션이 이미 있어 개발 시간 절감
- **영향**:
  - 코딩 규칙은 보일러플레이트의 `.claude/rules/*`를 따른다
  - 자세한 서비스 구성: [[Backend/01 - Overview]], [[Backend/20 - Service Template]]

---

## D-007 인증 전략 · `aud` 기반 JWT

- **일자**: 2026-04-23
- **상태**: **superseded by [[#D-019]]** (단일 서비스로 변경되어 서비스 간 `aud` 분기 의미 약화. JWT 자체는 유지하되 `aud` 분기는 제거 또는 클라이언트 컨텍스트 구분용으로 재정의)
- **결정**: 중앙 Auth 서버가 JWT 발급 시 **`aud` 클레임으로 대상 AX 서비스를 분기**한다. 각 AX 서비스는 자신의 `aud`가 아닌 토큰은 거부.
- **맥락**: 하나의 사용자 계정이 여러 AX 서비스를 구분해 접근해야 함. SSO 유사하되 서비스별 토큰 경계 필요.
- **대안**:
  - 서비스별 독립 인증 시스템: 사용자 관리 중복·UX 열화
  - OAuth2 Client Credentials 별도 등록: 구현 복잡도 증가
- **근거**: `aud`는 JWT 표준 클레임이라 의미·검증이 명확. NestJS 가드로 간단히 구현 가능.
- **영향**:
  - Auth 서버가 `/auth/{ax1|ax2|ax3}/login` 형태로 발급 경로 분기
  - 각 AX 서비스는 `EXPECTED_AUD` 설정값 기반 가드로 검증
  - 자세한 설계: [[Backend/10 - Auth Strategy]]

---

## D-008 RDS 엔진 = PostgreSQL

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: 공유 RDS 엔진으로 **PostgreSQL**을 채택한다.
- **맥락**: [[20 - AWS Deployment]]와 [[30 - Azure Migration]]에서 DB 엔진이 미확정 상태였고, Terraform(`modules/rds-single`) 작성 전에 확정 필요.
- **대안**:
  - MySQL: 운영 경험은 많지만 JSONB·CTE·스키마 기능이 PostgreSQL 대비 약함
  - Aurora(PostgreSQL 호환): 기능·성능 우수하나 AWS 특화 → [[30 - Azure Migration|Azure 이전]] 시 호환성·재작업 리스크
- **근거**:
  - 스키마 단위 논리 분리(D-002) 전략에서 **PostgreSQL의 스키마(schema) 개념이 네이티브로 일치**해 권한·분리 관리가 자연스러움
  - Azure 이전 시 **Azure Database for PostgreSQL**로 1:1 매핑 가능 (클라우드 중립성 확보)
  - JSONB · CTE · 파티셔닝 등 고급 기능이 AX 시리즈 비즈니스 로직에 유리
- **영향**:
  - `aws/modules/rds-single` 사용 시 `engine = "postgres"` 고정
  - ORM(NestJS TypeORM)은 PostgreSQL 드라이버(`pg`) 기준으로 설정
  - 마이그레이션·백업 도구도 PostgreSQL 계열(`pg_dump`, `pg_restore`)로 표준화

---

## D-009 인증 방식 = 자체 JWT

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: 외부 IdP(Google · Azure AD · Auth0 등)에 의존하지 않고, **Auth 서버가 직접 사용자 인증을 수행하고 JWT를 발급**한다.
- **맥락**: D-007에서 `aud` 클레임 기반 JWT 구조는 확정되었으나, **토큰 발급 주체**(자체 vs 외부 IdP 연동)가 미정이었음.
- **대안**:
  - **OIDC 외부 IdP**: Google/Azure AD 등을 ID 공급자로 위임. 보안·SSO 무료로 얻지만 외부 의존·계약/비용 발생
  - **하이브리드**(자체 + OIDC): 초기엔 과설계
- **근거**:
  - 서비스의 사용자 경계가 사내로 한정되지 않으며, 독립된 사용자 DB가 필요
  - 외부 IdP에 대한 계약·비용·장애 영향을 지금 시점에 떠안을 이유가 없음
  - D-006의 NestJS 보일러플레이트에 Auth/가드/Swagger 컨벤션이 이미 구비
- **영향**:
  - 사용자 저장소(users 테이블)·비밀번호 해시(bcrypt/argon2)·비밀번호 재설정·이메일 인증 등을 **Auth 서비스가 직접 구현**
  - 공개키는 Auth의 `/auth/.well-known/jwks.json`으로 노출, AX1/2/3 서비스가 검증 ([[10 - Architecture#3-2. 인증 플로우]])
  - 추후 SSO·소셜 로그인 요구가 생기면 **하이브리드로 확장**(기존 자체 JWT 위에 OIDC 클라이언트 추가) — 별도 결정 필요
  - 2FA · 계정 잠금 · 감사 로그 등 보안 기능을 **자체 로드맵으로 관리**

---

## D-010 서비스 간 통신 = 동기 REST

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: 서비스 간 직접 호출이 필요한 경우 **K8s 내부 ClusterIP 서비스를 통한 동기 REST 호출**을 기본으로 한다. 메시지 큐(SQS · Kafka 등) 인프라는 **현 시점 미도입**.
- **맥락**: [[10 - Architecture#4. 서비스 간 통신|서비스 간 통신]]에서 "직접 호출 지양"을 원칙으로 정했으나, 공유 DB로 해결되지 않는 호출이 필요할 때의 구현 방식이 미정이었음.
- **대안**:
  - **비동기 이벤트 큐**(SQS · RabbitMQ · Kafka): 장애 격리·부하 완충 우수하나 초기 인프라·운영 부담 큼
  - **혼합**: 이상적이나 조기 최적화
- **근거**:
  - 초기 유스케이스가 **공유 DB · stateless JWT로 대부분 해결**됨 (D-002, D-007)
  - 실제 서비스 간 호출 케이스가 많지 않을 것으로 예상
  - 큐 인프라(운영 · 모니터링 · 중복/순서 처리)를 선제 도입할 이유가 없음
- **영향**:
  - 서비스 간 호출은 `http://{service}.{namespace}.svc.cluster.local` 경로로 수행
  - Auth 검증은 여전히 **stateless JWT**(D-007), Auth 서버 호출 없음
  - **나중에 이벤트 기반이 필요해지는 트리거**가 생기면(예: 대량 배치 · 이메일 발송 · 외부 ERP 연동 · AX2 스케줄러의 지연 실행) **별도 D-### 로 SQS/RabbitMQ 도입 재검토**
  - 동기 REST 특유의 장애 전파를 완화하기 위해 **타임아웃 · 서킷 브레이커(예: NestJS `axios` + `nestjs-circuit-breaker`)** 적용을 서비스 보일러플레이트 수준에서 표준화

---

## D-011 Ingress 라우팅 = Path 기반

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: 단일 도메인(예: `api.changshin.io`) 하에 **경로(path) prefix로 서비스를 분기**한다. 서브도메인 분리는 사용하지 않는다.
- **맥락**: [[10 - Architecture#3-1. 라우팅]]에서 path vs 서브도메인이 미정이었고, ALB Ingress·ACM·Route53 구성 전 확정 필요.
- **대안**:
  - **서브도메인 기반**: `auth.changshin.io`, `ax{1,2,3}.changshin.io` — 독립성 ↑, DNS/인증서 관리 부담 ↑
  - **혼합**: Auth만 서브도메인 분리 — 조기 최적화
- **근거**:
  - 서비스 수가 4개로 적고, 내부/단일 프론트 중심 API 사용 예상
  - ACM 인증서·Route53 레코드·ALB 리스너 규칙이 **1세트로 끝나** 관리 단순
  - 같은 origin → **CORS 설정 부담 최소**
  - K8s Ingress 매니페스트가 단순해 Flux GitOps 배포 편의성 ↑
- **영향**:
  - 라우팅 규칙:
    - `/auth/*` → `auth-service`
    - `/api/ax1/*` → `ax1-service`
    - `/api/ax2/*` → `ax2-service`
    - `/api/ax3/*` → `ax3-service`
  - 각 NestJS 서비스는 `app.setGlobalPrefix('api/ax1')` 등으로 prefix를 자체 인식
  - Swagger 경로도 서비스별 prefix 하위(`/api/ax1/docs`)에 위치
  - 공통 경로(`/health`, `/metrics`) 충돌 방지 위해 **서비스별 prefix 내부에 배치**
  - 나중에 트래픽 분리·독립 배포 요구가 커지면 **서브도메인으로 전환은 Ingress 규칙만 바꿔** 대응 가능

---

## D-012 환경 계정 분리 = 단일 AWS 계정 + 환경 논리 분리

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: 단일 AWS 계정 안에서 **VPC · 리소스 이름 prefix · 태그 · IAM Role로 환경(dev/stage/prod)을 논리적으로 분리**한다. 환경별 AWS 계정 분리는 하지 않는다.
- **맥락**: [[20 - AWS Deployment#5. 환경별 Terraform 구성]]에서 환경 계정 분리 전략이 미정이었고, Terraform 백엔드·provider·IAM 설계 전 확정 필요.
- **대안**:
  - **환경별 AWS 계정 분리** (Organizations + account 3개): 격리 강하지만 초기 세팅 부담·Cross-account 설정 필요
  - **혼합** (prod만 분리, non-prod 통합): 실무 절충안이나 Organizations 관리 부담은 여전
- **근거**:
  - 초기 팀/운영 규모 대비 Organizations 세팅은 과도
  - Terraform 보일러플레이트(D-004)가 **`aws/env/{dev,stage,prod}/`** 구조로 이미 환경 분리 state를 지원
  - 리소스 이름 prefix·태그·VPC 분리만으로 초기 수준의 격리 달성 가능
  - 비용·IAM·결제가 **단일 계정에서 단순**하게 유지됨
- **영향**:
  - Terraform state: `aws/env/dev/`, `aws/env/stage/`, `aws/env/prod/` 환경별 workspace/디렉터리 분리
  - 리소스 네이밍: `changshin-{env}-*` 규칙으로 통일 (예: `changshin-prod-eks`, `changshin-dev-rds`)
  - 태그 표준: `Environment={dev|stage|prod}`, `Project=changshin`, `ManagedBy=terraform`
  - IAM: 환경별 Role로 접근 권한 분리 (개발자는 dev만, prod는 소수 인원)
  - VPC: 환경별 별도 VPC (CIDR 중복 금지, 예: dev=10.10.0.0/16, stage=10.20.0.0/16, prod=10.30.0.0/16)
  - **프로덕션 실수 방지 가드레일**이 단일 계정 대비 필수 → prod 전용 IAM·MFA·`prevent_destroy` Terraform 속성·변경 승인 프로세스 도입
  - **나중에 prod만 별도 계정으로 분리**하려는 요구가 생기면 리소스 네이밍·태그가 이미 환경 인지적이라 이관 비용이 낮음 (별도 D-### 필요)

---

## D-013 K8s 매니페스트 저장소 = 하이브리드

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: K8s 매니페스트를 **인프라용**과 **앱용**으로 나누어 서로 다른 저장소에 둔다.
  - **인프라·플랫폼 매니페스트**(operators, monitoring, infra, node-pools, cluster 설정): `changshin-iac` 리포의 `k8s/`
  - **앱 매니페스트**(Deployment · Service · Ingress · External Secret 등): **각 앱 리포지토리 내부** `k8s/clusters/{env}/{service}/`
- **맥락**: [[20 - AWS Deployment#4-3. GitOps 구조 (Flux)|GitOps 구조]]에서 매니페스트 저장소 전략(모노리포 vs 중앙 분리)이 미정. 데브옵스 담당자가 이미 사내 `weplanet-starter-nestjs` 계열 앱 리포에 `apps/` + `packages/` + `k8s/clusters/{env}/{svc}/` 구조로 구성해둔 선례 존재.
- **대안**:
  - **중앙 매니페스트 저장소**(예: `changshin-k8s-apps`): 환경 승격 워크플로우 깔끔, 하지만 앱 리포와 분리되어 PR 리뷰가 두 곳에서 진행
  - **인프라 리포에 앱까지 통합**: CI 봇 커밋으로 인프라 이력 오염
- **근거**:
  - 이미 사내에 **검증된 패턴**(pnpm + turbo 모노리포 + 자체 `k8s/`)이 존재
  - 앱 코드 변경과 매니페스트 변경을 **한 PR에서 리뷰** 가능 (예: 환경변수 추가 시 코드 + ConfigMap 동시)
  - 앱 리포가 자기 자신만 커밋하면 되므로 **CI 봇 무한 루프/권한 확장 문제 없음**
  - 인프라 매니페스트는 Terraform 모듈 변경과 함께 움직일 때가 많아 `changshin-iac`와 공존이 자연스러움
- **영향**:
  - `changshin-iac/k8s/` 하위에는 **operators · monitoring · infra · node-pools · clusters 템플릿**만 유지 (앱 매니페스트 두지 않음)
  - 각 앱 리포는 다음 구조를 따른다:
    ```
    (app-repo)/
    ├── apps/               ← NestJS 서비스 소스 (pnpm + turbo)
    ├── packages/           ← 공유 패키지
    ├── k8s/clusters/
    │   ├── dev/{svc}/      ← 환경별 매니페스트 세트
    │   ├── stage/{svc}/
    │   └── prod/{svc}/
    └── .gitlab-ci.yml      ← 브랜치 기반 CI → ECR push + 해당 env 디렉터리 image tag 자체 커밋
    ```
  - **Flux 구성**: 각 앱 리포마다 `GitRepository` + `Kustomization` 리소스를 `changshin-iac/k8s/clusters/{env}/` 수준에서 선언 → 앱 리포 추가 시 인프라 리포에 1회 등록
  - **브랜치 ↔ 환경 매핑**: 각 앱 리포의 CI가 `develop→dev`, `release→stage`, `main→prod` 규칙으로 매니페스트의 이미지 태그 업데이트
  - **환경 승격 워크플로우**: 앱 리포 내부에서 브랜치 머지로 처리 (중앙 매니페스트 저장소에서의 PR이 필요 없음)
  - 앱 간 공통 정책(공통 라벨·ResourceQuota 등)이 필요해지면 `changshin-iac/k8s/infra/`에 **공통 Policy 리소스**로 배포 (Kyverno · OPA Gatekeeper 등 도입은 별도 D-###)

---

## D-014 CI 플랫폼 = GitLab CI

- **일자**: 2026-04-23
- **상태**: accepted
- **결정**: 빌드·테스트·이미지 push·매니페스트 업데이트를 포함한 CI 파이프라인은 **GitLab CI**로 구축한다.
- **맥락**: [[20 - AWS Deployment#6. 배포 파이프라인 (초안)]]에서 CI 플랫폼 후보가 GitHub Actions / GitLab CI / CodePipeline으로 미정이었음. 리포지토리가 이미 `git.weplanet.co.kr`(GitLab)에 호스팅되며, 사내 `weplanet-starter-nestjs` 계열 앱 리포에도 `.gitlab-ci.yml` · `ci/` 구조가 이미 적용되어 있어 표준화 필요.
- **대안**:
  - **GitHub Actions**: 생태계·Marketplace 풍부하지만 GitLab → GitHub 미러링이 추가 축이 됨
  - **AWS CodePipeline**: 모듈(`modules/codepipeline`)이 보일러플레이트에 존재하나 **AWS 종속**, Azure 이전(D-003·[[30 - Azure Migration]]) 시 재작업 부담
- **근거**:
  - 소스·이슈·CI를 **GitLab 단일 플랫폼**으로 운영 → 토큰·SSO·권한 관리 단순
  - 사내 기존 앱 리포의 `.gitlab-ci.yml` 자산을 **재사용 · 표준화**
  - GitLab CI는 **클라우드 중립** → Azure 이전(AKS) 시에도 동일 파이프라인 유지 가능
  - Runner는 **self-hosted(EKS 위 GitLab Runner)** 또는 GitLab SaaS shared runner 선택 가능
- **영향**:
  - 각 앱 리포는 `.gitlab-ci.yml`로 브랜치 기반 파이프라인 정의 ([[#D-013]]의 환경 매핑과 연동)
  - `changshin-iac` 리포도 Terraform plan/apply를 GitLab CI 잡으로 구성 (MR에서 plan, main 머지 시 apply) — 자세한 구성은 후속 작업
  - **OIDC 인증**: `aws/modules/oidc-provider-ci` 모듈을 **GitLab OIDC** 설정으로 활성화해 CI가 long-lived AWS access key 없이 AssumeRole
  - **ECR 접근** · **매니페스트 저장소 커밋 권한**: GitLab CI/CD Variables(masked · protected)로 관리
  - **Runner 배치**: 초기에는 GitLab SaaS shared runner 사용, 비용·성능 이슈 발생 시 self-hosted Runner를 EKS(Karpenter spot)에 배치하는 방안을 별도 D-###로 검토
  - 보일러플레이트의 `modules/codepipeline` · `codepipeline-init`은 **사용하지 않음** → 환경별 Terraform에서 비활성화

---

## D-015 임시 도메인 호스트 = `changshin-api.dev.weplanet.co.kr`

- **일자**: 2026-04-27
- **상태**: accepted (임시, 클라이언트 도메인 확정 시 교체)
- **결정**: 클라이언트가 자체 도메인을 제공할 때까지 **임시 호스트 `changshin-api.dev.weplanet.co.kr`** 사용. D-011의 path 기반 라우팅 그대로 적용 (단일 host + `/auth`, `/ax1`, `/ax2`, `/ax3` prefix).
- **맥락**: D-011에서 라우팅 전략은 path 기반으로 확정했으나 실제 호스트값이 미정이었음. 첫 ALB 노출 시점에 결정 필요했고, 클라이언트 도메인은 아직 미수령.
- **대안**:
  - **changshin 전용 ACM 신규 발급** (`*.dev.changshin.weplanet.co.kr`) + Route53 별도 zone — 클라이언트 도메인 받기 전엔 과설계
  - **`api.dev.changshin.weplanet.co.kr`** 같은 4단계 subdomain — 기존 ACM 와일드카드(`*.dev.weplanet.co.kr`) 미커버 → 인증서 신규 발급 필요
- **근거**:
  - 기존 weplanet 공용 ACM `*.dev.weplanet.co.kr` 와일드카드가 즉시 매칭 → 인증서 작업 0
  - Route53 alias 레코드 1개로 외부 노출 완료
  - 클라이언트 도메인 확정 시 ingress의 `host` + 인증서 ARN만 바꾸면 됨 (마이그레이션 비용 낮음)
- **영향**:
  - Ingress: `host: changshin-api.dev.weplanet.co.kr`, path `/auth` Prefix
  - Route53 hosted zone `Z2ELFWHU9W34XL` (weplanet.co.kr) 에 alias 레코드 추가, ALB DNS로 향함
  - 모든 서비스(auth/ax1/ax2/ax3) ingress가 같은 host 공유하므로 ALB 1개로 통합 (`group.name: default`)
  - 클라이언트 도메인 받으면: 해당 도메인용 ACM 발급 → IngressClassParams 또는 cluster-vars 갱신 → ingress host 일괄 교체

---

## D-016 ECR 리포 분리 = `{svc}` + `{svc}-deploy`

- **일자**: 2026-04-27
- **상태**: accepted
- **결정**: 서비스별로 **이미지 ECR(`changshin-{svc}`)** 과 **Flux OCI artifact ECR(`changshin-{svc}-deploy`)** 을 분리해서 두 개씩 운영.
- **맥락**: 사내 `weplanet-starter-nestjs` 보일러 CI(`ci/job-nestjs.gitlab-ci.yml` · `ci/job-flux.gitlab-ci.yml`)가 두 ECR 입력값을 분리해서 받는 구조. taabshop도 동일하게 사용 중이나 iac에는 미선언 상태로 수동 생성한 IaC drift가 발견됨. changshin은 처음부터 iac에 등록.
- **대안**:
  - **단일 ECR + 태그 분리** — `:image-development`, `:flux-development` 등으로 구분. 보일러 CI 스크립트 리팩터링 필요. 이미지 태그(`:development`)와 artifact 태그(`:development`) 충돌 회피 위한 작업량 큼
  - **단일 공용 deploy ECR**(예: `changshin-deploy`) — 모든 서비스 artifact 한 ECR에 push, 태그로 구분. CI 일부 변경 필요
- **근거**:
  - 보일러 CI 그대로 채택 → 변경 비용 0
  - 이미지 retention과 artifact retention을 **독립 lifecycle 정책**으로 관리 가능
  - Lens/console에서 서비스별 분리되어 가시성 좋음
- **영향**:
  - `changshin-iac/aws/env/base.yml`의 `ecr.repositories`에 `*-deploy` 4개(`auth-deploy`, `ax1-deploy`, `ax2-deploy`, `ax3-deploy`) 추가
  - terraform이 lifecycle policy(미태그 이미지 1일 후 만료, dev/prod 태그 최근 20개 보존)도 자동 생성
  - 각 앱 리포 `.gitlab-ci.yml`의 `ecr_deploy: changshin-{svc}-deploy` 입력값 사용

---

## D-017 K8s 네임스페이스 = `changshin` (프로젝트명, 환경 무관)

- **일자**: 2026-04-27
- **상태**: accepted
- **결정**: 모든 changshin 앱 워크로드(`auth`, `ax1`, `ax2`, `ax3`)를 **`changshin` 단일 namespace**에 배치한다. namespace는 **프로젝트 식별자**로만 사용하고, 환경(dev/stg/prod)은 클러스터 자체로 구분(`changshin-dev` 클러스터 = dev, 향후 `changshin-prod` 클러스터 = prod).
- **맥락**: 보일러 매니페스트의 namespace 기본값이 `dev`로 들어 있어 환경명과 혼동 소지가 컸음. taabshop 컨벤션은 namespace=프로젝트명(`taabshop`). 둘 중 어느 쪽을 따를지 결정 필요했음.
- **대안**:
  - **namespace=환경명**(`dev`, `stg`, `prod`) — D-012(단일 AWS 계정 + 환경별 클러스터/VPC)와 의미 중복. 클러스터 자체가 환경 식별이라 namespace까지 환경 식별로 두는 건 redundant
  - **서비스별 namespace**(`auth`, `ax1`, ...) — 단일 프로젝트 내부에서 namespace 분리는 운영 부담 대비 격리 이득 작음
- **근거**:
  - 클러스터 단위로 환경이 이미 격리됨 → namespace까지 환경에 묶을 필요 없음
  - taabshop·기타 사내 프로젝트와 일관된 컨벤션
  - EKS Pod Identity association이 `namespace + service_account` 키 기반이라 namespace를 프로젝트로 두면 정합성 높음
- **영향**:
  - 각 앱 리포의 `k8s/clusters/dev/{svc}/kustomization.yaml` namespace 값을 `changshin`으로
  - `changshin-iac/k8s/clusters/dev/namespace.yaml`이 `changshin` namespace 생성 (기존 `dev` namespace는 prune됨)
  - `changshin-iac/aws/env/dev/config.yml`의 `eks.services[*].namespace = changshin`
  - 향후 stg/prod 클러스터 추가 시에도 namespace는 `changshin` 유지

---

## D-018 JWT 서명키 저장 = 수동 관리 K8s Secret

- **일자**: 2026-04-27
- **상태**: accepted
- **결정**: Auth 서비스의 JWT RSA 키 페어는 SSM Parameter Store/Secrets Manager 경유 없이 **수동 생성된 K8s Secret(`auth-jwt`)에 직접 주입**한다. 로컬 개발은 `apps/auth/.env`에 PEM을 직접 박는다.
- **맥락**: 보일러는 RSA 키를 SSM Parameter Store에 두고 `ExternalSecret` operator로 K8s Secret을 자동 생성하는 구조. 사용자 측 컨벤션 정립 시 단순화 요청 — env 직접 주입 방식 채택.
- **대안**:
  - **SSM Parameter Store + ExternalSecret** (보일러 기본): DB·기타 secret과 별도 store. 추가 IAM 부담
  - **Secrets Manager JSON + ExternalSecret** (DB credentials 패턴 통일): 회전 자동화 미사용 시 과설계
- **근거**:
  - JWT 키 회전은 토큰 invalidate 정책(version bump 등)을 코드에서 같이 처리해야 해서 **AWS-managed 회전이 의미 없음**
  - 키 회전 빈도 낮음 → 수동 절차로 충분
  - 단순한 K8s Secret이 인프라 복잡도 낮춤
- **영향**:
  - `apps/auth/.env`에 `RSA_PRIVATE_KEY="..."`, `RSA_PUBLIC_KEY="..."` 직접 입력 (multi-line PEM)
  - 클러스터에는 1회 수동 생성: `kubectl create secret generic auth-jwt -n changshin --from-file=RSA_PRIVATE_KEY=... --from-file=RSA_PUBLIC_KEY=...`
  - `deploy.yaml`이 `auth-jwt` K8s Secret을 `secretKeyRef`로 참조
  - Flux가 이 Secret을 관리하지 않으므로 prune 대상 아님 (안전)
  - **키 회전 시 절차**: 새 키 페어 생성 → `.env` 갱신 → K8s Secret 재생성(`kubectl create secret ... --dry-run=client -o yaml | kubectl apply -f -`) → 모든 사용자 토큰 version bump

---

## D-019 단일 통합 API 서비스로 변경

- **일자**: 2026-04-28
- **상태**: accepted (D-001 · D-007 · D-016 일부 supersede)
- **결정**: 4개의 분리 서비스(Auth + AX1/2/3) 구조를 폐기하고 **단일 통합 API 서비스**로 변경한다. 리포지토리는 `changshin/changshin-api` 1개, K8s에 배포되는 워크로드도 1개.
- **맥락**: 4서버 분리 구조로 초기 인프라(EKS · ECR · Pod Identity · Ingress 등)를 구축하던 중, 개발팀 내부 논의에서 **운영·개발 부담 대비 분리 이득이 크지 않다**는 결론에 도달. 단일 사용자 풀, 공유 DB(D-002), 동기 REST(D-010), path 기반 라우팅(D-011)을 이미 채택한 상태에서 분리 유지의 실익이 불분명했음.
- **대안**:
  - **기존 4서버 유지** ([[#D-001]]) — 운영 복잡도·CI 4벌 유지·디버깅 분산 비용 발생
  - **부분 분리** (Auth만 분리, AX1/2/3 통합) — 인증 게이트웨이 독립의 이득은 있으나 호출 추가 hop·세션/토큰 관리 복잡
- **근거**:
  - 사용자/도메인 경계가 분리 운영을 정당화할 만큼 명확하지 않음
  - 공유 DB (D-002) 위에서 4 서비스로 쪼갠 마이크로서비스는 **분산 모놀리식**에 가까웠음
  - 단일 NestJS monorepo에서도 모듈/디렉터리로 도메인 분리는 가능 (modular monolith)
  - Azure 이전(D-003) 시 단일 워크로드가 복제·이전 부담이 더 적음
  - K8s 1개 Deployment · 1개 Ingress · 1개 ECR로 운영 비용 대폭 감소
- **영향**:
  - **저장소**: `changshin/changshin-api` 1개로 통합. 기존 `changshin-auth-api`, `changshin-ax2-api`, `changshin-ax3-api`는 `changshin-legacy/`로 이전 (코드 자산 보존, 더 이상 빌드/배포 안 됨)
  - **앱 구조**: `changshin-api/apps/ax-api/` (NestJS) + `apps/batch/` (cron/배치). 도메인은 NestJS 모듈로 분리 (예: `controllers/auth/`, `controllers/ax1/`, `controllers/ax2/`, `controllers/ax3/`)
  - **K8s**: namespace=`changshin` 유지(D-017). Deployment 1개(`ax-api`), Ingress 1개(`/api/*` 또는 path 그대로). ECR `changshin-ax`(또는 기존 명) 1개 + `changshin-ax-deploy` 1개
  - **인증**: D-007 `aud` 기반 분기는 서비스 분기 목적으로는 의미 약화. JWT 자체는 유지(D-009). `aud`는 **제거하거나 클라이언트 컨텍스트(웹/모바일/외부 ERP) 구분용으로 재정의** — 후속 결정 필요
  - **DB 스키마**: D-002의 논리 분리 원칙은 유지 (단일 서비스가 여러 스키마에 접근). `auth.*`, `ax1.*`, `ax2.*`, `ax3.*`, `common.*` 분리 유지하되 단일 DB 계정/연결로 운영
  - **도메인 경계 보존**: AX-2 등 도메인별 모듈 폴더 구조를 명확히 유지해, 향후 다시 분리하고 싶을 때 비용을 낮춘다
  - **Ingress 라우팅**: D-011의 path 기반 그대로 (단일 backend로 모두 라우팅) 또는 단일 `/api/*` prefix로 단순화 — 후속 결정
  - **ECR**: D-016의 `{svc}` + `{svc}-deploy` 패턴이 4세트 → 1세트로 축소 (또는 기존 4세트 중 1세트만 유지)
  - **비용 절감**: Pod Identity association · IAM Role · Secret 등 4벌 → 1벌

---

## D-020 도메인 패키지 트랙별 디렉토리 분리 (ax / ax2 / ax3 / shared)

- **일자**: 2026-04-28
- **상태**: accepted (D-019 후속 — "도메인 모듈 디렉터리 컨벤션")
- **결정**: `packages/data/domain/src/` 평탄 구조를 폐기하고 **트랙별 sub-barrel 디렉토리**로 재배치한다.
  - `ax/` — AX-1 (마켓센싱) 도메인
  - `ax2/` — AX-2 (스케줄링) 도메인 (현재 비어 있음)
  - `ax3/` — AX-3 (구매정산) 도메인 (현재 비어 있음)
  - `shared/` — 트랙 간 공유 마스터 (`sku/` 등)
  - 최상위 `index.ts` 는 4개 트랙 sub-barrel 을 re-export
- **맥락**: D-019 로 단일 서비스 통합 후, AX-1/AX-2/AX-3 트랙을 동시에 작업하는 개발자가 늘어남. 평탄 구조에서는 신규 도메인 추가마다 동일 디렉토리·동일 `index.ts` 를 동시에 수정하게 돼 **머지 충돌 빈발 예상**. 트랙 경계가 코드 디렉토리 수준에서 드러나도록 분리 필요.
- **대안**:
  - **별도 패키지 분할** (`@data/domain-ax`, `@data/domain-ax2` 등) — 격리도는 강하지만 공유 엔티티(예: User, Vendor, SKU 등)가 `shared` 패키지로 빠져야 하고 cross-track relation 시 도로 얽힘. 빌드 그래프·tsconfig path 재구성 비용도 큼.
  - **CODEOWNERS + 파일 prefix 컨벤션 유지** — 격리도 약함, barrel 충돌 그대로
- **근거**:
  - 트랙별 sub-barrel(`ax/index.ts`, `ax2/index.ts`, `ax3/index.ts`, `shared/index.ts`) 로 신규 도메인 추가 시 다른 트랙 파일을 건드리지 않음 → 머지 충돌 격리
  - 한 패키지(`@data/domain`) 내부에서 cross-track import 가 자연스럽게 가능 (별도 패키지 분할 시의 순환 의존 문제 회피)
  - tsconfig path · 빌드 entry · barrel re-export 만 손대면 되는 가장 가벼운 변경
  - 향후 트랙 도메인이 진짜 독립적임이 확정되면 별도 패키지로 승격하는 비용 낮춤
- **영향**:
  - **저장소**: 기존 17개 도메인 디렉토리는 `ax/` 하위로 `git mv` (rename 73건). 트랙 prefix 가 `CSA/CSB/CSC...` 인 마켓센싱 자재코드 기준 트랙 분류 일치
  - **deep import**: `@app/data/domain/{name}` 형태 deep import 7건을 `@app/data/domain/ax/{name}` 로 갱신. tsconfig path `@app/data/domain/*` → `src/*` 매핑은 그대로 동작
  - **컨벤션 문서**: `.claude/rules/coding-conventions.md` 의 "Service는 domain에" 섹션과 금지 패턴(`packages/data/domain/src/` 바로 아래 도메인 직접 두기 금지) 갱신
  - **shared 디렉토리 도입**: 트랙 간 공유 마스터 도메인의 자리. 첫 입주는 SKU 마스터 (D-021)
  - **상위 barrel**: `src/index.ts` 가 단순한 `export * from './ax' / './ax2' / './ax3' / './shared'` 형태로 정리됨

---

## D-021 SKU 마스터 도메인 설계 (창신 제품 클렌징 기반)

- **일자**: 2026-04-28
- **상태**: accepted
- **결정**: 창신 제품 List(330건) 클렌징 작업의 기반으로 **5개 엔티티 + 1 enum** 으로 SKU 마스터를 구성하고 `packages/data/domain/src/shared/sku/` 에 배치한다.
  - **`Sku`**: 마스터 본체. PK 는 `id` (auto-increment) + UNIQUE(`materialCode`, `productCode`)
  - **`SkuSeries`**: 시리즈 마스터 (A, 장식A, B, C-3, ... V — 31종, 인적 입력 추가 대비 마스터화)
  - **`SkuComponentType`**: 부품 타입 마스터 (오버캡, 캡, 내캡, ... 주걱 — 31종, 부품 추가 시 enum 변경 회피)
  - **`SkuChamber`**: 챔버 자식 (단일/다중 챔버 모두 표현, `chamberIndex`/`capacityMl`/`baseMaterial`/`containerMaterial`)
  - **`SkuComponent`**: 부품-소재 매핑 junction (`material` 컬럼, `chamberIndex` 로 챔버별 부품 구분)
  - **`SkuContainerType` enum**: 12종 (Pump Bottle, Open Jar, Double Chamber Bottle 등 CSV 의 "최종 Mapping" 카테고리)
- **맥락**: 창신 제품 카탈로그 CSV 분석 결과 (1) 자재코드 1건이 소재 변형(PETG/PP)으로 1:N 으로 분기하는 8건, (2) 30+ 부품 컬럼이 sparse 한 행렬, (3) Double Chamber Bottle 의 챔버별 용량/소재 분리 필요 같은 모델링 결정사항이 다수 도출됨. 단순 1:1 매핑으로 import 하면 클렌징/조회/확장에 모두 부담.
- **대안**:
  - **자재코드를 string PK 로 사용** — 동일 자재코드 변형 처리에 `parentSkuCode` self-FK 또는 코드 합성 룰 필요. 프로젝트 컨벤션(다른 엔티티들이 `id` PK 사용)과도 어긋남
  - **30+ 부품 컬럼 인라인** — sparse 데이터 낭비, 신규 부품 추가 시 ALTER, 소재 통일 같은 클렌징 작업이 30 컬럼 모두 UPDATE
  - **부품 타입을 enum 으로 고정** — 인적 입력으로 부품이 추가될 때마다 enum 값 추가 + 마이그레이션 필요
  - **챔버 정보 컬럼 대칭 분리** (capacityMlPrimary/Secondary 등) — Double Chamber 만 위해 컬럼 6개 추가, 3챔버 등장 시 또 추가
  - **시리즈를 string 컬럼으로** — 표기 흔들림(`A` vs ` A`) 방지 못함, 부가 속성(설명·정렬) 붙이기 어려움
- **근거**:
  - **surrogate PK + natural unique** 가 프로젝트 컨벤션과 맞고 자식 테이블(SkuChamber/SkuComponent) FK 가 깔끔
  - **부품 정규화 (`SkuComponent`)**: 클렌징 작업 핵심인 "소재 표기 통일", "이상치 제거", "부품×소재 빈도 분석" 모두 한 컬럼 단위로 처리 가능
  - **`SkuComponentType` 마스터화**: 인적 입력 (사람이 카탈로그 추가) 컨텍스트라 부품 종류는 계속 추가될 가능성 높음. enum 보다 row 추가가 가볍고 부가 속성(설명/순서) 붙이기 쉬움
  - **`SkuChamber` 자식 테이블**: 단일 챔버는 1 row, 이중 챔버는 2 row, 3챔버 등장도 row 추가만으로 흡수. Double Chamber Bottle enum 이 정식 카테고리이므로 데이터 모델도 "챔버를 1급 시민" 으로 다루는 게 일관됨
  - **`SkuSeries` 마스터화**: 인적 입력으로 시리즈 추가 예정, 표기 통제 + 향후 부가 속성(설명·이미지·정렬) 자리 마련
- **영향**:
  - **위치**: `packages/data/domain/src/shared/sku/` (D-020 의 `shared/` 첫 입주). 엔티티 5개는 `entities/` 하위로 묶음
  - **컬럼 정규화 룰**: import 시 `productCode` 공백 제거 (`CSC60-DU - PETG` → `CSC60-DU-PETG`), `0` 값은 NULL, 슬래시 들어간 부품 셀(`PETG/PP`)은 다중 챔버에서 `chamberIndex` 별로 row 분리
  - **이미지 필드**: `Sku.imageUrl` (S3 object key 저장, 응답 시 presigned URL 변환). PDF 카탈로그에서 코워크가 추출 후 업데이트 예정
  - **클렌징 보존**: `소재 Sanitized` 의 `PMMA (아크릴)` 표기는 의도적 차이라 그대로 유지 (다른 컬럼은 `PMMA`). `장식A` 같은 시리즈 표기 흔들림도 그대로 유지
  - **import 스크립트**: `apps/ax-api/scripts/sku-import.ts` (1회성, 재현성 위해 커밋). 환경변수로 DB 연결 주입
  - **데이터 검토 후속 항목**: CSF60-DU / CSF100-DU 가 `containerType=doubleChamberBottle` 인데 CSV 표기는 단일 챔버 — PDF 카탈로그 검토 후 결정 필요 (미결정 항목으로 추가)

---

## D-022 SKU 이미지 S3 업로드 1차 완료 (보일러 컨벤션 적용)

- **일자**: 2026-04-29
- **상태**: accepted
- **결정**: SKU 마스터의 제품 이미지를 dev 버킷 `weplanet-files` 의 `images/skus/<uuid>.png` key 로 업로드하고, DB `Sku.imageUrl` 컬럼에는 **풀 CloudFront URL** (`https://dev-file.weplanet.co.kr/images/skus/<uuid>.png`) 을 저장한다. legacy 보일러 `files.controller` 패턴(`${type}s/${kind}` prefix + `crypto.randomUUID().${ext}` key + cloudfront 풀 URL 응답) 과 동일.
- **맥락**: 창신 제품 카탈로그 PDF 에서 추출한 289 개 SKU 이미지를 S3 + DB 에 1차 반영. 처음에는 `sku/<materialCode>.png` 로 적재했으나 보일러 컨벤션과 어긋나 cleanup 후 재실행. 이미지-자재코드 매칭은 이전 세션에서 완료(매칭률 88.6%, 39개 미매칭은 변형 코드/CSV 중복으로 후속 정규화 대상).
- **대안**:
  - **`sku/<materialCode>.png` (1차 시도, 폐기)** — 사람이 읽기 쉽지만 위플래닛 보일러(`files.controller` + 다른 프로젝트 sonova 등) 컨벤션과 어긋남. FE 가 `imageUrl` 을 그대로 사용 못하고 변환 단계가 필요해짐
  - **path-only 저장 (`images/skus/<uuid>.png`)** — 응답 시 cloudfront prefix 부착 필요. legacy 의 `files.controller` 가 클라에 풀 URL 반환 → 클라가 그대로 저장하는 흐름이라 풀 URL 저장이 더 일관됨
  - **수동 콘솔 업로드 + 직접 SQL** — 재현성 0
- **근거**:
  - **legacy `files.controller`**: `key = \`${crypto.randomUUID()}.${extensions[0]}\`` 로 temp 업로드 → `copyObject({ prefix: '${type}s/${kind}' })` 로 영구 버킷 이동 → 응답 `url: \`${cloudfront}/${path}\``. 이 흐름이 위플래닛 표준
  - 다른 위플래닛 프로젝트(sonova) 도 imageUrl 에 풀 cloudfront URL 저장 (예: `https://dev-file.sonovashop.co.kr/images/etc/<uuid>.jpg`)
  - 같은 materialCode 의 productCode 변형은 **같은 uuid · 같은 imageUrl 공유** (소재만 다른 동일 형상이라 이미지 1개로 충분)
- **영향**:
  - **스크립트**: `apps/ax-api/scripts/sku-image-upload.ts` 자산화. `pnpm --filter ax-api sku:image-upload <dir> [--dry-run] [--force]`. 기본은 `imageUrl` 비어있는 SKU 만 처리(재실행 안전), `--force` 로 새 uuid 덮어쓰기 가능
  - **버킷·prefix**: dev `s3://weplanet-files/images/skus/<uuid>.png`, ContentType=`image/png`
  - **DB 저장 형식**: `https://dev-file.weplanet.co.kr/images/skus/<uuid>.png` (풀 URL)
  - **실행 결과 (재시도, 2026-04-29)**: 이미지 파일 289 / S3 업로드 287 / S3 실패 0 / DB UPDATE rows 289 / distinct URL 287 (변형 productCode 2쌍 — `CSC100-DU`, `CSC60-DU` — 가 같은 URL 공유) / DB 매칭 실패 2 (`CSCY30-CPP_AL`, `CSCY50-UPP_AL`)
  - **1차 잘못된 적재 cleanup**: `s3://weplanet-files/sku/` 287 객체 삭제 + DB 289 rows `imageUrl=NULL` 되돌림 후 재실행
  - **후속**: prod 반영은 별도 결정. 미매칭 자재코드 39 + 2 정리 후 2차 실행. `Sku.imageUrl` 컬럼 코멘트(현재 "S3 object key 저장")는 풀 URL 저장으로 갱신 필요

---

## (템플릿) D-### 제목

- **일자**:
- **상태**: proposed
- **결정**:
- **맥락**:
- **대안**:
- **근거**:
- **영향**:

---

## 미결정 · 논의 필요

> [!tip] 단일 보드로 이전됨
> 미결정·논의 필요 항목은 [[00 - Action Board]] 에서 단일 소스로 관리. 새 결정으로 닫히면 위 `D-###` 섹션에 추가하고 Action Board 의 ✅ 로 옮긴다.

**현재 미결정 (요약)** — 자세한 맥락·우선순위는 [[00 - Action Board]] 참조:

- D-019 후속: `aud` 클레임 처리 / Ingress path 단순화 / ECR 통폐합 → [[00 - Action Board#A. changshin-api 통합 마무리 (D-019 후속)]]
- 클라이언트 도메인 확정 시 인증서 + host 교체 → [[00 - Action Board#D-2. 인프라 · 도메인 (임시 결정, 영구화 필요)]]
- 80 포트 listener 미동작 진단 → [[00 - Action Board#📥 백로그]]
- D-021 후속: CSF60-DU · CSF100-DU 챔버 구조 / 시리즈 표기 흔들림 → [[00 - Action Board#🧹 데이터 클렌징 후속]]
- D-019 후속 결정 완료: 도메인 모듈 디렉터리 컨벤션 → [[#D-020]] 로 결정 (트랙별 sub-barrel)

---

> 처음으로: [[00 - Infrastructure (Index)]]
