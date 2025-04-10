## Certificates API

- 새 사용자의 인증서를 위해 CA 서버에서 서명이 필요함
- CA 서버는 Master Node의 Controller Manager 컴포넌트 내에 위치하며, CSR 승인(CSR-Approving) 및 서명(CSR-Signing)을 담당함
- `CertificateSigningRequest` 리소스는 YAML 객체로 생성하고, `kubectl get csr` 명령어로 확인 가능
- 요청 승인/거부: `kubectl certificate approve <csr-name>` / `kubectl certificate deny <csr-name>`
- 요청에서 인증서 내용인 `spec.request` 필드는 DER 형식 CSR을 Base64로 인코딩한 값임
- 참고: https://kubernetes.io/docs/tasks/tls/certificate-issue-client-csr/

## KubeConfig

- 매 요청 시 `--client-certificate`, `--client-key`, `--certificate-authority` 파일을 명시하는 것은 번거로움
- 이 설정들을 kubeconfig 파일에 명시하고, `--kubeconfig` 플래그 하나로 대체 가능
- 기본 kubeconfig 파일 경로는 `$HOME/.kube/config`이며, `--kubeconfig` 플래그로 다른 경로 지정 가능
- kubeconfig 파일의 주요 구성 요소는 `clusters`, `users`, `contexts`
- `clusters`: API 서버 엔드포인트 및 CA 인증서(`certificate-authority-data` 또는 `certificate-authority`) 정의
- `users`: 클라이언트 인증서(`client-certificate-data`) 및 키(`client-key-data`), 또는 토큰 정의
- `contexts`: `cluster`, `user`, (선택적으로) `namespace`를 그룹핑하여 사용 편의성 제공
- `kubectl config view` 명령어로 현재 설정 확인 가능

## API Groups

- Kubernetes API는 기능에 따라 다음과 같은 엔드포인트로 그룹화됨:
  - `/healthz`, `/metrics`, `/version`
  - `/api` (core API 그룹, 예: Pods, Services)
  - `/apis` (확장된 API 그룹, 예: apps, batch, networking.k8s.io)
  - `/logs`, `/exec`, `/proxy` 등 디버깅용 서브리소스
- `kube-proxy`(클러스터 네트워크 프록시)와 `kubectl proxy`(API 서버 프록시)는 별도 개념

## Authorization

- 인증(Authentication) 후, 권한 부여(Authorization) 모듈이 요청 권한을 결정
- 주요 모드:
  - **Node**: kubelet이 API 서버에 요청할 때 인증된 노드 권한 검증
  - **ABAC**: 속성 기반 접근 제어(파일에 정의된 정책에 따라 허용/거부)
  - **RBAC**: 역할(Role)과 역할 바인딩(RoleBinding) 기반 접근 제어
  - **Webhook**: 외부 서비스(e.g. Open Policy Agent)에 권한 검증 위임
- `kube-apiserver` 실행 시 `--authorization-mode=Node,RBAC,Webhook` 등으로 지정
- 여러 모드가 지정된 경우, 하나의 모듈이라도 명시적으로 거부(Deny)하면 즉시 거부, 최소 하나가 허용(Allow)하고 거부가 없을 때 최종 허용

## Role-Based Access Control

- **Role**: 네임스페이스 범위 내 리소스에 대한 권한 집합 정의
- **RoleBinding**: 특정 Role을 사용자, 그룹, 서비스 계정에 바인딩
- 권한 테스트: `kubectl auth can-i <verb> <resource> [--namespace=<ns>]`

## ClusterRoles 및 ClusterRoleBindings

- 네임스페이스 리소스(Namespaced): Pods, ReplicaSets, Jobs, Deployments, Services, Secrets, ConfigMaps, Roles, RoleBindings 등
- 클러스터 리소스(Cluster-scoped): Nodes, PersistentVolumes, ClusterRoles, ClusterRoleBindings, CertificateSigningRequests, Namespaces 등
- 리소스 목록 확인: `kubectl api-resources --namespaced=true` 또는 `--namespaced=false`
- 네임스페이스 권한은 Role/RoleBinding, 클러스터 권한은 ClusterRole/ClusterRoleBinding으로 부여

## Service Accounts

- **User Account**: 사람 사용자, 인증서 또는 토큰으로 인증
- **Service Account**: 파드 내 애플리케이션이 사용하는 계정 (예: Prometheus, Jenkins)
- 리소스 종류: `ServiceAccount`
- 각 네임스페이스에 `default` 서비스 계정이 자동 생성
- 서비스 계정 토큰 자동 마운트 기능은 v1.24부터 비활성화, 필요 시 `kubectl create token <sa-name>` 또는 TokenRequest API 사용
- 토큰 자동 마운트는 `automountServiceAccountToken` 필드로 제어 가능

## Image Security

- 이미지 참조 형식: `[registry/][namespace/]repository[:tag|@digest]`
- 별도 namespace 지정 없으면 Docker Hub의 `library` 네임스페이스로 간주 (예: `nginx` → `docker.io/library/nginx`)
- 별도 registry 지정 없으면 기본 registry는 `docker.io`
- 프라이빗 레지스트리 접근 시 `Secret`(type: kubernetes.io/dockerconfigjson) 생성 후 ServiceAccount에 바인딩 필요

## Docker 보안 사전 준비

- 컨테이너는 네임스페이스, cgroup 등 커널 격리 메커니즘으로 호스트와 분리됨 (기본적으로 별도의 PID 네임스페이스 사용)
- `--user` 플래그로 컨테이너 내 실행 유저 지정 가능
- 기본적으로 컨테이너 루트(root)는 호스트의 root 권한과 동일하므로, 사용자 네임스페이스(User Namespace)를 통해 매핑하거나 권한을 제한해야 함
- 기본 Linux Capabilities가 제한되며, 필요한 경우 `--cap-add`, `--cap-drop`로 조정 가능

## Security Context

- **PodSecurityContext** (`spec.securityContext`): 파드 전체에 적용
- **Container SecurityContext** (`spec.containers[].securityContext`): 개별 컨테이너에 적용
- 설정 예: 사용자 ID, 그룹 ID, 읽기 전용 루트 파일시스템, Linux Capabilities, SELinux, AppArmor 프로필 등

## Network Policy

- 기본적으로 모든 네트워크 트래픽 허용 (All Allow)
- `NetworkPolicy` 리소프로 Ingress/Egress 트래픽 제어 가능
- 네트워크 플러그인(예: Calico, Cilium)은 NetworkPolicy 지원 필요 (`Flannel` 기본 플러그인은 미지원)

## NetworkPolicy 작성

- `spec.podSelector`: 정책 대상 파드 선택
- `spec.ingress[].from`: 허용할 Ingress 소스 정의
  - `podSelector`와 `namespaceSelector`를 동일 항목에 함께 명시하면 AND, 별도 항목으로 나열하면 OR
  - `ipBlock.cidr`로 특정 IP 대역 허용
- `spec.egress[]`: Egress 트래픽 제어 설정

## Custom Resource Definition (CRD)

- `CustomResourceDefinition` 리소스를 통해 사용자 정의 API 그룹/버전/리소스 생성
- CRD 생성 후 Custom Resource를 정의하여 ETCD에 저장 및 관리 가능

## Custom Controller

- 컨트롤 플레인 외부에서 CRD로 생성된 리소스를 모니터링하고 실제 워크로드를 관리
- 일반적으로 Go 언어와 `client-go` 라이브러리로 작성

## Operator Framework

- CRD와 Custom Controller를 패키징하여 Operator로 배포
- Operator SDK, Operator Lifecycle Manager(OLM) 등을 활용해 설치 및 관리 가능
