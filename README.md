ArgoCD Architecture
=
<p align="center">
  <img src="https://github.com/Yess-P/YesS.github.io/assets/79623220/97d00059-750d-4e93-a409-c0c21a87cdfe" alt="text" width="number" />
  https://github.com/argoproj/argo-cd/blob/master/docs/developer-guide/architecture/components.md
</p>




# Layer
- **UI**: 표현 계층. 사용자는 주로 이 계층의 구성 요소를 통해 Argo CD와 상호 작용
- **Application**: UI 계층에서 구성 요소를 지원하는 데 필요한 기능
- **Core**: 주요 ArgoCD gitops 기능은 Core 계층의 Kubernetes 컨트롤러와 구성 요소에 의해 구현됨
- **Infra**: ArgoCD가 인프라의 일부로써 의존하는 도구   
<br>
   
# Component
<br>

## UI

**Webapp**

- Kubernetes 클러스터에 배포된 애플리케이션을 관리할 수 있는 웹 인터페이스 제공

**CLI**

- 사용자가 ArgoCD API와 상호 작용하는 데 사용할 수 있는 CLI를 제공

<br>

## Application

**API Server**

- UI, CLI, CI/CD 시스템에서 사용되는 API를 호출하기 위한 gRPC/REST 서버
- 다음과 같은 역할을 담당
    - Application 관리 및 상태 보고
    - Application 작업 호출( 예: 동기화, 롤백, 사용자 정의 작업)
    - repository와 cluster 자격 증명 관리(kubernetes secret에 저장)
    - 외부 ID 공급자에 대한 인증 및 인증 위임
    - RBAC 적용
    - Git webhook 이벤트를 위한 listener/forwarder

<br>

## Core

**Application Controller**

- Application을 지속적으로 모니터링하는 Kubernetes 컨트롤러
- 모니터링을 통해 현재 배포되어 있는 형상(live state)과 목표 형상(desired target state)을 비교함
- 동기화되지 않은(OutOfSync) application 상태를 감지
- Lifecycle 이벤트를 활용한 user-defined hooks을 호출하는 역할
    - PreSync
    - Sync
    - PostSync
- ArgoCD 리소스중 유일하게 statefulset으로 구성

### ApplicationSet Controller

- ApplicationSet 컨트롤러는 ApplicationSet 리소스를 조정하는 역할
- Application Controller와 마찬가지로 ApplicationSet CRD를 지원하는 Kubernetes 컨트롤러
- 단일 repo를 활용하여 다수의 클러스터 배포시 application의 자동화와 유연성을 제공함 
(클러스터 관리자 관점에서의 시나리오를 지원하는 기능을 제공)
    - 단일 kubernetes manifest(applicationset)로 다수의 클러스터에 배포
    - 단일 kubernetes manifest(applicationset)로 다수의 Git repository(배포 파일)를 다수의 application 배포
    - 단일 Git repository(배포 파일)로 다수의 application 배포
    - 멀티 테넌트 클러스터 내에서 ArgoCD를 활용하여 application을 배포하는 개별 테넌트 기능 향상

### Repo Server

- Application manifest를 보관을 위해 Git repository의 로컬 cache를 유지하는 내부 서비스
- Git repository와 상호 작용하는 역할을 담당
- 아래의 input 값들을 kubernetes manifest로 변환
    - Repository URL
    - Revision (commit, tag, branch)
    - Application path
    - Template specific settings: parameters, helm values.yaml

<br>

## Infra

### Redis

- ArgoCD의 cache 저장소
- Kube API 혹은 Git provider를 사용하는 요청을 줄이거나 UI 동작 지원을 위해 사용됨
- redis는 버려지는 캐시로만 사용되며 손실될 수 있음


### Kube API

- Argo CD 컨트롤러는 reconciliation loop를 실행하기 위해 kubernetes API에 연결함
※ reconciliation loop - 특정 CR이 수정되는 이벤트를 관찰하고, 이벤트에 맞춰 코드를 실행하는 루프


### Git

- Gitops 도구로서 Argo CD는 원하는 상태의 Kubernetes 리소스가 Git 리포지토리에 제공되어야 함
- Git이라고 표현했지만 아래의 목록들을 지원함
    - Git repo
    - Helm repo
    - OCI artifact repo


### Dex

- ArgoCD의 외부 인증 서버 역할, 외부 OIDC 제공자와의 연동 지원
- 다른 앱과의 인증을 위해 OpenID Connect(OIDC)를 사용하는 identity 서비스
- LDAP, SAML, OAuth2 혹은 GitHub, Google과 같은 다양한 ID 공급자를 지원
- 2020년 6월 25일 CNCF에 sandbox 프로젝트로 등록됨
