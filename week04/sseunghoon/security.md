# Security

## Kubernetes Security Primitives

kube api-server에 대한 보안을 위해서는

- 누가 접근할 수 있고
- 무엇을 할 수 있는가

를 정해야 함

### Who can access

- Username and Passwords
- Username and Tokens
- Certificates
- External Authentication Providers
- Service Accounts

### What they can do

- RBAC Authorization
- ABAC Authorization
- Node Authorization
- Webhook Mode

### TLS Certificates

컴포넌트 간 통신의 보안은 TLS 인증서를 통해 이뤄짐

### Network Policies

Pod간 통신 보안은 네트워크 설정을 통해 제어 가능함

## Authentication

### Basic

- `password`, `username`, `userid` 로 구성된 csv 파일 생성
  - password 대신 token 넣으면 토큰 파일 생성 가능
    - 4번째 칼럼에 group 지정 가능
- `kube-apiserver.service` 파일에
- `--basic-auth-file={파일 이름}` 추가
  - `--token-auth-file`로 하면 토큰 파일도 가능

## TLS

### 대칭 암호화와 비대칭 암호화

#### 대칭 암호화

- 정보 전송 시 데이터를 암호화
- 암호화하는 키도 함께 전송

#### 비대칭 암호화

- 개인 키, 공용 자물쇠를 만듦
- 보안이 팔요한 서버를 공용 자물쇠로 잠그고 개인 키로 여는 형태
- 여러 서버를 하나의 개인 키로 관리 가능함

#### 이를 이용해 클라와 서버가 소통하는 방법

- 첫 접속 시 서버에서 공용 키를 내려줌
- 클라에서 생성한 대칭 키를 공용 키를 이용해 암호화해서 보냄
- 서버에서는 이를 받아서 개인키로 해독하면 대칭 키를 모두 갖게 됨

#### 문제점

서버에서 받아온 공개 키가 내가 원하는 서버의 공개 키인지 확인이 필요함

### 인증서

- 공인된 CA에서 인증받은(서명받은) 키를 사용하면 안전하게 통신 가능함

### TLS in Kubernetes

- 컴포넌트들은 각각 키 페어를 가짐
- 특정 컴포넌트와 통신하기 위한 키 페어를 둘 수도 있음
- CA를 하나 이상 가져야 함

### Creating Certificates

- CA 키페어를 생성할 때는 CA 자체로 서명
- 다른 컴포넌트의 경우에는 CA를 사용해서 서명
- openssl.cnf 파일에서 [alt_names] 설정해서 별칭 설정 가능

```
// Generate Key
openssl genrsa-out ca.key 2048
// Certificate Signing Request
openssl req -new -key ca.key-subj  "/CN=KUBERNETES-CA" -out ca.csr
// Sign Certificates
openssl x509 -req -in ca.csr-signkey ca.key-out ca.crt
```

### View Certificates

```
// apiserver 정보 조회
cat /etc/kubernetes/manifests/kube-apiserver.yaml
// 인증서 정보 조회
openssl x509 -in {인증서 경로} -text -noout
```
