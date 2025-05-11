## Design a Kubernetes Cluster

- 목적에 따라 클러스터 구성이 달라짐
- Education : Minikube, Single Node
- Development & Testing : Multi Node
- Production : High Availability Multi Node
- 마스터 노드는 다른 컴포넌트의 Control 기능만 하도록 하는 것이 권장됨
- kubeadm 같은 배포 도구는 Master Node에 Taint를 추가하여 호스팅 방지

## Choosing Kubernetes Infrastructure

- Turnkey Solutions : VM을 사용자가 직접 관리
- Openshift, Cloud Foundry Container Runtime, VMware Cloud PKS, Vagrant
- Hosted Solutions (Managed Solutions) : Kubernetes-As-A-Service 형태로 VM을 제공
- Google Kubernetes Engine (GKE), Openshift Online, Azure Kubernetes Service (AKS), Amazon Elastic Kubernetes Service (EKS)

## Configure High Availability

- Master Node가 다운되면 kube-apiserver 기능 및 pod 관리가 불가능
- 단일 실패 지점을 없애고 고가용성을 보장하기 위해 마스터 노드를 여러 개 두어야 함
- `kube-apiserver` 앞단에 Load Balancer를 두어 부하 분산
- `kube-control-manager`는 `Active-Standby` 형태로 선출된 Leader가 작업을 진행
- `Stacked ETCD`는 Master Node가 다운되면 Control Plane Instance가 유실되는 문제 발생
- `External ETCD`는 더 안전하지만 설정이 어려움

## ETCD in HA (High Availability)

- `Distributed ETCD` : Multi ETCD 구조에서 데이터를 복제하여 고가용성을 보장
- READ는 동시에 일어나지만 WRITE는 Leader ETCD에서 먼저 발생하고, Leader가 Follower에 복제본을 씀
- `Leader Election - RAFT` : 무작위 타이머가 작동하여 Leader 투표가 발생
- `Quorum = N / 2 + 1` : WRITE 응답에 대해 정족수(다수의 노드)를 만족하면 WRITE가 성공했다고 간주
- 최소 3개의 홀수 인스턴스를 구성하는 것이 권장됨
- ETCD API 버전은 v3을 사용해야 함 : `export ETCDCTL_API=3`
