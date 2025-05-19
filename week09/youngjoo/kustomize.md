# Kustomize

## 필요성

환경별 배포가 필요할 경우

- dev, stage, prd로 디렉토리 구분 후
- kubectl apply -f dev/와 같은 방식으로 배포 가능함
- 하지만 이게 이상적인 방식은 아님
  - 환경별로 동일한 파일을 모두 가져야 하기 때문

## Kustomize

- base, overlay 디렉토리로 분리해서 관리
- base는 기본이 되는 부분
- overlay는 환경별로 관리
  - base중 변경을 원하는 부분을 override하는 형태로 관리
- built in이라서 별도 설치 필요 없음

## Kustomize vs Helm

- Helm은 단순한 환경 관리 툴을 넘어선 패키지 관리자임
- conditional, hook, loop와 같은 복잡한 기능들을 제공함
- 템플릿을 활용하기 때문에 유효하지 않은 yaml 파일인 경우가 많고, 이게 복잡해지면 읽기 힘듦

## Kustomization.yaml

- 두가지 부분으로 구분됨
  - kustomize를 통해 관리돼야 하는 resource
  - 행해져야 하는 customizations

```
# kubernetes resources to be managed by kustomize
resource:
  - nginx-deployment.yaml
  - nginx-service.yaml
# Customizations that need to be made
commonLabels:
  company: KodeKloud
```

- `kustomize build k8s/` 명령어를 통해 빌드 가능
- 빌드를 한다고 적용되는 것이 아니라, 결과값만 출력됨
- 실제 적용을 위해서는 위 결과를 redirect해서 kubectl apply로 적용시켜야 함
  - `kubectl build k8s/ | kubectl apply -f -`
  - `kubectl apply -k k8s/`
- 삭제도 동일한 방식으로 가능
  - `kubectl delete -k k8s/`

### apiversion & kind

- `apiVersion: kustomize.config.k8s.io/v1beta1`
- `kind: Kustomization`

같은 방식으로 적을 순 있는데, 안적어도 상관 없음. 언제 변화가 생길 지 모르니 적는걸 권장

## Managing Directories

- root와 서브모듈로 분리
- root kustomization.yaml의 resource에는 서브모듈 경로를 넣어두고 서브모듈의 kustomization.yaml의 resource에서 각 모듈에 해당하는 리소스 관리

## Transformers

### Common Transformers

- `commonLabel`: 레이블 추가
- `namePrefix/Suffix`: 이름 suffix/prefix 추가
- `Namespace`: 네임스페이스 추가
- `commonAnnotations`: 어노테이션 추가

### Image Transformers

```
images:
  - name: nginx
  - newName: haproxy
  - newTag: 2.4
```

- container name이 아닌, container image name과 비교해 해당하는 image를 변경함

## Patches

- Transformer처럼 공통적인 부분이 아니라 특정 조건을 만족하는 리소스에 있어서 변화를 주고 싶다면 patch를 사용해야 함
- 3가지 부분으로 구성됨
  - operation type: 행위(add, replace, remove)
  - target: 변경 대상
  - value: 변경하고자 하는 값

Json 6902 Patch

```
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /metadata/name
        value: web-deployment
```

Strategic merge patch

```
patches:
  - patch: |-
      apiversion: apps/v1
      kind: Deployment
      metadata:
        name: api-deployment
      spec:
        replicas: 5
```

Separate File

- 기존 patch 부분을 `path: replica-patch.yaml`로 명시하고 해당 파일에서 patch 내용 작성하는 방식
- patch 내용이 많으면 가독성 좋아짐

### Patch List

List를 변경하거나 추가하거나 삭제하고 싶을 때
path 값을 통해 수정할 수 있음

- `/spec/.../0`: 인덱스 접근
- `/spec/.../-`: 마지막 위치
- Strategic Merge Patch로 할 경우 add는 원래 방식으로 하면 되지만 delete는 `$patch:delete`를 명시해줘야 함
