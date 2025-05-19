# Install Kubernetes the kubeadm way

## Introduction to Deployment with Kubeadm

1. Master | Worker Node 지정 
2. Container Runtime (containerd) 설치
3. kubeadm 도구 설치
4. Master Node 초기화
5. Network 설정
    - 일반 네트워크 연결은 충분하지 않음
    - 마스터와 워커 노드 간에 'POD 네트워크'라는 특별한 네트워킹 솔루션 필요
6. Worker Node Join
    - POD 네트워크가 설정되면 워커 노드를 마스터 노드에 연결

## Demo - Deployment with Kubeadm

- Master Node 초기화
    - 인증서 생성
    - 구성 파일 생성
    - Control Plane Component 시작
    
    `sudo kubeadm init --api-server-advertise-address=192.168.56.11 --pod-network-cidr=10.244.0.0/16 --upload-certs`
    
- kubeconfig 설정
    
    ```java
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
    
- Flannel CNI 플러그인 배포
    
    `kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml`
    
- Worker Node 추가 (Join)
    
    `sudo kubeadm join 192.168.56.11:6443 --token <토큰> --discovery-token-ca-cert-hash <해시>`
