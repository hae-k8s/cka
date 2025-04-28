# Cluster Maintenance

## OS Upgrades

- Node가 죽는 경우 Pod도 접근 불가하게 됨
- Replica Set인 경우에는 다른 Node의 Pod로 접근 가능
- 아니라면 접근 불가한 상태가 됨.
- 아래 명령어로 Node를 죽이기 전, 후 조치를 할 수 있음

```
// 사용 불가, 존재하던 pod 다른 node로 이동
kubectl drain node-1
// 사용 불가, 존재하는 pod에는 영향 없음
kubectl cordon node-1
// 사용 불가 상태 해제
kubectl uncordon node-1
```

## Kubernetes Software Version

- x.x.x 방식
- 모든 component가 같은 버전인 것은 아니다
  - ETCD Cluster
    - CoreDNS
- 각기 다른 버전을 사용할 수도 있다
  - kube-apiserver 버전이 `x`일 경우
    - Controller-manager, kube-scheduler는 `x-1`
    - kubelet, kube-proxy는 `x-2`
    - kubectl은 `x+1` ~ `x-1`

### Upgrade Strategy

#### 한번에 업그레이드

Downtime 발생

#### 하나씩 업그레이드

하나씩 drain시키고 업그레이드

#### 업그레이드된 버전의 새 노드 생성

버전 올라간 노드 생성하고 pod 옮기기

### Upgrade Process

master node upgrade

```
// plan 조회
kubeadm upgrade plan
// kubeadm upgrade
apt-get upgrade -y kubeadm=1.12.0-00
kubeadm upgrade apply v1.12.0
// kubelet upgrade
apt-get upgrade -y kubelet=1.12.0-00
systemctl restart kubelet
```

worker node upgrade

```
kubectl drain node-1
kubeadm upgrade plan
apt-get upgrade -y kubeadm=1.12.0-00
apt-get upgrade -y kubelet=1.12.0-00
kubeadm upgrade node config --kubelet-version v1.12.0
systemctl restart kubelet
kubectl uncordon node-1
```

## Backup and Restore

### Resource Configs

정의 파일로 생성하는 방식이면 자동으로 가능
없으면 아래 커맨드로 가능

```
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

### ETCD Cluster

```
etcdctl snapshot save snapshot.db
service  kube-apiserver stop
etcdctl snapshot restore snapshot.db --data-dir {data-dir}
```

etct.service 내에 data-dir 다시 지정

```
systemctl daemon-reload
service etcd restart
service kube-apiserver start
```
