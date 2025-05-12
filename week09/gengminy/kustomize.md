## Kustomize Problem Statement & idealogy

- 환경별로 배포 YAML 파일을 다르게 관리해야 할 때, 이를 복사하여 사용하는 것은 비효율적임
- `base`: 모든 환경에서 동일하게 적용될 구성. 기본 구성을 정의하고, 이를 환경별로 오버레이하여 덮어쓸 수 있음
- `overlay`: 환경별로 다른 설정을 지정. 기본 구성에서 덮어쓸 속성 또는 매개변수를 정의
- `apps/base`에 공통 설정을 배치하고, `apps/overlays/{name}`에 환경별 오버레이를 배치
- Helm Chart와 달리 템플릿 언어를 별도 학습할 필요가 없어 러닝 커브가 낮음

## Kustomize vs Helm

- `Helm`은 Go 템플릿을 사용하여 `templates/` 디렉터리에 정의된 매니페스트에 변수를 삽입하고, `values.yaml`에 값을 제공
- 기본 값은 `values.yaml`에 정의하고, 환경별로는 `values-dev.yaml`, `values-prod.yaml` 등 별도 파일을 사용
- `Helm`은 패키지 매니저 역할을 하며, 릴리즈 관리, 의존성 처리, Hook, 차트 패키징 등의 다양한 기능 제공
- Go 템플릿 구문이 복잡해 가독성이 떨어질 수 있음

## kustomization.yaml file

- Kustomize를 실행하려면 각 디렉토리에 `kustomization.yaml` 파일을 생성해야 함
- `resources`: Kustomize가 관리할 리소스 파일 목록을 나열
- `commonLabels`: 모든 리소스에 공통으로 적용할 metadata `labels`를 지정
- `commonAnnotations`: 모든 리소스에 공통으로 적용할 metadata `annotations`를 지정
- `images`: 이미지 이름·태그 일괄 변경에 사용
- `kustomize build <디렉토리>`: 매니페스트를 STDOUT으로 렌더링
- `kubectl apply -k <디렉토리>`: Kustomize 빌드 후 결과를 클러스터에 적용

## Kustomize Output

- `kustomize build <디렉토리> | kubectl apply -f -`: 렌더링한 매니페스트를 클러스터에 생성/업데이트
- `kubectl apply -k <디렉토리>`: `kustomize build` + `apply` 동작을 단일 명령으로 실행
- `kustomize build <디렉토리> | kubectl delete -f -`: 렌더링한 매니페스트를 클러스터에서 삭제
- `kubectl delete -k <디렉토리>`: `kustomize build` + `delete` 동작을 단일 명령으로 실행
- `-k` 플래그는 Kustomize 모드를 의미

## Kustomize ApiVersion & kind

- 유지보수를 위해 루트 `kustomization.yaml` 상단에 지정하는 것이 좋음
- `apiVersion: kustomize.config.k8s.io/v1beta1`
- `kind: Kustomization`

## Managing Directories

- 디렉토리가 많아지면 `kubectl apply -f <경로>`를 개별적으로 실행해야 함
- 각 디렉토리에 `kustomization.yaml`을 두면 하위 리소스를 체계적으로 관리 가능
- 루트 `kustomization.yaml`에는 서브디렉토리 경로를 `resources`로 나열
- 서브디렉토리 `kustomization.yaml`에는 해당 디렉토리 내 리소스 파일을 지정
- `kubectl apply -k <디렉토리>` 로 일괄 배포 가능

## Common Transformers

- `commonLabels`: metadata `labels` 일괄 추가/변경
- `namespace`: 모든 리소스를 지정한 네임스페이스로 이동
- `namePrefix` / `nameSuffix`: 리소스 이름 앞뒤에 접두·접미사 추가
- `commonAnnotations`: metadata `annotations` 일괄 추가/변경

## Image Transformers

- 관리중인 이미지를 일괄 수정하거나 변환
- `images[].name` : 변경하려는 이미지
- `images[].newName` : 변경될 이미지
- `images[].newTag` : 변경될 이미지 태그

```yaml
images:
  - name: my-app-image
    newName: my-registry/my-app
    newTag: v2.0.0
```

## Patches Intro

- 특정 리소스에만 변경 사항을 적용

- `JSON 6902 Patch ` : 변경할 대상 프로퍼티와 Operation 을 지정
- Operation : `add` | `remove` | `replace`
- Target : `Kind` | `Version` | `Group` | `Name` | `Namespace` | `labelSelector` | `AnnotationSelector`
- Value : 변경할 값 (add, replace 에만 적용)

#### JSON 6902 Patch

```yaml
patches:
  - target:
      kind: Deployment
	  name: api-deployment
	patch: |-
	  - op: replace
	    path: /metadata/name # 변경할 대상
		value: web-deployment
```

- `Strategic Merge Patch` : Kubernetes 구성 파일과 유사한 구조로 사용
- 업데이트할 리소스 종류와 변경할 프로퍼티 제공

#### strategic merge patch

```yaml
patches:
  - patch: |-
      apiVersion: apps/v1
	  kind: Deployment
	  metadata:
	    name: api-deployment
	  spec:
	    replicas: 5
```

## Defferent Types of Patch

- `Inline` : `Kustomization.yaml` 내에서 직접 정의
- `Separate File` : 외부 YAML 파일에 Patch를 정의하고 경로 지정 (Patch 가 많을 경우)

```yaml
# 파일 분리 예시
# Kustomization.yaml
patches:
  - path: replica-patch.yaml
    target:
	  kind: deployment
	  nginx-deployment
---
# replica-patch.yaml
- op: replace
  path: /spec/replicas
  value: 5
```

## Patches Dictionary

- `JSON 6902 Patch` 에서 필드 경로는 `/` 를 통해 구분 (`spec.template.metadata.labels` => `/spec/template/metadata/labels`)
- `Strategic Merge Patch` 에서 프로퍼티 경로는 동일
- `Strategic Merge Patch` 의 Remove Op 에서는 해당 프로퍼티 필드를 null 로 지정하면 지워짐

## Patches list

- `JSON 6902 Patch` 에서 리스트 프로퍼티는 index 로 접근
- `spec.template.spec.containers[0].name` => `/spec/template/spec/containers/0/name`
- 새로운 리스트 항목 추가 시 경로 끝에 `-` 사용 (`/spec/template/spec/containers/-`)
- `Strategic Merge Patch` 에서 리스트 프로퍼티 경로는 동일
- 새로운 요소를 리스트에 추가할 때 새로운 요소만 지정 해주면 됨
- 삭제 시에는 해당 프로퍼티에 `$patch: delete` 지정해주어야 함

## Overlays

- 환경별로 기본 속성을 덮어쓰기 해야할 때 bases 프로퍼티 지정

```yaml
# dev/kustomization.yaml`
bases:
  - ../../base # 기본 속성을 정의한 폴더 지정

resources:
  - grafana-dev.yaml # 환경별 리소스도 추가 가능

patch: |-
  - op: replace
    path: /data/LOG_LEVEL
    value: DEBUG
```

## Components

- 일부 Overlay 에 포함될 수 있는 재사용 가능한 구성 로직 정의
- `/components` 같은 컴포넌트 디렉토리 지정 후 Overlay 에서 import

```yaml
# components/db/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Component
resources:
  - db-config.yaml
---
# overlays/prod/kustomization.yaml
resources:
  - ../../base
components:
  - ../../components/db
```
