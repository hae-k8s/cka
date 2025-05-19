# Troubleshooting

## Application Failure

1. DB Host 에 작성된 Service name이 잘못됨
    
    ```bash
    k config --help 
    k config set-context --current --namespace=alpha
    k get pods
    k get deploy
    k get svc // IP 확인
    
    curl http://localhost:30081
    
    k describe deploy webapp-mysql
    k edit svc mysql
    
    (name 수정 후 저장)
    
    k delete svc mysql // ClusterIP 가 중복되기 때문에 삭제 필요
    k replace -f /tmp/kubectl-edit-2052310705.yaml --force
    
    ```
    
2. mysql-service의 endpoints가 3306 이 아닌, 8080을 가리키고 있음
    
    ```bash
    k get svc
    k describe svc mysql-service // service의 엔드포인트 확인
    k get pods -o wide // 실제 mysql Pod의 IP 확인
    
    (IP는 맞았으나, Diagram상의 3306과 8080이 일치하지 않음)
    
    k edit svc mysql-service
    
    (targetPort 수정)
    ```
    
3. mysql-service의 endpoint가 설정되어있지 않음
    
    왜인지 확인해보니, service의 selecter 가 `name=sql00001` 로 되어있음.
    
    하지만, mysql Pod의 name은 `mysql` 이기에 감지를 못한 것.
    
    `k edit service mysql-service` + Selector 수정
    
4. deploy 의 mysql-user가 잘못되어있었음 → root로 변경
5. mysql-user를 root로 변경, password 변경
6. service 의 Port 가 30081 이 아니라, 30088로 되어있음

## Control Plane Failure

[kubectl 치트 시트](https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/)

- describe Pod → Node가 none
    
    → 파드를 Node에 배치하는건 스케줄러의 역할
    
    → 스케줄러 상태 확인
    
    static Pod 라서 파일 수정
    
    `vi /etc/kubernetes/manifests/kube-scheduler.yaml` 
    

- `k scale deploy app —replicas=2`
- `k logs kube-controller-manager-controlplane -n kube-system`
- `vi /etc/kubernetes/mainfests/kube-controller-manager.yaml`

- `k logs kube-controller-manager-controlplane -n kube-system`
    
    `open /etc/kubernetes/pki/ca.crt: no such file or directory`
    
    `cat /etc/kubernetes/pki/ca.crt`
    
- `vi /etc/kubernetes/mainfests/kube-controller-manager.yaml`
    
    ```bash
    - hostPath:
        path: /etc/kubernetes/~~WRONG-PKI-DIRECTORY~~ -> pki
        type: DirectoryOrCreate
      name: kubeconfig
    ```
    
    k get pods -n kube-system `—-watch`
    

## Worker Node Failure

1. `kubectl get nodes` 명령어로 노드들이 Ready 상태인지 확인
2. Not Ready 상태라면 `kubectl describe node` 명령어로 세부사항 확인
    - OutOfDisk: ****디스크 공간 부족
    - MemoryPressure: 메모리 부족
    - DiskPressure**:** 디스크 용량 부족
    - PIDPressure: 프로세스 개수 초과
    - Ready**:** 노드가 정상 상태
- 마스터와 통신 중단 시
    - 노드가 크래시되면 상태가 `unknown`으로 변경됨
    - `Last Heartbeat Time` 필드를 확인해서 노드 크래시 시점 파악 가능