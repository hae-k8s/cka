# Design and Install A Kubenetes Cluster

## Choosing Kubernetes Infrastructure

- **Minikube**: 단일 노드 클러스터, VirtualBox 등 가상화 소프트웨어 사용
- **Kubeadm**: 단일/다중 노드 지원, 사전에 VM 구성 필요

### TurnKey Solution

> "턴키(turnkey)"는 영어로 "**열쇠를 돌리면 작동한다**"는 뜻으로, 건설이나 프로젝트 분야에서 사용되는 용어입니다. 이는 프로젝트를 맡기면 공급자가 모든 과정을 책임지고 완성한 후 고객에게 인도하는 방식
> 
- VM 유지/패치/업그레이드 → 사용자 책임
- OpenShift (Red Hat)
- Cloud Foundry Container Runtime
- VMware Cloud PKS
- Vagrant 스크립트

### Hosted/Managed Solution

- SaaS 형태의 쿠버네티스
- VM 및 클러스터 구성/관리가 제공자 담당
- Google Container Engine
- OpenShift Online
- Azure Kubernetes Service (AKS)
- Amazon Elastic Container Service for Kubernetes

## Configure High Availability

- 마스터 노드 장애 상황
    - 마스터 노드 손실 시에도 워커 노드의 컨테이너는 계속 실행됨
    - 하지만 새로운 문제 발생 시(예: 파드 충돌) 컨트롤러가 없어 복구 불가능
    - kube-apiserver가 없어 외부에서 클러스터 관리 불가
- SPOF를 피하기 위해 여러 마스터 노드 필요
    - 모든 구성요소(마스터 노드, 워커 노드, 컨트롤 플레인)의 중복성 확보

### 마스터 노드 구성 방식

1. **API Server**:
    - Active-Active
    - 로드 밸런서(nginx, HA proxy 등)를 통해 요청 분산
2. **Scheduler, Controller Manager**
    - Active-Standby
    - leader election 과정을 통해 하나만 활성화
    - 리더 선출 옵션:
        - 기본적으로 15초 동안 리스 유지
        - 10초마다 리스 갱신
        - 2초마다 리더가 되려고 시도
3. **ETCD**:
    1. 스택형 컨트롤 플레인 (마스터 노드와 같이 배치)
    2. 외부 etcd 서버 구성 (별도 서버에 배치)

## ETCD in HA

- etcd는 분산형, 신뢰할 수 있는 키-값 저장소
- 테이블 구조 대신 문서/페이지 형태로 정보 저장
- 복잡한 데이터는 JSON이나 YAML 형식으로 처리
- 리더 선출과 쓰기 처리
    - 모든 노드에서 읽기는 쉽게 가능
    - 쓰기는 리더 노드만 처리
    - 쓰기 요청이 팔로워 노드로 오면 내부적으로 리더에게 전달
    - 리더는 다른 노드들에게 데이터 복사본 배포

### Raft Protocol

- 분산 합의 알고리즘으로 리더 선출
- 무작위 타이머로 선출 프로세스 시작
- 리더는 주기적으로 다른 노드에게 알림 전송
- 리더와 연결이 끊기면 재선출 과정 시작

### Quorum

- 클러스터가 제대로 작동하기 위한 최소 노드 수
- 계산법: (전체 노드 수 ÷ 2) + 1
- 최소 3개 노드 권장 (1개 노드 장애 허용)
- 홀수 개의 노드가 권장됨 (3, 5, 7)
- 짝수 노드는 네트워크 분할 시 문제 발생 가능
- 5개 노드면 충분한 **Fault tolerance** 제공 (2개 노드 장애 허용)