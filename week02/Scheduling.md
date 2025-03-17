# Scheduling

## Manual Scheduling

### How scheduling works

1. 모든 파드에는 nodeName 필드가 존재하지만, 기본값으로 정의되지 않음
2. 스케줄러는 nodeName 필드가 없는 파드를 찾아서 파드에 노드를 지정함

### No Scheduler

- 스케줄러가 없다면 Pending 상태가 지속됨
    - nodeName 필드를 설정하여 직접 노드에 파드를 할당할 수도 있음
    - 주의할 점은 **`nodeName은 생성 시에만 지정가능`**
    - 파드가 이미 생성되어 있을 경우, 바인딩 개체를 생성하고 포드의 바인딩 API을 사용해야 함.
        
        → 실제 스케줄러가 하는 일을 모방
        

![image.png](Scheduling%201b8ae528fb5d80629c26f851de89413c/image.png)

## Labels & Selectors

### Labels

- 키-값 쌍 형태의 메타데이터
- 객체 구별, 분리 및 그룹화

```bash
metadata:
  labels:
    environment: production
    app: nginx
    tier: frontend
    version: v1.0
```

### Selectors

- 종류
    - **Equality-based**: `=`, `==`, `!=` 연산자 사용
    - **Set-based**: `in`, `notin`, `exists` 연산자 사용

`kubectl get pods -l environment=production,tier=frontend`

```yaml
selector:
  matchLabels:
    app: nginx
  matchExpressions:
    - {key: tier, operator: In, values: [frontend]}
    - {key: environment, operator: NotIn, values: [dev]}
```

### Annotations

- 전화번호, 이메일 등 통합 목적으로 작성

## Taints and Tolerations

- Taint
    - 벌레가 사람 몸에 앉는 것을 방해하기 위해서 뿌리는 스프레이
    - 종류
        - `NoSchedule`: 톨러레이션이 없는 파드는 스케줄링되지 않음
        - `PreferNoSchedule`: 가능하면 스케줄링하지 않으나, 대안이 없으면 허용
        - `NoExecute`: 새 파드의 스케줄링을 방지하고, 이미 실행 중인 톨러레이션 없는 파드도 제거
    
    `kubectl taint nodes node1 key=value:NoSchedule`
    
- Tolerant
    - 스프레이를 참을 수 있는 벌레가 가지는 인내성
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
      tolerations:
      - key: "key"
        operator: "Equal"
        value: "value"
        effect: "NoSchedule"
    ```
    

> 마스터 노드는 아래와 같은 테인트가 적용되어 있음
`node-role.kubernetes.io/master:NoSchedule`
> 

## Node Selectors

- 파드(Pod)를 특정 노드(Node)에 배치하기 위한 가장 간단한 제약 방법
- 명령어 양식
    
    `kubectl label nodes <node-name> <label-key>=<label-value>`
    
- 명령어 예시
    
    `kubectl label nodes worker-node-1 disk=ssd`
    

### 한계점

- 복잡한 조건 불가
- 선호도(preference)가 아닌 필수 요구사항으로만 작동
    - 일치하는 노드가 없으면 스케줄링 X

## Node Affinity

- 종류
    - `requiredDuringSchedulingIgnoredDuringExecution`
        
        반드시 충족해야 하고, 실행 중인 Pod에 영향 X
        
    - `preferredDuringSchedulingIgnoredDuringExecution`
        
        충족하지 않는 노드들만 있어도 스케줄링 됨. 각 조건에 가중치 부여 가능
        
    - `requiredDuringSchedulingRequiredDuringExecution`
        
        반드시 충족해야 하고, 실행 중인 Pod에 영향 O
        
- 연산자
    - `In`: 레이블 값이 지정된 값 중 하나와 일치
    - `NotIn`: 레이블 값이 지정된 값과 일치하지 않음
    - `Exists`: 지정된 키의 레이블이 존재
    - `DoesNotExist`: 지정된 키의 레이블이 존재하지 않음
    - `Gt`: 레이블 값이 지정된 값보다 큼(Greater than)
    - `Lt`: 레이블 값이 지정된 값보다 작음(Less than)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disk-type
            operator: In
            values:
            - ssd
```

## Taints and Tolerations vs Node Affinity

- 두 조합을 함께 사용해야 디테일한 설정 가능
- **Taints/Tolerations**: "이 노드에 어떤 파드가 올 수 없는지" 제어 (배제 메커니즘)
- **Node Affinity**: "이 파드가 어떤 노드에 배치되어야 하는지" 제어 (선택 메커니즘)

## Resource Requirements and Limits

![image.png](Scheduling%201b8ae528fb5d80629c26f851de89413c/image%201.png)

![image.png](Scheduling%201b8ae528fb5d80629c26f851de89413c/image%202.png)

![image.png](Scheduling%201b8ae528fb5d80629c26f851de89413c/image%203.png)

### Resource Requests

- 컨테이너가 필요로 하는 최소한의 리소스 양
- 쿠버네티스 스케줄러가 파드를 배치할 노드를 결정할 때 이 값을 사용

### Resource Limits

- 컨테이너가 사용할 수 있는 최대 리소스 양
    - 초과하면 컨테이너는 제한(CPU) 또는 종료(메모리)

![image.png](Scheduling%201b8ae528fb5d80629c26f851de89413c/image%204.png)

### Resource Quotoas

- NameSpace 별로 Resource를 제한할 수 있음

## DaemonSets

- 자동 Pod 배포
    - 클러스터에 새로운 노드가 추가될 때 자동으로 Pod 생성
    - 노드가 제거될 때 해당 Pod도 자동 삭제
- 고정 실행 보장
    - 노드당 정확히 하나의 Pod 실행을 보장
    - Pod가 실패하거나 삭제되면 자동으로 재생성
- 사용 예시
    - 로그 수집기 (Fluentd, logstash)
    - 모니터링 에이전트 (Prometheus Node Exporter)
    - 노드 모니터링 도구
    - CNI 네트워크 플러그인
    - kube-proxy

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-agent
spec:
  selector:
    matchLabels:
      name: monitoring-agent
  template:
    metadata:
      labels:
        name: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent:latest

```

### DaemonSet vs ReplicaSet

- ReplicaSet: 클러스터 전체에서 지정된 수의 Pod 복제본 유지
- DaemonSet: 각 노드에 정확히 하나의 Pod 실행 보장

## Static Pods

- kube-apiserver가 아닌 개별 노드의 kubelet에 의해 관리되는 파드
- 개별 노드의 `/etc/kubernetes/manifests` 에 저장된 manifest 파일에 의해 정의
- `/var/lib/kubelet/config.yaml` 에 의해 경로 변경 가능
- Static Pod가 실행되면 노드 이름이 설정됨
    - [원본-pod-이름]-[노드이름]

## Multiple Schedulers

### Custom Scheduler

- 고유의 논리를 가지고 있는 커스텀 스케줄러
- 스케줄링 알고리즘과 로직을 직접 구현
- schedulerName 필드 사용

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  schedulerName: my-custom-scheduler
  containers:
  - name: my-container
    image: my-image

```

## Configuring Scheduler Profiles

여러 스케줄링 프로필을 정의하고 각 프로필에서 다른 플러그인 집합 또는 설정을 가질 수 있음
`/etc/kubernetes/manifests/kube-scheduler.yamlkube-scheduler --config <filename>`

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: default-scheduler
  plugins:
    queueSort:
      enabled:
        - name: PrioritySort
    preFilter:
      enabled:
        - name: PodTopologySpread
    filter:
      enabled:
        - name: NodeUnschedulable
        - name: NodeName
    score:
      enabled:
        - name: NodeResourcesBalancedAllocation
          weight: 1
        - name: ImageLocality
          weight: 1
- schedulerName: custom-scheduler
  plugins:
    queueSort:
      enabled:
        - name: PrioritySort
    preFilter:
      enabled:
        - name: PodTopologySpread
    filter:
      enabled:
        - name: TaintToleration
        - name: NodeAffinity
    score:
      enabled:
        - name: NodeResourcesAllocated
          weight: 1
        - name: InterPodAffinity
          weight: 1
```

## **Admission Controllers**

- `kube-apiserver.yaml` 의 command 옵션으로 활성화 / 비활성화 (구분자=`,`)
    - `--enable-admission-plugins`
    - `—disable-admission-plugins`

## Validating and Mutating Admission Controllers

- `spec:securityContext:runAsNonRoot` 기본 값: false

API 서버로의 요청을 가로채서 검증 및 변환 작업을 수행하는 플러그인

- Mutating Admission Controllers
요청처리 전, 수정 (기본 값을 설정, Annotation을 부여 등)
    - PodPreset: 설정을 자동으로 적용하여 Pod 사양을 수정
    - DefaultTolerationSeconds: Pod에 기본 toleration 설정 추가
- Validating Admission Controllers
요청처리 전, 검증 (정책 준수 확인 및 거부)
    - NamespaceLifecycle: 네임스페이스 수명 주기 관리. 삭제된 네임스페이스에 리소스를 생성하는 요청 차단
    - ResourceQuota: 네임스페이스가 할당된 리소스 쿼터를 초과하지 않도록 요청을 제한
    - PodSecurityPolicy: Pod의 보안 정책을 검증합니다.