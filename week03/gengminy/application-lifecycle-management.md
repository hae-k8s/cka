## Rolling Updates and Rollbacks

- Deployment 기본 전략은 Rolling Update (새 ReplicaSet 생성 및 순차적 교체)
- 롤아웃 상태 확인: `kubectl rollout status deployment/<deployment-name>`
- 롤아웃 히스토리 조회: `kubectl rollout history deployment/<deployment-name>`
- 롤백: `kubectl rollout undo deployment/<deployment-name>`

## Commands and Arguments

| Dockerfile | Kubernetes              | 설명                                |
| ---------- | ----------------------- | ----------------------------------- |
| ENTRYPOINT | spec.containers.command | 컨테이너 실행 시 호출되는 실행 파일 |
| CMD        | spec.containers.args    | ENTRYPOINT에 전달될 기본 인자       |

## Configure Environment Variables in Applications

- Inline 설정: `spec.containers.env` (name/value)

## Configure ConfigMaps in Applications

| 작업          | 명령어 / 필드                                                                                                     | 설명                                 |
| ------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| 생성 (리터럴) | `kubectl create configmap <name> --from-literal=<key>=<value>`                                                    | 단일 키-값 ConfigMap 생성            |
| 생성 (YAML)   | `kubectl apply -f <configmap.yaml>`                                                                               | 파일로부터 ConfigMap 생성            |
| 조회          | `kubectl get configmaps`                                                                                          | 클러스터 내 모든 ConfigMap 목록 출력 |
| 전체 매핑     | `spec.containers[].envFrom.configMapRef.name`                                                                     | 모든 키를 환경 변수로 주입           |
| 특정 키 매핑  | `spec.containers[].env[].valueFrom.configMapKeyRef.name`, `spec.containers[].env[].valueFrom.configMapKeyRef.key` | 지정 키만 환경 변수로 주입           |

## Configure Secrets in Applications

| 작업          | 명령어 / 필드                                                                             | 설명                              |
| ------------- | ----------------------------------------------------------------------------------------- | --------------------------------- |
| 생성 (리터럴) | `kubectl create secret generic <name> --from-literal=<key>=<value>`                       | 단일 키-값 Secret 생성            |
| 생성 (YAML)   | `kubectl apply -f <secret.yaml>`                                                          | 파일로부터 Secret 생성            |
| 조회          | `kubectl get secrets`                                                                     | 클러스터 내 모든 Secret 목록 출력 |
| 인코딩        | `echo -n '<text>' \| base64`                                                              | 평문 → Base64 인코딩              |
| 디코딩        | `echo '<base64>' \| base64 --decode`                                                      | Base64 → 평문 디코딩              |
| Env 주입      | `spec.containers[].envFrom[].secretRef.name`, `spec.containers[].envFrom[].secretRef.key` | Secret 키를 환경 변수로 주입      |
| Volume 마운트 | `spec.volumes[].secret.secretName`                                                        | Secret을 파일로 마운트            |

## Encrypting Secret Data at Rest

- secret 은 ETCD 에 평문으로 저장되어, 누구나 이 정보에 엑세스 할 수 있음
- ETCD 데이터 자체도 암호화하려면 `EncrytionConfiguration` 객체 생성
- `EncryptionConfiguration` 생성 후 API 서버에 `--encryption-provider-config=<file>` 플래그 추가

## Multi Container Pods

- `spec.containers[]`에 여러 컨테이너 정의 (동일 수명주기 공유)

## InitContainers

- `spec.initContainers[]`에 정의, Pod 생성 전 순차 실행
- 실패 시 Pod 재시작

## Horizontal Pod Autoscaling (HPA)

- 리소스 사용량 확인 : `kubectl top <object>`
- 수동: `kubectl scale deployment <name> --replicas=<count>`
- 자동: `kubectl autoscale deployment <name> --cpu-percent=<target> --min=<min-replica> --max=<max-replica>`
- 확인: `kubectl get hpa`

- deploy.yml 의 `spec.template.spec.containers[].resources.requests` 의 (memory/cpu) 임계치를 비교하여 오토스케일링
- `HorizontalProdAutoscaler` 객체 생성해서 지정도 가능

## In-place Resize of Pods

- pod resource 변경 시 기본 동작은 기존 pod 를 삭제하고 new pod 를 spin-up
- Kubernetes v1.33+에서 `InPlacePodVerticalScaling` 활성화 시 Pod 삭제 없이 CPU/Memory 조정 가능
- 이 기능은 CPU, Memory resources 변경에 대해서만 작동됨

## Vertical Pod Autoscaler (VPA)

- VPA 는 기본 제공되지 않아 별도 설치 필요
- 수동 : 설정 파일 변경 시 pod destroy & recreate
- `VPA Recommender` : 메트릭을 수집하고 최적의 값에 대한 권장 사항 제공. pod 를 직접 제어하지 않음.
- `VPA Updater` : VPA Recommender 에서 정보를 가져와 실제 pod 와 비교 후, 임계값을 초과하면 pod 제거.
- 'VPA Admission Controller` : VPA Recommender 의 권장 값을 적용하도록 pod 사양 변경
- `VerticalPodAutoscaler` 정의 파일로 생성 가능
- `spec.updatePolicy.updateMode` 지정으로 VPA 동작 방식 설정 : `Off` | `Initial` | `Recreate` | `Auto`
- In-place Resize 가 가능한 이후 버전에서는 `Auto` 모드가 기본 동작이며, 가장 선호될 것
- VPA 권장사항 확인 : `kubetctl describe vpa <name>`
- VPA 는 CPU 또는 Memory 를 많이 사용하는 Stateful Service 에 적합 (DB, JVM Based APP, AIML)
- HPA 는 빠른 확장이 필요한 Stateless Service 에 적합 (Web App, MicroService, API Server)
