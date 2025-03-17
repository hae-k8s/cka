## Cluster Architecture

<b>Master Node</b>

- node controller, replica controller 로 구성
- `kube-apiserver` 를 통해 master 노드 내 컨트롤러들을 오케스트레이션

<b>Worker Node</b>

- `kubelet` 은 master node 의 명령을 수행하는 선장 역할
- master node 의 지시에 따라 container 를 생성, 파괴함.
- kube proxy 는 노드 내 pods 간, 노드 간 통신을 지원함

## ETCD

- ETCD 는 key-value store
- node pods config 등 정보 저장

## kube-apiserver

- `kube-apiserver` 는 ETCD 에서 데이터를 읽고 각 node 와 상호작용하게 됨
- 유저인증, 요청검증, 데이터수신, ETCD 업데이트, 스케쥴링, kubelet 명령 제어
- kubelet 과 `kube-apiserver` 는 상호보완적

## kube controller manager

- container 를 감시하고 필요한 조치를 함
- 대표적으로 node-controller, replication-controller
- 모든 컨트롤러는 기본적으로 모두 활성화됨

```bash
# 중요한 기본 설정 옵션
—node-monitor-period=5s # 노드 상태 검사 주기
—node-monitor-grace-period=40s # 노드 NotReady 상태로 전환하기 까지 유예 기간
—pod-eviction-timeout=5m0s # NotReady 상태의 노드를 퇴출하기까지 대기 시간
```

## kube scheduler

- pod 를 어떤 node 에 놓을지 결정

1. 리소스에 따라 필터링
2. 우선순위 알고리즘에 따라 노드에 점수를 매기고 결정

## kubelet

- master 의 지시를 받아 컨테이너 실행
- 지속적으로 kube api server 에 상태 보고

## kube-proxy

- 쿠버네티스의 컨테이너끼리 연결할 수 있도록 가상 iptable 을 통해 트래픽 라우팅
- 각 pod 컨테이너는 쿠버네티스 메모리에 동적으로 생성되고 외부 식별이 불가하기 때문에 필요함

## Pods

- pod 는 쿠버네티스에서 구성할 수 있는 가장 작은 단위 (단일 인스턴스)
- pod 와 container 는 보통 1:1 관계, 스케일 아웃할 경우 새 pod 를 생성하게 됨
- 그러나 helper container 같이 서로 밀접한 관련이 있는 container 는 하나의 pod 에 구성할 수도 있음.
  이럴 경우 두 container 가 같이 생성되고 같이 파괴됨

## Pods with YAML

- 쿠버네티스 pod-definition 파일은 항상 apiVersion, kind, metadata, spec 이라는 4개의 루트레벨 속성을 가짐 (required)

```yaml
apiVersion: 개체 생성시의 쿠버네티스 API Version
kind: 개체 유형
metadata: 개체의 이름이나 label, type 같은 메타데이터
spec: container 의 속성 리스트 (docker image 속성)
```

## ReplicaSets

- 특정 pod 가 항상 실행되도록 보장 (pod 가 1개여도 동작)
- selector 가 반드시 정의되어야 함
- replication controller (구버전) -> replica set (신버전)

```yaml
# replication controller 정의
apiVersion: v1
kind: ReplicationController
spec.template: # 정의할 pod 의 metadata, spec 명시
spec.replicas: # 복제본 개수
```

```yaml
# replica set 정의
apiVersion: apps/v1
kind: ReplicaSet
spec.template: # 정의할 pod 의 metadata, spec 명시
spec.replicas: # 복제본 개수
spec.selector: # rc 와 replicaset 의 가장 큰 차이. replicaset 에 속하지 않는 기존 pod 도 label 참조로 가져올 수 있음 spec.selector.matchLabels.{label} 사용
```

## deployments

- pods 의 배포, 롤백을 담당

```yaml
# deployment 정의
apiVersion: apps/v1
kind: Deployment
```

- 모든 객체 보기 = `kubectl get all`
- yaml 파일 쉽게 생성 = `kubectl run nginx --image=nginx --dry-run=client -o yaml`

## services

- 외부에서 쿠버네티스 내부 pods 에 접근하기 위해 생성

```bash
1. NodePort : 내부 포트를 엑세스할 수 있도록 오픈
2. ClusterIP : 클러스터 내에서 가상 IP 를 만들어 통신
3. LoadBalancer : 로드밸런싱
```

<b>NodePort</b>

- TargetPort : pods 의 오픈 포트
- Port : 클러스터에서 접근할 포트
- NodePort : 해당 노드에 접근하여 서비스에 접근

```yaml
spec:
  type: NodePort
  ports:
    - targetPort: # (Opitonal)
      port: # (required)
      nodePort: # (Opitonal)
  selector: # pod 나 replicaset 의 labels 필드 참조하여 pods 를 선택!
```

- selector 에 선택된 pod 가 여러개면 자동으로 무작위 알고리즘으로 로드밸런싱 함
- nodePort 정보는 선택된 모든 pod 가 같은 값을 가지게됨

## services - cluster IP

- tier 간 통신 시 tier 끼리 묶어 단일 통신 지점을 제공 (frontend, backend, db ...)

```yaml
spec:
  type: ClusterIP
  ports:
    - targetPort: # (Opitonal)
      port: # (required)
  selector: # pod 나 replicaset 의 labels 필드 참조하여 pods 를 선택!
```

## services - LoadBalancer

```yaml
spec:
  type: LoadBalancer
  ports:
    - targetPort: (Opitonal)
      port: (required)
      nodePort: (Opitonal)
```

## namespaces

```bash
기본 - Default
기본 시스템 - kube-system
기본 리소스 - kube-public
```

- dev, prod 분리할 때 유용
- 각각에서 리소스 할당량 지정 가능 (yml 파일)
- 다른 네임스페이스에 접근할 때는 서비스에 네임스페이스를 추가해야함
- 다른 네임스페이스에 접근하려면 `--namespace={namespace}` 옵션 추가 필요
  <br/>
- 네임스페이스 생성은 namespace 정의 파일 yml 사용
- set-context 로 기본 namespace 이동 가능

## Imperative vs Declarative

- Imperative - 단계별로 정확하게 어떻게 할지 명령
  단일성이고 현재 환경 설정을 인지하고 변경하기 전에 반드시 확인이 필요함

- Declarative - 최종 목표만 지정.
  어떻게가 아니라 무엇을 할지 지정

## kubectl apply command

- `kubectl apply -f nginx.yaml` 과 같은 apply 명령어로 적용
- live object configuration 은 상태 저장을 위한 다른 필드도 포함됨
- local file 과 live object configuration 을 비교하여 live 설정을 업데이트 시킴
