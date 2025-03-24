## Monitor Cluster Components

- `Heapster` (Deprecated) / `Metrics Server`
- Cluster 하나당 Metrics Server 1대
- `cAdvisor` 는 kubelet 내부에서 Container Metric 을 수집하고 kubelet API 로 노출
- 리소스 사용량 확인 : `kubectl top <object>`

## Application Logs

- pod log 확인 : `kubectl logs <pod-name>`
- 다중 컨테이너 pod log : `kubectl logs <pod-name> -c <container-name>`
- 실시간 log 확인 : `kubectl logs -f <pod-name> [-c <container-name>]`
