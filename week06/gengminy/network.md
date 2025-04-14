## Prerequisite – CNI

1. Network Namespace 생성
2. Bridge Network/Interface 생성
3. Veth Pair 생성 (`veth0` ↔ `veth1`)
4. Veth 한쪽을 Namespace에 연결
5. 다른 Veth 한쪽을 Bridge에 연결
6. IP 주소 할당
7. Interface 상태 확인 (`ip addr`, `ip link`)
8. IP 포워딩 및 NAT 설정 (`sysctl net.ipv4.ip_forward=1`, `iptables -t nat …`)

- `Bridge`: 컨테이너 간 통신을 위한 가상 네트워크
- 위 과정은 모든 Container Runtime이 Bridge 네트워크를 설정할 때 수행하는 기본 루틴
- CNI(Container Network Interface)는 컨테이너 런타임 환경에서 네트워킹을 표준화하기 위한 플러그인 인터페이스

## Cluster Networking

- 각 Node에는 최소 하나 이상의 Network Interface 필요
- Master Node 구성 요소(API Server, Scheduler, Controller Manager 등)는 각각 별도의 Listening Port 사용
- `ip address show type bridge`: 브리지 인터페이스 확인
- `netstat -npl | grep -i <pod-name>`: Pod 관련 프로세스의 Listening Port 확인
