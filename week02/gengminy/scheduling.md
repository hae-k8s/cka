## Manual Scheduling

- pod-definition.yml 에서 `nodeName` 지정
- 이 속성은 자동으로 설정되지만 수동으로 설정 가능
- 생성할때만 지정 가능 (수정 불가)
- 나중에 지정하고 싶다면 pod-bind-definition.yml 설정하여 `kind: Binding` 객체 생성

## Labels & Selectors

- 쿠버네티스 클러스터에 수백 수천가지 객체가 생길 수 있음
- label 이 객체를 구별하고 분리 및 그룹화할 수 있게 해줌
- label 은 여러 개 지정할 수 있음

- Replicaset 의 metadata label 은 ReplicaSet 자체의 label
- `spec.selector.matchLabels` 와 `spec.template.metadata.label` 은 다른 객체를 연결하기 위한 label

## Taints and Tolerations

- 노드에 어떤 포드를 스케쥴링 할지에 대한 제한 설정
- node 는 taint 설정
- `kubectl taint nodes node-name key=value:{taint-effect}`
- taint-effect = `NoSchedule | PreferNoSchedule | NoExecu-te`
- pod 는 tolerations 설정 (`spec.tolerations`)
- tolerations 하위 key, operator, value, effect 설정 필요
- 이 설정은 특정 pod 가 반드시 특정 node 로 이동시키라는 설정이 아니고, 특정 pod 만 받을 수 있게 하려는 것임을 알자
- taint 지울때는 `key=value:{taint-effect}-` 처럼 뒤에 - 값을 붙이면 된다

## Node Selectors

- 특정 pod 가 실행될 node 지정
- pod-definition.yml 에서 `spec.nodeSelector` 지정
- `kubectl label nodes <node-name> <label-key>=<label-value>`
- 하지만 Node Selector 로는 단순 매칭만 확인 가능하고 세부적인 표현식이나 요구사항을 충족할 수 없음

## Node Affinity

- 특정 pod 가 실행될 node 지정
- operator 를 이용해 고급 표현식 사용 가능
- `spec.affinity.nodeAffinity` 지정
- `nodeSelectorTerms.matchExpressions` 지정
- `requiredDuringSchedulingIgnoredDuringExecution` => 표현식을 반드시 만족해야 pod 배치. 기존 노드는 영향 x
- `preferredDuringSchedulingIgnoredDuringExecution` => 표현식을 반드시 만족하지 않으면 아무데나 배치. 기존 노드는 영향 x
- `requiredDuringSchedulingRequiredDuringExecution` => 표현식을 반드시 만족해야 pod 배치. 기존 노드도 만족하지 않으면 퇴출

## Node Affinity vs Taints and Tolerations

- 두 기능을 함께 사용하면 특정 node 에 특정 pod 만 위치하게 할 수 있음
- Traints and Tolerations 를 이용하여 다른 pod 가 원치않는 node 에 위치하는 것을 막음
- Node Affinity 를 이용하여 특정 pod 가 다른 node 에 위치하게 되는 것을 막음

## Resource Requirements and Limits

- scheduler 는 각 pod resource 를 모니터링 하고 pod 자원 할당을 조절
- `spec.containers[].resources.requests` 에서 pod 의 memory, cpu 요구사항 지정 가능
- cpu 는 소숫점 단위로 지정 가능
- `spec.containers[].resources.limits` 에서 pod 가 사용할 수 있는 memory, cpu 한도 지정 가능
- pod 는 지정된 memory 사용량을 초과하게 되면 소멸
- Request, Limit 을 적절하게 설정해야 하나의 pod 가 자원을 모두 사용하는 상황 방지 가능
- LimitRange 객체로 resource 제어 가능
- ResourceQuota 객체로 cluster 내 전체 pod 의 resource 제어 가능

## Daemon Sets

- ReplicaSet 과 비슷하지만 Daemonsets 는 노드마다 pod 를 1개씩만 할당됨
- Monitoring Agent, Log Collector 에 주로 사용
- kube-proxy, networking
- 설정파일은 kind : DaemonSet 지정하면 나머지는 Replicaset 과 동일함

## Static Pods

- worker node 는 master node 없이도 작동할 수 있음
- `/etc/kubernetes/manifests` 에 pod yml 설정을 넣으면 kubelet 이 주기적 배포 실행
- master node 의 영향이 없는 node 에 존재하는 pods 들을 static pods 라고 함
- `--pod-manifest-path` 변경 시 설정 파일 읽는 디렉토리 변경 가능
- 또는 `/var/lib/kubelet/config.yaml` 의 staticPodPath 를 변경하여 설정 파일 디렉토리 변경
- static pod 는 읽기 전용으로 kubectl 에서 읽을 수는 있지만 삭제 불가
- master node 를 정의할 때 주로 사용함

## Multiple Schedulers

- `kind: KubeSchedulerConfiguration` 설정하고 `profiles.schedulerName` 지정
- `leaderElection` 설정으로 마스터 노드에서 여러 개 스케쥴러 설정 가능
- pod definition 에서 `spec.schedulerName` 설정하여 커스텀 스케쥴러 사용 가능
- `kubectl get events` 또는 `kubectl logs` 명령어로 스케쥴러 선택 로그 확인 가능

## Scheduler Profiles

- `spec.priorityClassName` 에 PriorityClass 객체 지정하여 node 우선순위 설정 가능
- `Scheduling Queue - Filtering - Scoring - Binding` 단계
- Scoring 단계에서는 Filtering 된 node 에 가중치를 설정
- 각 단계에는 plugin 설정 및 커스텀 가능
- Extension Point 설정으로 플러그인 전처리, 후처리 등을 확장 설정 가능
- v1.18 이상 버전부터 하나의 scheduler 설정 파일에서 multiple scheduler 설정 가능 (scheduler profile)

## Admission Controllers

- 기존 RBAC (Role Based Access Controller) 을 사용하면 특정 API 사용 제한 가능하지만 상세한 제어는 불가
- Admission Controller 는 세부 제어 가능
- 기본으로 설정되지 않은 Admission Controller 는 `--enable-admission-plugins={name}` 으로 활성화 가능
- kube-apiserver pod 확인 = `kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h`
- Adminssion Controller 상태 확인 = `ps -ef | grep kube-apiserver | grep admission-plugins`

## Validating & Mutating Admission Controllers

- `Mutating Controller` : 요청을 변경
- `Validating Controller` : 요청 유효성 검사
- 일반적으로 Mutating Controller 를 먼저 호출한 후 Validating Controller 호출하여 유효성 검사 진행함
- JSON Input/Output 가능한 Admission Webhook Server 를 만들어 자체 Controller 제공 가능
- `Admission Webhook Server` 는 Cluster 내부나 외부 API 서버로 제공 가능
