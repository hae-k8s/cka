### Security Primitives

- 기본적으로 `kube-apiserver` 접근 제어가 1차 보안임
- 같은 클러스터 내 통신은 기본적으로 허용됨

### Authentication

- Kubernetes는 사용자 계정을 지원하지 않으며, 서비스 계정만 생성할 수 있음
- `kube-apiserver` 보안 관리 정책: `Static Password File` | `Static Token File` | `Certificates` | `Identity Services (LDAP, Kerberos)`
- **Static Password File**: CSV 파일 생성 후 `kube-apiserver` 설정에서 `--basic-auth-file=<filename>`으로 등록 (권장하지 않음)
- **Static Token File**: CSV 파일에 password 대신 token을 설정 (권장하지 않음)

### TLS Basics

- 서버 간 통신 암호화를 위해 사용됨
- **대칭키 암호화**: 암호키 복사본과 데이터를 함께 전송 → 암호키가 노출되면 위험함
- **비대칭키 암호화**: private key와 public key를 모두 사용하며, 암호키를 함께 암호화하여 전송하므로 더 안전함
- 비대칭키 암호화 시, 공인 기관(CA)이 서명한 인증서를 함께 전송하여 인증함
- Public Key 또는 Certificates: `*.crt`, `*.pem` 확장자
- Private Key: `*.key`, `*-key.pem` 확장자 또는 key라는 단어가 포함된 PEM 파일

### TLS in Kubernetes

Client Certificates for Clients

- 어드민 유저 → `kube-apiserver` (kubectl REST API)
- `kube-scheduler` → `kube-apiserver` (kubectl REST API)
- `kube-controller-manager` → `kube-apiserver`
- `kube-proxy` → `kube-apiserver`

Server Certificates for Servers

- `kube-apiserver` → `etcd server`
- `kube-apiserver` → `kubelet server`
- `kubelet server (node)` → `kube-apiserver`

### TLS in Kubernetes - Certificate Creation

- 키 생성: `openssl genrsa -out <key-name> 2048`
- 인증서 생성: `openssl req -new -key <key-name> -subj "/CN=KUBERNETES-CA/O=system:<group>" -out <certificate-name>`
- CA 서명: `openssl x509 -req -in <certificate-name> -signkey ca.key -out <ca-certificate-name>`
- `kube-apiserver`의 인증서를 생성할 때는 DNS 및 별칭 정보를 모두 포함시켜야 함

### View Certificate Details

- 인증서를 디코딩하여 확인: `openssl x509 -in <crt-file> -text -noout`
- 발행인과 만료일을 확인해야 함
