### 클러스터 아키텍처

#### 마스터 노드

- 클러스터 전체 관리 (항구의 관제탑 역할)
- 구성 요소:
  - Kube API Server
    - 요청 처리 (인증 및 유효성 검사)
    - ETCD 저장소 관리 및 업데이트
    - 클러스터 컴포넌트 조정
  - ETCD
    - 고가용성 분산 키-값 저장소
    - 클러스터 상태 저장 (파드, 노드, 서비스)
  - Kube Scheduler
    - 자원을 고려하여 적절한 Worker Node에 파드 스케줄링
  - Controller Manager
    - 노드 컨트롤러, 레플리카 컨트롤러, 엔드포인트 컨트롤러, 서비스 계정 및 토큰 컨트롤러 포함

#### 워커 노드

- 실제 컨테이너 실행 (컨테이너를 운반하는 선박 역할)
- 구성 요소:
  - Kubelet
    - 마스터 노드 명령 수행
    - 노드 및 컨테이너 상태 보고
  - Kube Proxy
    - iptables 또는 IPVS를 사용하여 노드 간 통신 규칙 관리
  - 컨테이너 런타임 엔진
    - ContainerD, Docker 등

### Docker vs ContainerD

- 초기에는 Docker가 널리 사용됨
- Docker는 원래 containerd를 포함
- Docker → 포괄적인 플랫폼
- containerd → 독립적이고 경량화된 런타임 (CRI 호환)
- Docker의 dockershim 제거 후, containerd가 독립됨

#### CLI 비교

- ctr: containerd 디버그용 CLI (사용성 낮음)
- nerdctl: Docker와 유사한 containerd용 CLI (Docker 명령어와 매우 유사)
- crictl: Kubernetes 커뮤니티 유지 관리 CLI, 파드 수준 작업 가능

### ETCD

- 분산 고가용성 키-값 저장소
- Kubernetes 클러스터 상태 및 설정 관리
- 기본 포트: 2379
- 고가용성(HA) 클러스터의 각 ETCD는 서로 인지해야 함

#### 기본 ETCD 명령어

```bash
# ETCD 실행
./etcd

# 키-값 설정
./etcdctl set key1 value1

# 값 조회
./etcdctl get key1
```

### 쿠버네티스 컴포넌트 상호작용

#### 파드 생성 과정

1. 사용자가 명령 실행 (`kubectl apply`)
2. Kube API Server가 요청 인증, 유효성 검사 후 ETCD에 상태 저장
3. 스케줄러가 적절한 워커 노드 선택
4. Kubelet이 파드 스펙을 받아 컨테이너 런타임에 생성 지시
5. Kubelet이 API 서버에 파드 상태 보고, ETCD 업데이트

### 파드 (Pod)

- 가장 작은 배포 단위
- 컨테이너와 보통 1:1 관계
- 밀접하게 관련된 컨테이너들을 한 파드로 구성 가능

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app-container
      image: nginx:latest
```

### ReplicaSet

- 지정된 수의 파드 복제본 유지
- selector를 사용 (복잡한 조건 가능)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: web-container
          image: nginx
```

### Deployment

- ReplicaSet 관리
- 롤링 업데이트, 롤백, 자동 복구 기능 제공

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: web-container
          image: nginx
```

### Service

- 파드에 대한 추상화된 네트워크 접근 제공

#### 서비스 유형

- ClusterIP: 클러스터 내부 통신
- NodePort: 외부에서 노드 IP와 포트로 접근
- LoadBalancer: 클라우드 환경 로드밸런서 연동
- ExternalName: DNS를 통한 외부 서비스 접근
- Headless: ClusterIP 없이 개별 파드 DNS 접근

NodePort 예시:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30008
```

### 네임스페이스 (Namespace)

- 다중 사용자 또는 환경(dev, staging, prod) 간 논리적 리소스 분리

```bash
kubectl create namespace dev
kubectl config set-context --current --namespace=dev
```

### Imperative vs Declarative

- Imperative: 구체적인 명령어 및 절차를 직접 지정
- Declarative: 원하는 상태 정의, 쿠버네티스가 상태 유지

### kubectl apply command

```bash
kubectl apply -f deployment.yaml
```
