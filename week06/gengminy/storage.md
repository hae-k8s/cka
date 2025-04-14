## Storage in Docker

- Docker는 기본적으로 `/var/lib/docker`에 데이터를 저장
- 레이어드 아키텍처로 구성되어 변경되지 않은 요소를 캐시에서 가져와 빌드 속도 향상
- **이미지** 내부 데이터는 `read-only`, 컨테이너는 최상위 쓰기 계층(`rw layer`)이 추가되어 일시적으로 생성/삭제됨
- 데이터를 영구 유지하려면 `volume`을 마운트해야 함
- `-v` 또는 `--volume` (Named Volume) : `볼륨명:컨테이너경로` 형식으로 지정하면 `/var/lib/docker/volumes`에 볼륨 생성 후 마운트
- `-v` 또는 `--volume` (Bind Mount) : `호스트디렉토리:컨테이너경로` 형식으로 지정하면 호스트 경로를 마운트
- `--mount` (New Mount Syntax) : `type`, `source`, `target`, `readonly`, `consistency` 등의 매개변수 명시
- Docker는 OS 및 커널에 따라 기본 Storage Driver를 자동으로 선택 (예: Linux의 경우 `overlay2`, Windows의 경우 `windowsfilter` 등)

## Volume Driver Plugins in Docker

- 볼륨은 `Storage Driver`가 아닌 `Volume Driver Plugin`이 처리 (예: `local`, `nfs`, `azurefile`, `flocker` 등)

## Container Storage Interface (CSI)

- `Container Runtime Interface (CRI)` : 컨테이너 런타임 간 통신 표준
- `Container Network Interface (CNI)` : 네트워크 플러그인 표준
- `Container Storage Interface (CSI)` : 스토리지 플러그인 표준

## Volumes (Kubernetes)

- Docker 컨테이너와 마찬가지로 Pod는 일시적으로 생성/파괴 ⇒ 데이터를 유지하려면 볼륨 설정 필요
- `spec.containers[].volumeMounts[].mountPath` : 컨테이너 내부 마운트 경로
- `spec.containers[].volumeMounts[].name` : 참조할 볼륨 이름
- `spec.volumes[].name` : 볼륨 이름 정의
- `spec.volumes[].hostPath.path` : 호스트 경로 지정
- `spec.volumes[].hostPath.type` : `Directory`, `File`, `Socket`, `DirectoryOrCreate` 등 호스트 패스 타입
- HostPath 볼륨은 Node 종속적이므로 다중 Node 환경에서는 권장되지 않음
- 다중 Node에서는 외부 스토리지 솔루션 사용 권장

## Persistent Volume (PV)

- 다중 Node 환경에서는 중앙 스토리지로 `PersistentVolume` 리소스를 사용
- 외부 스토리지(예: NFS, iSCSI, 클라우드 볼륨 등)를 연결
- `PersistentVolume` 리소스를 정의하여 생성

## Persistent Volume Claim (PVC)

- `PersistentVolumeClaim` 리소스를 정의하여 사용자가 스토리지 요청
- PV에는 `spec.persistentVolumeReclaimPolicy` 설정 (Retain, Delete, Recycle)으로 PVC 해제 시 동작 결정
- PVC는 조건에 맞는 PV가 없으면 `Pending` 상태로 대기
- Pod에서 사용하려면 `spec.volumes[].persistentVolumeClaim.claimName`에 PVC 이름 지정

## StorageClass

- Static Provisioning : 관리자가 미리 PV를 생성하여 클러스터에 제공
- Dynamic Provisioning : `StorageClass`를 지정하여 PVC 생성 시 자동으로 PV를 프로비저닝
