## Introduction to Deployment with Kubeadm

- `kubeadm`은 Kubernetes 모범 사례를 이용해 다중 클러스터 설정을 도와줌
- 1. Master Node 지정
- 2. Container Runtime (containerd) 설치
- 3. kubeadm 설치
- 4. Master Node 초기화
- 5. POD Network 설정
- 6. Node Join

## Demo - Deployment with Kubeadm

- 컨테이너 메모리 제한 설정 시 cgroup 설정 필요 (v1.22 이후 기본은 `systemd`)
- VM과 설정을 동일하게 맞추어야 함 => 현재 사용 중인 cgroup 확인 : `ps -p 1`
- `--apiserver-advertise-address` : LB나 외부 통신에 사용될 IP 지정
- `--pod-network-cidr` : 해당 CIDR 규칙으로 Pod IP 생성
