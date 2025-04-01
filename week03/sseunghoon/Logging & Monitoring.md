# Logging & Monitoring

# Logging & Monitoring

## Monitor Cluster Components

- Metrics Server
  - 클러스터당 메트릭 서버 1개
  - 노드와 파드들에서 매트릭 수집
  - 인메모리 모니터링 솔루션
  - `minikube addons enable metrics-server`
    or
    `git clone` + `kubectl create`
  - `kubectl top node`
  - `kubectl top pod`

## Managing Application Logs

`docker run kodekloud/event-simulator`

`kubectl logs -f event-simulator-pod`

`kubectl logs -f event-simulator-pod event-simulator`

```yaml
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # 다중 컨테이너 파드
kubectl logs <pod-name> --previous  # 이전 컨테이너 로그
```

- `/var/log/containers/`: 컨테이너 로그 심볼릭 링크
- `/var/log/pods/`: 포드별 로그 디렉토리

### 로깅 아키텍처 패턴

1. **사이드카 패턴**
   - 메인 컨테이너와 함께 로그 수집기 컨테이너 배포
   - 로그 파일 처리나 전송에 유용함
2. **노드 레벨 에이전트**
   - DaemonSet으로 배포된 로그 수집 에이전트
   - 모든 노드의 로그를 중앙 시스템으로 전송
3. **중앙 집중식 로깅**
   - EFK(Elasticsearch, Fluentd, Kibana) 스택
   - ELK(Elasticsearch, Logstash, Kibana) 스택
   - Loki + Grafana

# Application Lifecycle Management

## Rolling Updates and Rollbacks

### Rollout

- 기존 상태 = Revision 1
- 롤아웃 → Revision 2 생성
- `kubectl rollout status deployment/myapp-deployment`
- `kubectl rollout history deployment/myapp-deployment`

### Deployment Strategy

![image.png](Logging%20&%20Monitoring/image.png)

- Image 변경 (아래 명령어 사용시에도 Rolling Update)
  - `kubectl apply -f depoyment-definition.yml`
  - `kubectl set image deployment/myapp-deployment nginx-container=nginx:1.9.1`

![image.png](Logging%20&%20Monitoring/image%201.png)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1 # 원하는 파드 수 이상으로 생성 가능한 최대 파드 수
      maxUnavailable: 1 # 업데이트 중 사용 불가능한 최대 파드 수
```

### Rollbacks

`kubectl rollout undo deployment/myapp-deployment`

- **이전 버전으로 복원**: 문제 발생 시 이전 안정 버전으로 빠르게 복구
- **리비전 관리**: 쿠버네티스가 자동으로 배포 이력 관리
- **즉시 롤백**: 문제 발견 시 빠른 복구 가능
- **배포 이력 확인**:
  `kubectl rollout history deployment/example-app`
- **특정 리비전 상세 정보**:
  `kubectl rollout history deployment/example-app --revision=2`
- **이전 버전으로 롤백**:
  `kubectl rollout undo deployment/example-app`
- **특정 버전으로 롤백**:
  `kubectl rollout undo deployment/example-app --to-revision=2`
- 롤링 업데이트 모니터링
  `kubectl rollout status deployment/example-app`

## Commands

- Docker 명령어 배우는 시간
  - docker ps
  - docker run
  - docker build
  - CMD vs ENTRYPOINT

## Commands and Arguments

![image.png](Logging%20&%20Monitoring/image%202.png)

`k edit pod/{Pod 명}`

`k replace —force -f /tmp/{tmp파일 명}`

## Configure Environment Variables in Applications

## Configuring ConfigMaps in Applications

![image.png](Logging%20&%20Monitoring/image%203.png)

![image.png](Logging%20&%20Monitoring/image%204.png)

`kubectl create configmap my-config --from-file=config.json`

`kubectl create configmap my-config --from-literal=key1=value1`

## Configure Secrets in Applications

![image.png](Logging%20&%20Monitoring/image%205.png)

![image.png](Logging%20&%20Monitoring/image%206.png)

`kubectl create secret generic db-secret --from-literal=password=mysecret`

## Demo: Encrypting Secret Data at Rest

### ⭐ `k create secret generic -h` ⭐ → 이런 식으로 커맨드 입력 시 예시 커맨드 출력

`k get secret my-secret -o yaml`

```yaml
etcdctl -> 설치 확인
apt-get install etcd-client
etcd-client
```

- Config 파일 예시

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
      - configmaps
      - pods.spec.containers[*].env[*].value
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ== # "secret is secure"를 base64로 인코딩
            - name: key2
              secret: dGhpcyBpcyBwYXNzd29yZA== # "this is password"를 base64로 인코딩
      - identity: {} # 복호화 용도로 필요
```

- API 서버 구성 파일에 아래 행 추가
  - `--encryption-provider-config=/etc/kubernetes/encryption-config.yaml`

## Multi Container Pods

![image.png](Logging%20&%20Monitoring/image%207.png)

`kubectl -n elastic-stack exec -it app — cat /log/app.log`

## Multi-container Pods Design Patterns

![image.png](Logging%20&%20Monitoring/image%208.png)

## InitContainers

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ["sh", "-c", "echo The app is running! && sleep 3600"]
  initContainers:
    - name: init-myservice
      image: busybox:1.28
      command:
        [
          "sh",
          "-c",
          "until nslookup myservice; do echo waiting for myservice; sleep 2; done;",
        ]
    - name: init-mydb
      image: busybox:1.28
      command:
        [
          "sh",
          "-c",
          "until nslookup mydb; do echo waiting for mydb; sleep 2; done;",
        ]
```

## Introduction to Autoscaling

![image.png](Logging%20&%20Monitoring/image%209.png)

## Horizontal Pod Autoscaler (HPA)

![image.png](Logging%20&%20Monitoring/image%2010.png)

![image.png](Logging%20&%20Monitoring/image%2011.png)

![image.png](Logging%20&%20Monitoring/image%2012.png)

![image.png](Logging%20&%20Monitoring/image%2013.png)

- **메트릭 기반 스케일링**: 주로 CPU/메모리 사용률 기준으로 동작함
- **사용자 정의 메트릭**: 커스텀 메트릭을 통해 확장 가능 (Prometheus 등 활용)
- **확장 범위 설정**: 최소/최대 Pod 수를 설정할 수 있음
- **안정화 기간**: 급격한 변동을 방지하기 위한 쿨다운 기간 설정 가능

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: 애플리케이션-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: 애플리케이션
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

## In-place Resize of Pods

### Pod를 삭제 후 다시 만들 필요 없이 실행 중인 Pod의 CPU/메모리 리소스를 조정

- **스테이트풀 애플리케이션**: 상태를 유지하면서 리소스 조정이 필요한 애플리케이션에 유용함

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: 리사이즈-테스트-파드
spec:
  containers:
    - name: 앱
      image: nginx
      resources:
        requests:
          memory: "100Mi"
          cpu: "100m"
        limits:
          memory: "200Mi"
          cpu: "200m"
      resizePolicy:
        - resourceName: cpu
          restartPolicy: NotRequired
        - resourceName: memory
          restartPolicy: NotRequired
```

`kubectl patch pod 리사이즈-테스트-파드 --patch '{"spec":{"containers":[{"name":"앱","resources":{"requests":{"cpu":"200m","memory":"200Mi"},"limits":{"cpu":"400m","memory":"400Mi"}}}]}}’`

## Vertical Pod Autoscaling (VPA)

![image.png](Logging%20&%20Monitoring/image%2014.png)

### VPA vs HPA

| 특징               | VPA (수직 스케일링)                     | HPA (수평 스케일링)                            |
| ------------------ | --------------------------------------- | ---------------------------------------------- |
| 스케일링 방법      | 기존 Pod의 CPU & 메모리 증가            | 로드에 따라 Pod 추가/제거                      |
| Pod 동작           | 새 리소스 값 적용 시 Pod 재시작         | 기존 Pod 계속 실행                             |
| 트래픽 급증 처리   | ❌ Pod 재시작으로 인해 불가능           | ✅ 즉시 추가 Pod 생성                          |
| 비용 최적화        | ✅ CPU/메모리 과다 프로비저닝 방지      | ✅ 불필요한 유휴 Pod 방지                      |
| 최적 사용 워크로드 | 스테이트풀, CPU/메모리 집중 앱 (DB, ML) | 웹 앱, 마이크로서비스, 스테이트리스 서비스     |
| 사용 사례          | MySQL, PostgreSQL, JVM 앱, AI/ML        | 웹 서버(Nginx, API), 메시지 큐, 마이크로서비스 |

- **VPA Recommender**: 과거 및 현재 리소스 사용량 데이터를 분석해 최적의 리소스 설정값을 계산
- **VPA Updater**: Pod를 업데이트하고 필요한 경우 재시작을 트리거
- **VPA Admission Controller**: 새로운 Pod 생성 시 리소스 요청을 자동으로 수정
- 모드
  - **Off**: 추천만 제공하고 자동 조정은 수행하지 않음
  - **Initial**: 새로 생성되는 Pod에만 리소스 설정을 적용함
  - **Auto**: 실행 중인 Pod도 자동으로 조정함 (Pod 재시작 필요)
  - **Recreate**: 리소스 변경 시 Pod를 재생성함

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: 내-앱-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: 내-앱
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: "*"
        minAllowed:
          cpu: 100m
          memory: 50Mi
        maxAllowed:
          cpu: 1
          memory: 500Mi
        controlledResources: ["cpu", "memory"]
```
