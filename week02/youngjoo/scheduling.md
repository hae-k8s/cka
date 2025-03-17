# Scheduling

## 동작 방식

- nodeName 지정
- pod 생성시에만 지정 가능
- 이미 생성된 경우 -> 바인딩 개체 생성하고 binding api 요청

```
spec:
    nodeName: node01
```

## Label & Selector

- label -> 구분하기 위한 장치
- selector -> 필터링

### Label

labels 속성을 통해 lable 지정 가능

```
metadata:
    labels:
        app: app1
        function: Front-end
```

### Selector

```
kubectl get pods --selector app=app1
```

서로 다른 오브젝트를 연결할 수 있음
replica-set에 label을 설정할 떄 실수 많이 하는 부분

```
metadata:
    labels:
        app: app1 // 여기는 replica set 자체
    annotations:
        buildVersion: 1.34
spec:
    selector:
        matchLabels:
            app: app1
    template:
        metadata:
            labels:
                app: app1 // 여기 넣어야 함
```

### Annotation

- 주석
- 정보 표시

## Toleration & Taints

- 어떤 pod가 어떤 node에 접근 가능한지 설정
- 기본적으로는 제한이 없다
- Taints -> Node
- Tolerance -> Pods
- 해당 노드로 가도록 강제하는 게 아님

### Taints

```
kubectl taint nodes node-name key=value:taint-effect
kubectl taint nodes node-name app=blue:NoSchedule
```

taint-effect

- NoSchecule, PreferNoSchedule, NoExecute

### Tolerations

taints 생성할때와 동일한 값을 넣어줘야 함

```
apiVersion:
kind: Pod
metadata:
    name: bee
spec:
    tolerations:
    - key: "app"
      operator: "Equal"
      value: blue
      effect: "NoSchecule"
```

- 마스터 노드에는 기본적으로 taint가 설정돼 어떤 pod도 할당되지 않음
- 설정으로 끌 순 있음

## Node Selector

원하는 node에 pod를 붙이기 위한 장치

```
spec:
    nodeSelector:
        size: Large
```

node에 label을 먼저 붙여야 함

```
kubectl label nodes node01 size-Large
```

### 한계

- 복잡한 경우에는? -> Node Affinity

## Node Affinity

- 원하는 pod에 node를 배치
- 복잡한 기준을 제공하는 만큼 속성도 복잡함

```
spec:
    affinity:
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution: // 라이프사이클 정의
                nodeSelectorTerms:
                - matchExpressions:
                  - key: size
                    operator: Equal // NotIn, Exists, ..
                    values:
                    - Small
                    - Large
```

- requiredDuringSchedulingIgnoredDuringExecution -> 반드시 만족해야 pod 배치. 기존 노드는 영향 x
- preferredDuringSchedulingIgnoredDuringExecution -> 표현식을 만족하지 않으면 아무데나 배치. 기존 노드는 영향 x
- requiredDuringSchedulingRequiredDuringExecution -> 표현식을 반드시 만족해야 pod 배치. 기존 노드가 만족하지 않으면 퇴출

## Node Affinity vs Taints & Tolerance

- Taints & Tolerance는 다른 node에 배치될 가능성이 있음
- Node Affinity는 다른 pod가 node에 배치될 가능성이 있음
- 둘을 함께 사용하면 해결 가능

## Resource Requirements and Limits

- 스케쥴러 관리자는 리소스에 맞춰 pod를 할당함

요청 리소스 명시

```
spec:
    containers:
        resources:
            requests:
                memory: "4Gi"
                cpu: 2
            limits:
                memory: "8Gi"
                cpu: 3
```

- 지정된 limit을 초과하면
  - cpu는 넘지 않게 조절함
  - memory는 죽음

### Limit Range

기존에 존재하는 pod는 영향 안받음

```
apiVersion: v1
kind: LimitRange
metadata:
    name: cpu-resource-contraint
spec:
    limits:
    - default:
        cpu: 500m
      defaultRequest:
        cpu: 500m
      max:
        cpu: "1"
      min:
        cpu: 100m
      type: Container
```

### Resource Quota

클러스터 전체 pod의 resource 제어 가능

```
apiVersion: v1
kind: ResourceQuota
metadata:
    name: cpu-resource-contraint
spec:
    hard:
        requests.cpu: 4
        requests.memory: 4Gi
        limits.cpu: 10
        limits.memory: 10Gi
```

## Daemon Sets

- replica set과 비슷하지만, 노드 당 한개의 pod만 할당함(항상).
- monitoring, logging solution
- kube-proxy, networking
- replica set 생성과 유사함.
  - kind: DaemonSet
- 구현 방법 -> 각 pod에 nodeName을 지정한다 (예전 방법)

## Static Pods

- Master Node가 없어도 pod 실행이 가능함
- /etc/kubernetes/manifests 에 pod yaml 파일을 두면 생성 가능
- api server 영향을 받지 않음
- deployment나 replica set은 불가능
- kubeconfig.yaml 파일에 명시해서 경로 수정도 가능
- docker 커맨드로 확인 가능
- master node를 정의할 때 사용 가능

## Multiple Schedulers

- kind: KubeSchedulerConfiguration
- config에서 profile.schedulerName을 정의
- pod에서 spec.schedulerName 명시

스케쥴러 로그 확인

```
kubectl get events
kubectl logs
```

## Scheduler Profiles

### Scheduling Queue

우선순위에 따라 정렬이 일어남

- kind: PriorityClass 생성
- pod에서 priorityClassName 지정

### Filtering

배치될 수 있는 노드 필터링

### Scoring

해당 노드에 배치된 이후 남는 리소스 기반으로 점수를 매김

### Binding

노드에 pod 바인딩

### Scheduling Plugins

- 각 단계에서 다양한 플러그인을 사용할 수 있음
- 한 플러그인이 여러 플러그인에서 사용되기도 함
- Extention Points에서 사용 가능

## Admission Controllers

- 기존 rbac -> api 제어 가능
- 더 상세한 제어를 하려면 Admission Controller 필요

```
// view
kube-apiserver -h | grep enable-admission-plugins
// enable
// kube-apiserver.service
--enable-admission-plugins=...
```

## Validating & Mutating Admission Controllers

### Mutating Controller

- 요청을 변경

### Validating Controller

- 요청 유효성 검사
- 일반적으로 Mutating Controller 이후에 거침

### AdmissionWebhookServer

- webhook server 구축해서 validating 혹은 mutating admission webhook 처리 가능
- 내부, 외부 둘다 가능
