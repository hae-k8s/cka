# Networking

## Prerequisite

### Switching Routing

- 같은 네트쿼크 내의 ip와 통신 가능함
- 다른 네트워크와 통신하기 위해서는 라우터를 이용해야 함
- 이때 gateway를 통해 출입함
- 여러 네트워크와 통신할 경우 각각 라우터를 모두 추가해줘야 하며, 0.0.0.0 과 같은 방식으로 default 라우터 설정이 가능함

### DNS

- ip에 name을 붙여 관리할 수 있다
- 이를 통합해서 관리하기 위해 dns server에서 관리
- Org DNS -> Root DNS -> .com DNS -> Google DNS
- 한번 찾은 후에는 캐싱

### Namespace

- 네트워크를 격리하기 위한 수단
- Namespace 내의 네트워크는 본인 Namespace 내의 네트워크만 볼 수 있고ㅡ host는 모두 볼 수 있음
- Namespace끼리 연결도 가능함

### Docker Networking

- no network
  - 외부와 통신하지 않음
- host network
  - 호스트와 동일한 네트워크 사용
- bridge network
  - docker0 == bridge
  - 새로운 컨테이너를 만들면 한쪽 끝은 bridge에, 한쪽 끝은 컨테이너에 연결된 케이블이 만들어짐
- iptables에 rule을 추가해 8080 -> 80으로 라우팅되도록 함

### Container Networking Interface

1. Network Namespace 생성
2. Bridge Network/Interface 생성
3. Veth Pair 생성
4. Veth 한쪽을 Namespace에 연결
5. 다른 Veth 한쪽을 Bridge에 연결
6. IP 주소 할당
7. Interface 상태 확인
8. IP 포워딩 및 NAT 설정

- docker는 CNI 표준을 따르지 않고, CNM이라는 별개 표준을 정의해 사용함
- 직접 CNI 플러그인 사용은 불가능하지만, network를 none으로 선언 후 bridge를 호출해 직접 연결하는 방식으로는 사용 가능

## Cluster Networking

- 각 노드는 각각의 인터페이스, ip address, hostname, mac address를 가져야 함
- 컴포넌트별로 별도의 리스닝 포트를 뚫어줘야 함
