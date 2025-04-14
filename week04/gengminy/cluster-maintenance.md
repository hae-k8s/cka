## OS Upgrade

- Pod는 일정 시간 동안 타임아웃되면 퇴출됨 (기본 5분)
- `kubectl drain <node>` : node 내 Pod를 다른 node로 이동시킴
- `kubectl cordon <node>` : node 스케줄링 비활성화 (새로운 앱이 배포되는 것을 차단)
- `kubectl uncordon <node>` : node 스케줄링 활성화 (새로운 앱이 배포되는 것을 허용)

## Cluster Upgrade Process

- `Control Plane` 구성 요소들은 각기 다른 버전을 가질 수 있음
- 하지만 `kube-apiserver`는 주요 구성 요소이므로 `kube-apiserver`보`다 높은 버전일 수 없음
- `kubectl`은 `kube-apiserver`보다 높은 버전일 수 있음
- 업그레이드는 최신 버전으로 점프하지 말고 Minor 버전을 1단계씩 높이는 것을 권장함
- 업그레이드 순서는 master node를 먼저 업그레이드한 후 worker node 순으로 진행하며, Pod 내 애플리케이션은 계속 동작함
- 업그레이드 정보 확인: `kubeadm update plan`
- 버전 업그레이드: `kubeadm upgrade apply <version>`
- Kubernetes 업데이트 시, pkgs.k8s.io를 사용하여 apt repository를 업데이트해야 함

## Backup and Restore Method

- 선언적으로 정의된 리소스를 Git으로 관리하면 언제든 재배치가 가능함
- 더 나은 방법은 `kube-apiserver`에서 쿼리 명령어를 사용하여 모든 리소스를 백업하는 것임
- 또는 ETCD의 `data-dir`에 접근하여 `snapshot.db` 파일로 스냅샷을 캡처 및 복구할 수 있음
