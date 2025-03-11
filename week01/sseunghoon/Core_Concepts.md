## Cluster Architecture

### 용어 (+비유)

- Worker Node
    - 컨데이너를 적재할 수 있는 선박
- Master Node
    - 적재를 계획하고 실행하고 선박에 대한 데이터를 저장하고 추적하는 등의 적재 관리
- ETCD
    - 고가용 Key-Value Store
    - 각 컨테이너가 어떤 선박에 몇시에 적재되었는지 저장
- Kube-Scheduler
    - 컨테이너를 설치하기 위한 올바른 노드를 결정
    - 컨테이너 리소스 요구사항
    - 워커 노드 용량
    - 정책, 제약 조건 테인트(?), 관용, 노드 친화성 규칙 등
- Controller Manager
    - Node Controller
    - Replication Controller
- Kube API Server
    - 각 요소들끼리 통신하는 수단
    - 모든 작업을 오케스트레이션
- Container Runtime Engine
    - docker, containerd, rocket
- Kubelet
    - 각 노드에서 실행되는 에이전트
    - Kube API 서버의 지시를 따라 컨테이너를 배포 및 파괴
    - 상태 보고 (노드, 컨테이너)
- Kube Proxy
    - Worker Node간의 통신 규칙

## Docker vs ContainerD
- `containerd`는 `Docker`의 일부였지만, 현재는 독립된 프로젝트
    - containerd를 포함한 Docker를 설치할 필요없이 containerd만 있어도 container를 설치 가능함.

## ETCD
- Key Value Store
    - 데이터가 복잡해지면 json, yaml 등의 포맷에서 트랜잭션 처리
- Port 2379
- ETCDCTL
- 노드 추가, 파드 배포 등의 작업은 ETCD 서버에 업데이트 되어야 완료로 간주
- 고가용성 환경에서 마스터 노드가 여러 개가 되고, ETCD도 여러 개가 되었을 때, 각각의 ETCD가 서로를 알게끔 해야 함

## Kube API Server

### 파드 생성 프로세스

1. kubectl 또는 API 요청
    - 사용자가 kubectl 명령어나 API를 통해 파드 생성 요청
    - 요청은 Kube API Server로 전달됨
2. API Server 처리
    - 인증 및 권한 검증 수행
    - 요청을 검증하고 ETCD에 저장
3. 스케줄러 작업
    - Kube-Scheduler가 새로운 파드를 감지
    - 적절한 Worker Node 선택 (리소스, 제약조건 등 고려)
4. Kubelet 실행
    - 선택된 노드의 Kubelet이 파드 스펙 수신
    - Container Runtime(containerd/docker)에게 컨테이너 생성 지시
5. 상태 업데이트
    - Kubelet이 파드 상태를 API Server에 보고
    - API Server가 ETCD에 최신 상태 저장

## Kube Controller Manager
- Node Controller: 노드의 상태를 모니터링하고 관리
- Replication Controller: 지정된 수의 파드 Replication 관리
- Endpoints Controller: 서비스와 파드를 연결
- Service Account & Token Controllers: 새로운 네임스페이스에 대한 기본 계정과 API 접근 토큰을 생성

### Node Monitor Period와 Grace Period

- Node Monitor Period: 노드 컨트롤러가 노드의 상태를 확인하는 주기 (기본값: 5초)
- Node Monitor Grace Period: 노드를 'Unreachable' 상태로 표시하기 전까지 기다리는 시간 (기본값: 40초)

### POD Eviction Timeout

- 노드가 'Unreachable' 상태가 된 후, 해당 노드의 파드들을 제거하기까지 기다리는 시간 (기본값: 5분)
- 이 시간이 지나면 노드의 파드들이 강제로 종료되고 다른 정상 노드로 재스케줄링됨

> (Kubeadm)
Controller Manager 는 마스터 노드의 kube-system 네임스페이스에 POD로 위치
+ POD 정의 파일에서 옵션을 볼 수 있음
`/etc/kubernetes/manifests/kube-controller-manager.yml`
> 

> Non-Kubeadm
`/etc/systemd/system/kube-controller-manager.service`
> 

### Kubeadm

- 자동화된 클러스터 설정 도구
- 쿠버네티스 컴포넌트들이 Pod로 실행됨
    - 설정 파일: `/etc/kubernetes/manifests/` 디렉토리에 위치
    - 컴포넌트 관리가 쿠버네티스 자체적으로 이루어짐
- 업그레이드와 관리가 상대적으로 쉬움

### Non-Kubeadm

- 수동으로 각 컴포넌트를 설치하고 구성
- 컴포넌트들이 systemd 서비스로 실행됨
    - 설정 파일: `/etc/systemd/system/` 디렉토리에 위치
    - 각 컴포넌트를 개별적으로 관리해야 함
- 더 많은 커스터마이징이 가능하지만 관리가 복잡함

## Kube Scheduler

> 주의: 파드를 노드에 놓는 주체가 아님. (Kubelet이 포드를 노드에 넣는 주체)
>
### 스케줄링 프로세스

1. 필터링 (Filtering)
    - Pod가 실행될 수 있는 적절한 노드를 찾음
    - 리소스 요구사항, 하드웨어/소프트웨어 제약조건, 어피니티 규칙 등을 고려
2. 스코어링 (Scoring)
    - 필터링을 통과한 노드들에 대해 순위를 매김
    - 리소스 사용량, 파드 분산도, 노드 선호도 등을 고려하여 점수 계산
3. 바인딩 (Binding)
    - 가장 높은 점수를 받은 노드에 Pod를 할당
    - 결정된 내용을 API 서버를 통해 etcd에 저장

## Kubelet

- 각 노드에서 실행되는 에이전트
- Pod 관리
- 노드-마스터 통신
- 상태 모니터링
- 리소스 관리

## Kube Proxy

- 로드 밸런싱
- 네트워크 규칙 관리
- 프록시 모드
    - **iptables** 모드: Linux 커널의 iptables를 사용
    - **ipvs** 모드: IP Virtual Server

## Kubernetes Service

> Pod 집합에 대한 네트워크 액세스를 제공하는 추상화된 방법
서비스는 Pod의 IP 주소가 바뀌어도 서비스 이름을 통해 접근할 수 있게 해줌

⭐ Service의 종류가 다양함
> 
- **ClusterIP**
    - 기본 Service 유형
    - 클러스터 내부에서만 접근 가능한 가상 IP 할당
    - 클러스터 내 Pod 간 통신에 사용
- **NodePort**
    - 모든 노드의 특정 포트를 개방하여 외부 접근 허용
    - 포트 범위: 30000-32767 (기본 설정)
    - `<노드IP>:<NodePort>`로 접근
    
    (예시 YAML)
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      type: NodePort
      selector:
        app: my-app
      ports:
        - port: 80        # 클러스터 내부에서 사용할 포트
          targetPort: 8080  # Pod의 포트
          nodePort: 30007   # 외부에서 접근할 노드 포트 (생략 가능)
    ```
    
- **LoadBalancer**
    - 클라우드 제공업체의 로드 밸런서를 프로비저닝
    - 외부 트래픽을 클러스터의 Service로 라우팅
    - AWS ELB, GCP CLB, Azure Load Balancer 등과 통합
- **ExternalName**
    - 클러스터 내에서 외부 서비스에 접근할 때 사용
    - DNS CNAME 레코드를 생성하여 외부 도메인 이름으로 리다이렉션
    - 프록시나 포워딩 없이 DNS 레벨에서 작동
- **Headless**
    - ClusterIP가 `None`으로 설정된 Service
    - 로드 밸런싱 없이 모든 Pod의 DNS 레코드 생성
    - 고급 서비스 디스커버리 시나리오에 사용
- **ExternalIP**
    - 특정 외부 IP를 Service에 할당
    - 해당 IP로 들어오는 트래픽을 Service로 라우팅

## Pods with YAML

- Kubernetes의 configuration yml 파일은 항상 다음 4가지를 가짐
    - apiVersion
    - kind
    - metadata
        - name
        - `labels` **딕셔너리**, 원하는 대로 키-값을 가질 수 있음
    - spec
        - `containers` **배열**
            - name
            - image

`kubectl describe pod myapp-pod`

`kubectl apply -f pod.yaml` (apply = create)

## ReplicaSets

- **파드 개수 유지**: 설정된 복제본 수(replicas)에 따라 파드 수를 자동으로 유지
- **자가 복구**: 노드 장애나 파드 충돌로 파드가 삭제되면 자동으로 새 파드를 생성
- **파드 선택**: 라벨 셀렉터(label selector)를 사용하여 관리할 파드를 식별
- **스케일링**: `kubectl scale` 명령어로 쉽게 확장/축소

### Replication Controller vs Replica Set

**Replication Controller (RC)**

- 기본적인 라벨 셀렉터만 지원 (equality 기반: `key=value`)
- 파드 템플릿 변경 시 기존 파드에 영향 없음

**ReplicaSet (RS)**

- 더 강력한 셀렉터 지원 (`matchLabels`, `matchExpressions` 포함)
- 집합 기반 셀렉터 지원 (`in`, `notin`, `exists` 연산자)

`kubectl replace -f {filename}`

`kubectl scale --replicas=6 -f {filename}`

`kubectl scale --replicas=6 {TYPE:replicaset} {NAME}`

`kubectl set image replicaset/new-replica-set busybox-container=busybox`

## Deployments

- 애플리케이션의 원하는 상태를 정의
- `롤링 업데이트`
    - 애플리케이션을 다운타임 없이 업데이트
    - 기존 버전의 Pod를 하나씩 종료하고 새 버전의 Pod를 생성
- `롤백`
    - **리비전 히스토리**: Deployment는 각 업데이트의 리비전 기록을 유지
    
    ```bash
    # 리비전 히스토리 확인
    kubectl rollout history deployment/my-app
    
    # 특정 리비전 상세 정보 확인
    kubectl rollout history deployment/my-app --revision=2
    
    # 이전 버전으로 롤백
    kubectl rollout undo deployment/my-app
    
    # 특정 버전으로 롤백
    kubectl rollout undo deployment/my-app --to-revision=2
    
    # 롤아웃 상태 확인
    kubectl rollout status deployment/my-app
    ```
    
- 스케일링(확장/축소) 관리
- 자동 복구 메커니즘

> **ReplicaSet과의 차이점**: 
Deployment는 ReplicaSet을 관리, ReplicaSet은 Pod를 관리
Deployment가 없는 ReplicaSet만 사용하면 롤링 업데이트나 롤백이 불가능
> 

## NameSpace

> 같은 클러스터 내에서 서로 다른 팀, 프로젝트, 환경을 위한 리소스를 분리
> 
- 기본 네임스페이스
    - `default`: 기본적으로 리소스가 생성되는 공간
    - `kube-system`: 쿠버네티스 시스템 구성요소용
    - `kube-public`: 모든 사용자가 읽을 수 있는 공개 정보용
    - `kube-node-lease`: 노드 가용성 모니터링용
- 주요 사용 사례
    - **멀티 테넌트 환경**: 여러 팀이 하나의 클러스터를 공유
    - **환경 분리**: 개발, 테스트, 스테이징, 프로덕션 환경 분리
    - **리소스 할당**: 네임스페이스별 리소스 쿼터 설정 가능
    - **액세스 제어**: RBAC을 통한 네임스페이스별 권한 관리

## Declarative (vs Imperative)

- 무엇을 원하는지 선언 (vs 어떻게 실행할지 명령)
- 구성 변경 사항 추적 용이
- 대규모 클러스터 관리에 적합
- 반복 가능하고 일관된 배포

`kubectl expose pod redis --name=redis-service --port=6379`

`kubectl run custom-nginx --image=nginx --port=8080`

`kubectl expose pod httpd --name=httpd --type=ClusterIP --port=80 --target-port=80`

## Kubectl Apply Command

> 로컬 파일 수정 후 재 적용 시
Local file configuration ↔ Live object configuration 를 비교
>
