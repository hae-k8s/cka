## Pod Networking

- 모든 Pod는 고유한 IP 주소를 가진다
- Pod는 같은 노드의 다른 Pod와 NAT 없이 직접 통신 가능해야 한다
- Pod는 다른 노드의 Pod와 NAT 없이 직접 통신 가능해야 한다
- Kubernetes는 CNI(Container Network Interface)를 사용해 Pod 생성 시 네트워크 구성을 자동화

## CNI in Kubernetes

- CNI 바이너리 디렉터리: `/opt/cni/bin`
- CNI 설정 파일 디렉터리: `/etc/cni/net.d`
- 다양한 CNI 플러그인(Calico, Flannel, Weave Net 등)을 선택해 설치 가능

## CNI Weave

- Weaveworks에서 제공하는 오픈소스 CNI 플러그인
- 각 노드에 DaemonSet 형태로 `weave`와 `weave-npc` Pod 배포
- Pod 간 트래픽을 VXLAN(또는 mTLS) 터널로 캡슐화해 전달
- 기본 네트워크 CIDR: `10.32.0.0/12` (약 1,048,576개 IP 할당)
- CNI 설정 파일(예: `/etc/cni/net.d/10-weave.conf`)의 `ipam` 섹션에서 서브넷, IP 풀, 게이트웨이 등을 정의
- Weave Net은 내부적으로 `host-local` IPAM을 사용해 Pod IP 주소를 할당·관리

## Service Networking

- Pod는 노드에 분산되어 있지만 Service는 클러스터 전체에서 단일 가상 IP(ClusterIP)로 접근 가능
- Service 유형
  - **ClusterIP**: 클러스터 내부 통신용 (기본값, IP 범위 기본 `10.96.0.0/12`)
  - **NodePort**: 클러스터 외부에서 `<노드IP>:<NodePort>`로 접근
  - **LoadBalancer**: 클라우드 제공 로드밸런서를 통해 외부 노출
- `kube-proxy`가 ClusterIP를 실제 Pod로 포워딩 (iptables 또는 IPVS 모드)
- 포워딩 룰 확인:
  ```bash
  iptables -t nat -L
  # 또는
  ipvsadm -L
  ```
- `ps aux | grep kube-apiserver` : IP Range 확인
- `cat /var/log/kube-proxy.log` : 포워딩 로그 확인

## DNS in Kubernetes

- 서비스 디스커버리를 위해 CoreDNS가 기본 DNS 서버로 작동
- 각 노드의 `/etc/resolv.conf`에 CoreDNS IP 추가
- DNS 이름 형식:
  ```bash
  <service>.<namespace>.<type>.cluster.local
  # type : svc, pod
  ```
- 같은 네임스페이스 내에서는 `http://<service>` 만으로도 접근 가능

## CoreDNS in Kubernetes

- Kubernetes v1.12 이후 기본 DNS 솔루션: CoreDNS
- CoreDNS 설정은 `ConfigMap`(`coredns`)에 저장 (`kubectl -n kube-system get configmap coredns -o yaml`)
- 실제 설정 파일 경로: `/etc/coredns/Corefile`

## Ingress

- HTTP/HTTPS 요청을 클러스터 내부 서비스로 라우팅하기 위한 Kubernetes API 오브젝트
- **Ingress Controller**: Ingress 리소스를 구현하는 컨트롤러(NGINX, Traefik 등)를 별도 설치해야 함
- Ingress 리소스에 `rules`로 호스트/경로 기반 라우팅 정의
- **Default Backend**: 정의된 규칙과 매치되지 않는 요청을 처리할 서비스

## Introduction to Gateway API

- `Ingress` 는 멀티테넌시에 대한 지원이 불가하고 호스트 매칭, 경로 매칭과 같은 기본 규칙만 지원
- `Gateway` 는 복잡한 라우팅, 멀티테넌시, TLS 설정 등을 CRD로 세분화하여 표현
- 주요 리소스
  - **GatewayClass**: 인프라 플랫폼(Cloud, On‑Premise 등) 정의
  - **Gateway**: 실제 LoadBalancer/Proxy 인스턴스 및 리스너 정의
  - **HTTPRoute**, **TCPRoute** 등: Gateway에 바인딩되어 트래픽 라우팅 규칙 설정
- 확장성 높은 표준 API로 Ingress 한계 극복
