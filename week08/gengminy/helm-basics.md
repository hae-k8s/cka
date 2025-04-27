## What is Helm

- `Helm`은 Kubernetes의 패키지 관리자
- `Helm`은 게임 설치자처럼 하나의 명령어로 모든 오브젝트를 Kubernetes에 추가해줌
- 각 앱에 대한 변경 사항을 추적하고 설치, 업데이트, 롤백, 삭제가 가능

## A Quick Note about Helm2 vs Helm3

- `Helm2` : Kubernetes에서 RBAC이나 Resource Control 기능이 부족해 `Tiller`라는 중개자가 필요했음
- `Tiller`는 sudo 모드로 동작하여 보안 이슈가 존재
- `Tiller`는 RBAC, Custom Resource Definition 기능 출시 이후 제거됨
- `Helm3`는 RBAC으로 동작
- `Helm2`는 Helm Chart를 비교하여 롤백 결정을 내림 : kubectl 등 외부에서 수정된 사항은 비교하지 않음
- `Helm3`는 되돌리려는 상태와 라이브 상태를 비교하여 외부 수정 사항도 비교 가능

## Helm Components

- `Chart` : 클러스터가 객체 생성을 위해 알아야 할 정의 파일의 모음
- `Release` : `Helm chart`를 이용하여 어플리케이션을 한 번 설치하는 것
- `Metadata` : `Release`, `Chart`, `Revision`을 추적하기 위해 저장된 데이터
- `Templating` : 다른 YAML 파일의 내용을 참조하는 것
- `Helm Hub`: `Artifact Hub`에서 원하는 `Helm Chart` 사용 가능

## Helm Charts

- `Templating` : 다른 YAML 파일의 내용을 참조하는 것
- `Template` : 다른 YAML 파일 내용을 참조하는 객체들
- `Helm Chart`를 사용하면 `Chart.yaml` 파일이 생성됨
- `apiVersion`: `v1 - Helm 2` | `v2 - Helm 3`
- `appVersion` : 배포되는 어플리케이션의 버전
- `version` : `Helm Chart`의 버전

## Working with Helm: Basics

- `helm search hub` : `artifacthub.io`를 기본으로 지정하여 검색
- `helm search repo` : 수동으로 지정된 `Helm Repository` 에서 검색

## Customizing Chart Parameters

- `Helm Chart` 설치 시 템플릿 값을 파라미터로 전달하는 방법
  - `helm install --set <parameter=value>`
  - `helm install --values <yaml-file>`
  - `helm pull --untar <chart>`로 로컬로 가져와 값 수정 후 `helm install <name> <dir>`

## Lifecycle Management with Helm

- 각 Release는 다른 Release와 리비전이 격리됨
- Release를 업그레이드하면 Helm이 모든 변경 사항을 추적함
- `helm history <release>` : 리비전 검색
- `helm rollback <release> <revision-number>` : 리비전 롤백, 새로운 롤백 리비전이 생성됨
- `helm rollback`은 설정 파일만 롤백하고, PVC, DB 같은 외부 데이터는 복원하지 않음
