# Networking

## Pod Networking

- 기본적으로 내장돼있는 솔루션은 없음
- 필수 요건만 제시함
  - 모든 pod는 각자의 IP 주소를 가져야 함
  - 모든 pod는 동일 노드 내의 다른 모든 pod와 소통 가능해야 함
  - 모든 pod는 NAT 없이 다른 노드의 모든 pod와 소통 가능해야 함

pod 네트워킹

```
 # Create veth pair
 iplink add ……
 # Attach vethpair
 iplink set ……
 iplink set ……
 # Assign IP Address
 ip-n <namespace> addradd ……
 ip-n <namespace> route add ……
 # Bring Up Interface
 ip-n <namespace> link set ……
```

다른 node 네트워킹

```
 node1$ ip route add 10.244.2.2 via 192.168.1.12
 node1$ ip route add 10.244.3.2 via 192.168.1.13
 node2$ ip route add 10.244.1.2 via 192.168.1.11
 node2$ ip route add 10.244.3.2 via 192.168.1.13
 node3$ ip route add 10.244.1.2 via 192.168.1.11
 node3$ ip route add 10.244.2.2 via 192.168.1.12
```

net-script.sh

```
 ADD)
   # Create veth pair
   # Attach veth pair
   # Assign IP Address
   # Bring Up Interface
   ip-n <namespace> link set ……
 DEL)
   # Delete veth pair
   ip link del ……
```

## CNI

/opt/cni/bin 폴더에 CNI 플러그인들이 들어가있음
/etc/cni/net.d 폴더에 configration 파일들이 있음

### weaveworks

```
// 배포
KUBEVER=$(kubectl version | base64 | tr -d '\n')
kubectl apply -f https://reweave.azurewebsites.net/k8s/net?k8s-version=$KUBEVER
// 조회
kubectl get pods -n kube-system
```

## Service Networking

- 한 pod를 클러스터 내에서 접근 가능하기를 원할 때 service를 생성하는 방식으로 가능함
  - 해당 클러스터 ip > pod ip로 향하는 iptable 생성됨
- kube-proxy를 사용하면 같은 방식으로 작동하는 것을 볼 수 있음
  - 서비스가 생성됨

```
// proxy 정보 확인(ip range, proxy 종류 등등)
k logs {proxy-pod-name}
```

## DNS in Kubernetes

- 클러스터 생성 시 기본적으로 dns 서버가 생성됨
- 서비스 생성 시 dns server에 호스트네임 - ip 매핑이 저장됨
- {hostname}.{namespace}.{type}.{root}
- pod의 경우 hostname은 ip 주소에서 .을 -로 바꾼 값이 됨

### CoreDNS

- 1.12.0 이전에는 kube-dns
- 이후에는 CoreDNS
- pod 생성 시 ip 주소에서 .을 -로 바꾼 값으로 hostname을 설정해 매핑을 dns 서버에 저장함
- /etc/coredns/Corefile 을 통해 coredns 설정 가능

## Ingress

- 배포하는데 Ingress controller 솔루션이 필요함
  - nginx ingress controller
  - gce
  - haproxy
  - traefik
  - istio
- 위 컨트롤러 중 하나를 배포해야 함 (리소스 정의만 한다고 해서 작동 안함)
- Deployment, Service, ConfigMap, Auth(ServiceAccount)를 준비해야 Controller 사용이 가능함

Ingress Resource

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  rules:
  - http:
    paths:
    - path: /wear
  	  backend:
        serviceName: wear-service
	    servicePort: 80
    - path: /watch
  	  backend:
        serviceName: watch-service
	    servicePort: 80
  // host로 구별
  - host: wear.my-online-store.com
	http:
	  paths:
      - backend:
		  serviceName: wear-service
		  servicePort: 80
  - host: watch.my-online-store.com
	http:
	  paths:
      - backend:
		  serviceName: watch-service
		  servicePort: 80
```

## Gateway API

- ingress는 multi-tenency 환경에 적합하지 못함
- path 및 host를 제외한 것들(tcp, ssl, ...)을 조정하기가 불편함

3가지 리소스가 있음

- GatewayClass
  - 인프라공급자 정의(Cloud, 온프레미스 등등)
  - gateway controller 필요
- Gateway
  - 리스너 정의
- HTTPRoute
  - 세부 룰 정의
