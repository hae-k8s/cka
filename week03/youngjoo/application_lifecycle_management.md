# Application Lifecycle Management

## Rollout and Versioning

### Rollout Command

```
kubectl rollout status {deployment_name}
kubectl rollout history {deployment_name}
```

### Deployment Strategy

- 두가지 방식이 있음
  - 재생성 방식
    - 기존에 존재하던 모든 인스턴스 파괴 후 새 인스턴스 배포
    - 셧다운 시간이 생김
  - Rolling Update
    - 하나씩 파괴하고 재배포하는 방식
    - 셧다운 없음

![](https://velog.velcdn.com/images/0_zoo/post/18c2ed12-1b43-41ff-9f59-c787a73077dd/image.png)

- 한번에 모두 내리고 올리기 vs 하나씩 내리고 올리기

```
// 과정 보기
kubectl get replicasets
// rollout 취소
kubectl rollout undo {deployment_name}
```

## Commands & Arguments

- docker command는 pass

### Env Variables

Kubernetes

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
	containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
    	- containerPort: 8080
	  env:
		- name: APP_COLOR
		  value: pink
      // Configmap
      env:
		- name: APP_COLOR
		  valueFrom: configMapKeyRef
      // Secrets
      env:
		- name: APP_COLOR
		  value: secretKeyRef
```

#### ConfigMap

명령문으로 생성

```
kubectl create configmap {config-name} --from-literal={key}=value
// example
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MOD=prod
// 파일로부터 생성
kubectl create configmap {config-name} --from-file={path-to-file}
```

정의 생성

```
apiVersion: v1
kind: ConfigMap
metadata:
	name: app-config
data:
	APP_COLOR: blue
	APP_MODE: prod
```

조회

```
kubectl get configmaps
kubectl describe configmaps
```

pods에서 사용

```
spec:
	containers:
    	envFrom:
        	- configMapRef:
            	name: app-config
```

#### Secret

- 민감정보 저장에 사용

명령문으로 생성

```
kubectl create secret generic {secret-name} --from-literal={key}={value}

kubectl create secret generic {secret-name} --from-file={path-to-file}
```

정의 생성

```
apiVersion: v1
kind: Secret
metadata:
	name: app-secret
data: // encode
	DB_Host: bXlzcWw==
	DB_User: cm0vdA==
    DB_Password: cGFzd3Jk
```

```
kubectl create –f secret-data.yaml
```

조회

```
kubectl get secrets
kubectl describe secrets
kubectl get secret app-secret –o yaml
```

pods에서 사용

```
spec:
	containers:
    	envFrom:
        	- secretRef:
            	name: app-secret
        env:
          -name: DB_Password
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_Password
        volumes:
          - name: app-secret-volume
            secret:
	            secretName: app-secret
```

#### 주의할 점

- Secret은 암호화가 아닌 인코딩이다.
- 암호화를 위해서는 EncryptionConfiguration을 사용할 수 있다.
- rbac을 활용해서 접근 통제해라.
- AWS Provider와 같은 third party solution을 사용해보자.

## Multi Container Pods

- 여러개 서비스가 동시에 필요할 수 있다
- 함께 생성되고 함께 파괴된다

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
  	name: simple-webapp
spec:
	containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
      	- containerPort: 8080
	- name: log-agent
      image: log-agent
```

Design Pattern이 있긴 한데 CKAD 내용이라 pass

## InitContainer

- 컨테이너 시작 시 실행할 구문을 지정할 수 있다.
- 어플리케이션 구동 전에 실행된다.
- 하나당 한번씩 순서대로 실행된다.

```
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
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

## Autoscaling

### Verticle Scaling

- 서버 자체를 키우는 방식

### Horizontal Scaling

- 서버 개수를 늘리는 방식

## Autoscaling in Kubernetes

### Scaling Cluster Infra

#### Verticle Scaling

클러스터 자체를 키우는 방식

- 일반적으로 사용되는 방식은 아님

#### Horizontal Scaling

클러스터 개수를 늘리는 방식

```
kubeadm join ...
```

- Cluster Autoscailer

### Scaling Workloads

#### Verticle Scaling

pod 크기를 키우는 방식

```
kubectl edit ...
```

- Vertical Pod Autoscaler(VPA)

#### Horizontal Scaling

pod 개수를 늘리는 방식

```
kubectl scale ...
```

#### Horizontal Pod Autoscaler(HPA)

기본적으로는 아래 명령어로 scale out할 수 있음

```
kubectl scale deployment my-app --replicas=3
```

오토스케일링 설정

```
// autoscale 설정
kubectl autoscale deployment myapp --cpu-percent=50 --min=1 --max=10
// 조회
kubectl get hpa
// 삭제
kubectl delete hpa myapp
```

- pod 정의 내 resources.limits를 읽고 pod를 지속적으로 모니터링함

정의하는 방식으로 생성

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
	name: my-app-hpa
spec:
	scaleTargetRef:
    	apiVersion: apps/v1
        kind: Deployment
        name: my-app
    minReplicas: 1
    maxReplicas: 10
    metrics:
    - type: Resource
      resources:
      	name: cpu
        target:
        	type: Utilization
            averageUtilization: 50 // target utilization
```

#### In-place Resize of Pod Resources

pod 정의에서 resource를 변경할 경우 제거 후 생성된다. (제자리 변경 X)
1.27부터 제자리 변경이 가능해짐

Feature flag 설정

```
FEATURE_GATES=InPlacePodVerticalScaling=true
```

pod 정의

```
...
spec:
	containers:
    	resizePolicy:
        	- resourceName: cpu
              restartPolicy: NotRequired
            - resourceName: memory
              restartPolicy: RestartContainer
```

제한 사항

- cpu와 메모리만 변경 가능함
- resource requests와 limits가 한번 설정되면 제거 불가능함

#### Verticle Pod Autoscaling(VPA)

기본적으로는 아래 명령어로 scale up 가능

```
kubectl edit deployment myapp
```

다운로드받아서 사용해야 함

```
kubectl apply -f "https://github.com/kubernetest/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml"

// 조회
kubectl get pods -n kube-system | grep vpa
```

VPA Recommender

- 정보를 수집해서 업데이트할 pod 추천

VPA Updater

- Recommender로부터 추천을 받고 실제 지표 확인 후 임계값 넘은 pod 퇴출

VPA Admission Controller

- Updater가 pod를 퇴출할 때 새 리소스 견적을 짜서 pod 생성

VPA 정의

```
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       my-app
  updatePolicy:
    updateMode: "Auto" // Off, Initial, Recreate, Auto
  resourcePolicy:
    containerPolicies:
    - containerName: "my-app"
      minAllowed:
        cpu: "250m"
      maxAllowed:
        cpu: "2"
      controlledResources: ["cpu"]
```

updateMode

- Off
  - 추천만 하고 아무 작업 안함
- Initial
  - pod 생성시에만 작동
- Recreate
  - 제한 범위 넘어가면 퇴출
- Auto
  - Recreate와 동일하게 작동
    - 제자리 변경이 활성화되면 Recreate보다 선호

Recommendation 조회

```
kubectl describe vpa my-app-vpa
```
