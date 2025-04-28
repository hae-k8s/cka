# Design and Install Kubernetes Cluster

## 운영 방식 선택

- 목적
  - 학습
    - Minikube
    - AWS/GCP 싱글 노드
  - 개발 & 테스트
    - 멀티 노드
  - 운영
    - High Availability 멀티 노드
      - 마스터 노드는 taint 추가해서 운영 역할만 하도록
- 클라우드 혹은 온프레미스
- Workloads
  - 종류
  - 필요한 리소스
  - 예상 트래픽

## Choosing Kubernetes Infrastructure

Turnkey Solution

- 사용자가 직접 vm 관리
- Openshift, VMWare Cloud PKS, ...
  Hosted Solution
- GKE, EKS, Openshift Online, AKS, ...

## High Availability

- 마스터 노드가 하나라면 위험함
- 여러개의 마스터 노드를 두어야 함
- 로드 밸런서를 앞단에 두어 분산하면 가능
- 동시에 모두 사용되는 것이 아니라 Active-Standby 모드도 작동함
- Leader Election 과정을 통해 이뤄짐

### External ETCD Server

- 안정성이 올라감
- 설계가 더 어려워지고, 서버 개수도 두배로 필요함
- `kube-apiserver.service` 에서 `etcd-servers` 옵션으로 경로 지정 가능

## ETCD HA

- 여러 노드가 있을 때 동일한 key-value를 가져야 하기 때문에 복제 작업이 일어남
- 리더에 쓰기 작업이 발생하고 이게 퍼져나가는 형태
- 3개 이상의 홀수 노드로 설정하는걸 권장함
