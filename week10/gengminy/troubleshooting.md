# Troubleshooting

## Application Failure

- 서버 접근 확인
  - `curl -I http://<SERVICE_URL>:<PORT>` 로 HTTP 응답 상태 코드 확인
- Node Selector 설정 확인
  - Deployment/Pod 스펙에서 `spec.nodeSelector` 확인
- Pod 상태 확인
  - `kubectl get pods -n <NAMESPACE>`
  - `kubectl describe pod <POD_NAME> -n <NAMESPACE>`
- Pod 로그 확인
  - `kubectl logs <POD_NAME> -n <NAMESPACE>`
  - 옵션: `-f` (실시간 로그), `--previous` (이전 인스턴스 로그)
- 데이터베이스 Pod 확인
  - `kubectl get pods -l app=db -n <NAMESPACE>`

## Control Plane Failure

- Node 및 Pod 상태 확인
  - `kubectl get nodes`
  - `kubectl get pods -n kube-system`
- kube-system 네임스페이스 Pod 상세 확인
  - `kubectl get pods -n kube-system -o wide`
- Control Plane 컴포넌트 상태 확인
  - Master 노드에서 `docker ps | grep kube-` 또는  
    `kubectl get pods -n kube-system -l component=kube-`
- kube-apiserver 로그 확인
  - `sudo journalctl -u kube-apiserver -f`

## Worker Node Failure

- 워커 노드 상태 확인
  - `kubectl get nodes`
- 노드 리소스·시스템 로그 확인
  - SSH 접속 후 `top`, `df -h`, `dmesg`, `journalctl -xe`
- kubelet 서비스 상태 확인
  - `sudo systemctl status kubelet`
- kubelet 인증서 유효성 확인
  - `sudo kubeadm certs check-expiration`
