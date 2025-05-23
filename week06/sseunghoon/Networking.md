# Networking

## Prerequisite - Switching Routing

- **네트워크란**: 두 컴퓨터가 통신하기 위해 스위치로 연결된 환경
- **스위치**: 같은 네트워크 내에서만 통신 가능하게 함
- **라우터**: 서로 다른 네트워크를 연결하는 장치
- **게이트웨이**: 다른 네트워크로 나가는 "문"의 역할

### 리눅스 네트워크 명령어

- `ip link`: 호스트의 네트워크 인터페이스 확인/수정
- `ip addr`: 인터페이스에 할당된 IP 주소 확인
- `ip addr add`: 인터페이스에 IP 주소 할당
- `route` 또는 `ip route`: 라우팅 테이블 확인
- `ip route add`: 라우팅 테이블에 항목 추가

### 라우팅 설정

- 다른 네트워크에 접근하려면 게이트웨이 설정 필요
- 기본 게이트웨이(default gateway): 알려지지 않은 모든 네트워크 요청을 처리
- `0.0.0.0`은 모든 IP 목적지를 의미

### 리눅스 호스트를 라우터로 설정하기

1. 두 네트워크에 연결된 호스트 필요
2. 각 호스트에 적절한 라우팅 테이블 항목 추가
3. 패킷 포워딩 활성화: `/proc/sys/net/ipv4/ip_forward` 값을 1로 설정
4. 재부팅 후에도 설정 유지하려면 `/etc/sysctl.conf` 파일 수정 필요
- 포인트
    - 네트워크 설정 변경은 재부팅 시 초기화됨
    - 영구적으로 설정하려면 `/etc/network/interfaces` 파일에 저장 필요
    - 리눅스에서는 기본적으로 보안을 위해 인터페이스 간 패킷 포워딩이 비활성화되어 있음

## Prerequisite - DNS

- 로컬 이름 해석 (/etc/hosts)
    - 작은 네트워크에서는 각 시스템의 `/etc/hosts` 파일에 항목을 추가하여 이름 해석 가능
    - 구문: `[IP주소] [호스트이름]` (예: `192.168.1.11 db`)
    - 호스트는 이 파일에 있는 정보를 신뢰하며 실제 호스트 이름을 확인하지 않음
    - 여러 이름을 같은 IP에 매핑할 수 있음
- DNS 서버 사용
    - 네트워크가 커지면 모든 항목을 중앙 DNS 서버로 이동하는 것이 효율적
    - DNS 서버 설정은 `/etc/resolv.conf` 파일에서 구성
    - 구문: `nameserver [DNS서버IP]` (예: `nameserver 192.168.1.100`)
- 이름 해석 순서
    - 기본적으로 시스템은 먼저 로컬 `/etc/hosts` 파일을 확인한 다음 DNS 서버에 질의함
    - 이 순서는 `/etc/nsswitch.conf` 파일에서 `hosts` 항목으로 제어됨
    - 기본값: `files dns` (먼저 로컬 파일, 그 다음 DNS)
- 외부 도메인 해석
    - 외부 사이트(예: facebook.com)에 접속하려면 외부 DNS 서버가 필요
    - 공용 DNS 서버 사용 가능 (예: Google의 8.8.8.8)
    - 조직 내부 DNS 서버가 외부 DNS로 알 수 없는 호스트 이름을 전달하도록 구성 가능
- 도메인 이름 구조
    - 도메인 이름은 점으로 구분된 계층 구조로 구성
        - 루트(.) → 최상위 도메인(.com, .org 등) → 도메인 이름 → 서브도메인
    - 예: `apps.google.com` (apps=서브도메인, google=도메인, .com=최상위 도메인)
- 도메인 검색
    - `/etc/resolv.conf`에 `search` 항목을 추가하여 호스트 이름에 도메인을 자동 추가 가능
    - 예: `search mycompany.com prod.mycompany.com`
    - `web`만 입력해도 `web.mycompany.com`으로 해석됨
- DNS 레코드 타입
    - A 레코드: 호스트 이름을 IPv4 주소에 매핑
    - AAAA 레코드: 호스트 이름을 IPv6 주소에 매핑
    - CNAME 레코드: 이름을 다른 이름에 매핑 (별칭)
- DNS 테스트 도구
    - `nslookup`: DNS 서버에 호스트 이름을 질의 (로컬 hosts 파일 무시)
    - `dig`: 더 자세한 DNS 정보를 반환하는 도구 (로컬 hosts 파일 무시)

## Prerequisite - Network Namespaces

- 네임스페이스는 마치 집 안의 여러 방처럼 호스트 시스템 내에서 분리된 공간을 제공함
- 각 네임스페이스(자식)는 자신의 공간만 볼 수 있고 다른 공간은 볼 수 없음
- 호스트(부모)는 모든 네임스페이스를 볼 수 있음
- 네트워크 네임스페이스
    - 컨테이너는 자체 네트워크 공간을 가짐 (인터페이스, 라우팅 테이블, ARP 테이블)
    - 호스트 네트워크와 완전히 분리됨
- 명령어
    - 생성: `ip netns add [이름]`
    - 조회: `ip netns`
    - 네임스페이스 내 명령 실행:
        
        `ip netns exec [이름] [명령어]` 또는 `ip -n [이름] [명령어]`
        

### 네임스페이스 간 연결 방법

1. **직접 연결 (두 네임스페이스만)**
    - 가상 이더넷 페어(virtual ethernet pair) 생성
    - 각 인터페이스를 각 네임스페이스에 연결
    - IP 주소 할당 후 인터페이스 활성화
2. **다중 네임스페이스 연결**
    - 가상 스위치(Linux bridge) 생성
    - 각 네임스페이스를 가상 스위치에 연결
    - 각 인터페이스에 IP 주소 할당

### 외부 네트워크 연결

1. **호스트-네임스페이스 연결**
    - 브릿지 인터페이스에 IP 주소 할당
2. **외부 네트워크 연결**
    - 네임스페이스에 라우팅 테이블 추가 (게이트웨이 설정)
    - NAT 설정 (iptables 사용)
    - 기본 게이트웨이 설정
3. **외부에서 네임스페이스 접근**
    - 외부 호스트에 라우트 추가 방법
    - 포트 포워딩 설정 방법 (iptables 사용)

## Prerequisite - Docker Networking

### 도커 네트워크 옵션

1. **None 네트워크**
    - 컨테이너가 어떤 네트워크에도 연결되지 않음
    - 외부 세계와 통신 불가능
    - 여러 컨테이너 간 통신도 불가능
2. **Host 네트워크**
    - 컨테이너가 호스트 네트워크에 직접 연결됨
    - 호스트와 컨테이너 간 네트워크 격리 없음
    - 포트 매핑 필요 없이 호스트 포트로 직접 접근 가능
    - 같은 포트를 사용하는 여러 컨테이너 실행 불가능
3. **Bridge 네트워크**
    - 도커 설치 시 기본으로 생성되는 내부 프라이빗 네트워크
    - 기본 주소는 172.17.0.0
    - 도커에서는 `Bridge`로 불리지만 호스트에서는 `Docker0`로 표시됨

### 컨테이너 네트워크 연결 방식

1. 도커는 컨테이너 생성 시 네트워크 네임스페이스 생성
2. 가상 케이블(virtual cable)을 생성해 양쪽에 인터페이스 연결
    - 한쪽은 브릿지(Docker0)에 연결
    - 다른 쪽은 컨테이너 네임스페이스에 연결
3. 컨테이너에 네트워크 내 IP 할당 (예: 172.17.0.3)

### 포트 매핑

- 기본적으로 컨테이너는 내부 네트워크에 있어 외부에서 접근 불가
- 도커의 포트 매핑 기능으로 외부 접속 허용 가능
- 예: 호스트의 8080 포트를 컨테이너의 80 포트로 매핑
- 구현 방식: IP 테이블을 사용한 NAT 규칙 생성
- 도커는 자동으로 IP 테이블에 규칙 추가하여 트래픽 전달

## Prerequisite - Prerequisite - CNI

- CNI(Container Network Interface)란?
    - 컨테이너 런타임 환경의 네트워킹 문제를 해결하기 위한 표준 세트
    - 프로그램 개발 방법과 컨테이너 런타임이 이를 호출하는 방법 정의
    - 이런 것들을 플러그인이라고 부름

- 컨테이너 런타임 책임
    - 각 컨테이너에 대한 네트워크 네임스페이스 생성
    - 컨테이너가 연결될 네트워크 식별
    - 컨테이너 생성 시 `add` 명령으로 플러그인 호출
    - 컨테이너 삭제 시 `del` 명령으로 플러그인 호출
    - JSON 파일로 네트워크 플러그인 구성
- 플러그인 책임
    - `add`, `del`, `check` 명령줄 인수 지원
    - 컨테이너에 IP 주소 할당
    - 컨테이너 간 통신에 필요한 경로 구성
- 지원 플러그인
    - 기본 플러그인: Bridge, VLAN, IP VLAN, MAC VLAN, Windows 등
    - IPAM 플러그인: Host Local, DHCP 등
    - 서드파티 플러그인: Weave, Flannel 등

- Docker는 CNI를 구현하지 않고 자체 Container Network Model(CNM) 사용
- Docker와 CNI는 직접 통합되지 않음
- Kubernetes에서 Docker 사용 시: 네트워크 없이 Docker 컨테이너 생성 후 수동으로 CNI 플러그인 호출하는 방식 사용

## Cluster Networking

- 노드 기본 요구사항
    - 마스터와 워커 노드로 구성된 쿠버네티스 클러스터 필요
    - 각 노드는 최소 하나의 네트워크 인터페이스 필요
    - 각 인터페이스에 주소 구성 필요
    - 고유한 호스트명과 MAC 주소 필요 (특히 VM 클로닝 시 주의)
- 마스터 노드
    - 6443: API 서버 (외부 사용자 및 모든 구성요소가 접근)
    - 10250: kubelet
    - 10259: kube scheduler
    - 10257: kube controller manager
    - 2379: ETCD 서버
- 다중 마스터 노드
    - 위의 모든 포트 + 2380 (ETCD 클라이언트 간 통신용)
- 워커 노드
    - 10250: kubelet
    - 30000-32767: 외부 서비스 접근용 포트
- 주의사항
    - 클라우드 환경(GCP, Azure, AWS)의 방화벽/네트워크 보안 그룹 설정 시 위 포트들 고려
    - 문제 발생 시 이 포트들이 제대로 열려있는지 확인 필요