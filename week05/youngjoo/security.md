## Certificate API

관리자가 아닌 유저를 인증하기 위한 방법

유저

```
// 키 생성
openssl genrsa -out jane.key 2048
// 인증서 요청 생성
openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr
```

관리자

SigningRequest 생성

```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
name: jane
spec:
groups:
- system:authenticated
usages:
- digital signature
- key encipherment
- server auth
request: {유저 key | base64}
```

Review & Share

```
// 조회
kubectl get csr
// approve
kubectl certificate approve jane
// reject
kubectl certificate deny jane
// delete
kubectl delete csr jane
// extract
kubectl get csr jane -o yaml
// decode
echo “LS0…Qo=” | base64 --decode
```

controller manager cert 설정

```
cat /etc/kubernetes/manifests/kube-controller-manager.yaml

spec:
containers:
- command:
- kube-controller-manager
- --address=127.0.0.1
===================================================
- --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
- --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
===================================================
- --controllers=*,bootstrapsigner,tokencleaner
- --kubeconfig=/etc/kubernetes/controller-manager.conf
- --leader-elect=true
- --root-ca-file=/etc/kubernetes/pki/ca.crt
- --service-account-private-key-file=/etc/kubernetes/pki/sa.key
- --use-service-account-credentials=true
```

## KubeConfig

인증을 하려면 아래 명령어로 지정해줘야 하는데, 귀찮음

```
kubectl get pods
--server my-kube-playground:6443
--client-key admin.key
--client-certificate admin.crt
--certificate-authority ca.crt
```

kubeconfig 파일에 명시하면 자동으로 주입됨

`HOME/.kube/config`

```
--server my-kube-playground:6443
--client-key admin.key
--client-certificate admin.crt
--certificate-authority ca.crt
```

kubeconfig는 세가지 부분으로 이뤄짐

#### Clusters

접근할 클러스터

#### Users

접근할 유저

#### Contexts

유저와 클러스터 조합

```
apiVersion: v1
kind: Config
current-context: my-kube-admin@my-kube-playground // default context

clusters:
- name: my-kube-playground
  cluster:
    certificate-authority[-data(데이터 자체 제공)]: ca.crt
	server: https://my-kube-playground:6443

contexts:
  - name: my-kube-admin@my-kube-playground
  context:
	cluster: my-kube-playground
	user: my-kube-admin
    namespace: finance // 이 context로 전환하면 자동으로 namespace에 들어감

users:
- name: my-kube-admin
  user:
	client-certificate: admin.crt // 전체 경로 이용하는게 나음
	client-key: admin.key
```

```
kubectl config view
// context 변경, kubeconfig에 반영됨
kubectl config use-context prod-user@production
```

## API Groups

![](https://velog.velcdn.com/images/1124750/post/04100f66-ac98-4715-9ec9-e480f1d73d09/image.png)
![](https://velog.velcdn.com/images/1124750/post/88460116-43ff-45ef-80b3-d4b2f4e80a43/image.png)

- Core, Named API group이 있다.
- 각 api group 하위에는 resource, 그 하위에는 Verb(동사)가 있다
- 이를 기반으로 접근 허용/불가를 관리할 수 있다.

## Authorization

### Node

Kubelet이 api server에 접근을 요청하면 Node Authorizer가 처리함

### ABAC(Attribute based)

```
 {"kind": "Policy", "spec": {"user": “dev-user", "namespace": "*", "resource": “pods", "apiGroup": "*"}}
```

와 같은 형태의 json을 매번 수정하고 kube api server를 다시 시작해야 함
관리하기 힘듦

### RBAC(Role based)

역할별 가능한 작업을 정의하고, 해당 역할에 유저를 연결

### Webhook

외부에서 권한 관리
kube api server에 요청 -> 외부 서비스에 질문 -> 답변 받아와서 처리

### AlwaysAllow, AlwaysDeny

- `authorization-mode` 설정으로 가능
- 지정하지 않으면 기본적으로 `AlwaysAllow`
- 여러개를 설정할 경우 모듈 하나씩 검증을 진행하고 승인되면 approve

## RBAC

### Role

```
 apiVersion: rbac.authorization.k8s.io/v1
 kind: Role
 metadata:
   name: developer
 rules:
 - apiGroups: [""]
   resources: ["pods"]
   verbs: ["list“, "get", “create“, “update“, “delete"]
   resourceNames: ["blue", "green"] // 특정 리소스에만 접근 가능
 - apiGroups: [""]
   resources: [“ConfigMap"]
   verbs: [“create“]
```

### RoleBinding

```
apiVersion: rbac.authorization.k8s.io/v1
 kind: RoleBinding
 metadata:
   name: devuser-developer-binding
 subjects:
 -kind: User
   name: dev-user
   apiGroup: rbac.authorization.k8s.io
 roleRef:
   kind: Role
   name: developer
   apiGroup: rbac.authorization.k8s.io
```

### 조회

```
k get roles
k get rolebindings
k describe role developer
```

#### Check Access

```
k auth can-i {verb} {resources} [as {user}] [--namespace {namespace}]
```

## Cluster Roles and Role Bindings

#### Namespace

```
// namespace에 속할 수 있거나, 불가능한 리소스 목록 볼 수 있음
k api-resources --namespaced=[true/false]
```

### Cluster Roles

```
apiVersion: rbac.authorization.k8s.io/v1
 kind: ClusterRole
 metadata:
   name: cluster-administrator
 rules:
 - apiGroups: [""]
   resources: [“nodes"]
   verbs: ["list“, "get", “create“, “delete"]
```

### Cluster Role Binding

```
apiVersion: rbac.authorization.k8s.io/v1
 kind: ClusterRoleBinding
 metadata:
   name: cluster-admin-role-binding
 subjects:
 - kind: User
   name: cluster-admin
   apiGroup: rbac.authorization.k8s.io
 roleRef:
   kind: ClusterRole
   name: cluster-administrator
   apiGroup: rbac.authorization.k8s.io
```

## Service Account

#### User vs Service Account

- User는 유저가 사용
- Service Account는 기계가 사용
- ex) Prometheus, Jenkins, ...

### Create Service Account

```
k create serviceaccount my-service-account
```

- service account를 생성하면 토큰이 자동으로 생성됨
- 해당 토큰을 Bearer 헤더에 넣어서 전송하면 api call 가능

### 기본 Service Account

- namespace 생성되면 기본적으로 service account를 생성하고 토큰을 마운트한다.
- `spec.automountServiceAccountToken`=false로 지정하면 해제 가능하다.
- `spec.serviceAccountName` 속성을 통해 serviceAccount 지정 가능하다.

### 1.22 변경사항

- 기존에 생성되던 기본 토큰의 유효시간이 무제한인 부분이 문제가 됨
- namespace 기본 service account 토큰에 의존하는 것이 아닌,
- token request api를 통해 토큰을 만드는 방식으로 변경

### 1.24 변경사항

- service account 생성 시 기본적으로 토큰을 생성하지 않음
- 아래 명령어를 통해 직접 생성해야 함
- 기본적으로 유효기간은 1시간

```
k create token dashboard-sa
```

- 정의 파일을 통해 유효기간이 없는 토큰을 만들 수 있긴 한데 지양해야 함
- token request api를 사용하지 못하는 경우에만 이 방식으로 생성하라고 권고(강요..?)함

## Image Security

```
 kubectl create secret docker-registry regcred \
 --docker-server= private-registry.io \
 --docker-username= registry-user \
 --docker-password= registry-password \
 --docker-email= registry-user@org.com
```

nginx-pod.yaml

```
spec:
   imagePullSecrets:
   - name: regcred
```

## Security in Docker

- 이미지를 올릴 때 이미지에 user를 정의하던가, 이미지 실행 시 지정하는 방식으로 user 설정이 가능함
- host의 root와 container에서의 root는 다름 (host의 root는 너무 강력하기 때문)
- /usr/include/linux/capability.h에서 조회 가능

## Security Contexts

- pod level, container level에서 설정 가능
- 둘 다 설정하면 container level이 overwrite함
- 아래 속성을 spec 하위 혹은 containers 하위에 넣어서 제어 가능

```
securityContext:
  runAsUser: 1000
  capabilities:
    add: ["MAC_ADMIN"]
```

## Network Policies

- 기본적으로 모든 pod는 서로 통신할 수 있음
- 웹 서버와 db 서버처럼 서로 통신을 하지 못하도록 제한을 해야 하는 경우가 있음

```
apiVersion: networking.k8s.io/v1
 kind: NetworkPolicy
 metadata:
 name: db-policy
 spec:
   podSelector:
     matchLabels:
       role: db

   policyTypes:
   - Ingress
   - Egress
   ingress:
   - from: // -를 붙이면 각각 rule이 되고, 안붙이면 하나의 rule이 됨
     - podSelector:
 	 	matchLabels:
 		  name: api-pod
     - namespaceSelector:
         matchLabels:
           name: prod
     - ipBlock:
       cidr: 192.168.5.10/32
     ports:
     - protocol: TCP
 	   port: 3306
    egress:
    - to:
      - ipBlock:
          cidr: 192.168.5.10/32
      ports:
      - protocol: TCP
        port: 80
```

- Network Policy를 지원하지 않는 솔루션도 있음(ex. Flannel)

## Custom Resource Definition

- custom resource를 사용하기 위해서는 CRD와 custom controller가 필요함

CRD 예시

```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
      - ct
```

### Custom Controller

- github에 Sample Controler가 있음
- 클론해서 로직 변경해서 사용하면 됨

### Operator Framework

operator.io에서 프로메테우스, argoCD, Grafana 등 인기있는 Operator 설치 지침을 확인할 수 있음
