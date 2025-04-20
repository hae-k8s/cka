# Storage

## Docker Storage

- /var/lib/docker 폴더에 저장됨
- 계층 구조로 이뤄지고, 각 계층은 이전 계층에서의 변경점만 업데이트함
- image layer에는 읽기 전용 데이터가 쌓이고 - Read Only
- container layer에는 컨테이너 실행 중 쌓이는 데이터들이 저장됨(로그 등..) - Read & Write
- 컨테이너가 돌아가는 중 쌓이는 데이터들은 컨테이너가 제거되면 사라짐
- 이를 저장하기 위해서는 마운트를 해줘야 함 (볼륨 혹은 외부 디렉토리)

```
docker volume create data_volume
docker run -v data_volume:/var/lib/mysql mysql
docker run -v {volume_name}:{컨테이너 내부 디렉토리} {컨테이너 이름}

 // 다른 디렉토리에 마운트하고싶다면
docker run -v {디렉토리 경로}:{컨테이너 내부 디렉토리} {컨테이너 이름}

 // -v는 이전 방식이고, --mount를 사용하는 방식이 권장됨
docker run --mount type=bind, source=/data/mysql, target=/var/lib/mysql mysql
```

### Volume Driver

- 기본적으로 local 드라이버를 사용함
- /var/lib/docker/volumes 경로에 저장함
- 외부 드라이버를 사용할 수 있음
  - Azure File Storage
  - Convoy
  - Flocker
  - ...

```
docker run -it --name mysql --volume-driver {driver_name} --mount src={src}, target={target} {image}
```

## Container Storage Interface

- `Container Runtime Interface (CRI)` : 컨테이너 런타임 간 통신 표준
- `Container Network Interface (CNI)` : 네트워크 플러그인 표준
- `Container Storage Interface (CSI)` : 스토리지 플러그인 표준

## Volumes

```
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
	name: alpine
	command: ["/bin/sh","-c"]
	args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
	- mountPath: /opt
	  name: data-volume
  volumes:
  - name: data-volume
    hostPath:
	  path: /data
	  type: Directory
    // 클라우드 서비스 사용
    awsElasticBlockStore:
      volumeID: {volume_id}
      fsType: ext4
```

- 다중 노드 환경에서는 권장되지 않음 (노드 종속적이기 때문)
  - 공용 클라우드 사용

## Persistent Volumes

Cluster Wide Volume

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: /path/to/nfs
    server: nfs-server.example.com
  persistentVolumeReclaimPolicy: Retain // claim 삭제될 시 행위 (retain, delete, recycle)
```

## Persistent Volume Claims

- Volume Claim과 Volume은 1대1 관계임
- 남아있는 Volume이 없다면 Pending 상태가 됨.
- 매칭할때는 accessMode, 용량 등을 고려해 최적의 Volume을 매칭함

```
apiVersion:v1
kind:PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
	- ReadWriteOnce
  resources:
	requests:
	  storage: 500Mi
```

### PVC in Creating Pods

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

## Storage Class

- Static Provisioning: 클라우드에서 스토리지를 만들고 해당 스토리지에 매핑
- Dynamic Provisioning: 동적으로 생성

```
apiVersion: v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
```

- 수동으로 pv를 생성할 필요가 없게 됨
- pvc에서는 spec.storageClassName만 명시해주면 됨
