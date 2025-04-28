# Helm Basics

## What is Helm

- 모든 컴포넌트를 yaml 파일로 관리하는건 힘듦
- Helm은 쿠버네티스를 위한 패키지 관리자임
- 어플리케이션 설치, 롤백, 업데이트, 삭제를 한번에 처리할 수 있게 함
- 앱을 오브젝트 집합이 아닌 앱 자체로 관리할 수 있게 함

## Helm 2 vs Helm 3

- Helm 2는 Tiller 중개자 존재
  - RBAC, CRD가 없었기 때문
  - 보안 위험성 존재(sudo 모드 실행)
  - 위 시스템들이 생기고 나서 Tiller 제거
- 롤백 시 Helm 2는 Helm을 통해 변경된 내역만 탐지, Helm 3은 실제로 작동중인 오브젝트의 변경 내역 탐지
  - kubectl을 통해 이미지를 변경할 경우 Helm 2는 탐지하지 못함

## Helm Components

- `chart`: 정의 파일 집합
- `release`: chart를 이용해 어플리케이션 설치
- `repository`: chart 저장소(hub)

## Helm Charts

- `Template`: 다른 파일의 정보를 가져와 사용할 수 있음 (liquid template같은 느낌)
- Chart.yaml 파일이 자동으로 생성됨
  - `apiVersion` 필드는 Helm2의 경우 v1, Helm3의 경우 v2
  - `dependencies` 필드를 통해 의존성있는 차트를 가져올 수 있음

## Working with Helm

- helm --help
- helm repo --help

## Customizing chart parameters

- heml install 후 Values.yaml 값을 오버라이딩하거나
  - `--set a=b`
  - `--values custom-values.yaml`
- helm pull -> Values.yaml 수정 후 install하는 방식

## Lifecycle Management wth Helm

- 각 Release 안에서 Revision으로 구분됨
- Helm이 변경사항 추적
- `helm history {release}` : 리비전 조회
- `helm rollback {release} {revision}` : 리비전 롤백
  - 설정 파일만 롤백
  - PVC, DB 같은 외부 데이터는 롤백하지 않음
