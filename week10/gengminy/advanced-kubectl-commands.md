## Advanced Kubectl Commands

- JSON PATH로 query하는 것은 수백, 수천 개 개체를 다루는 클러스터에서 아주 유용
- `kubectl`과 `kube-apiserver`는 JSON으로 통신
- `kubectl get <object>` 명령어에는 모든 정보가 표시되지 않기 때문에 JSON PATH 활용이 유용
- 여러 필드를 한번에 표시:
  ```bash
  kubectl get <object> -o=jsonpath={<JSON-PATH-1>}{"\n"}{<JSON-PATH-2>}
  ```
- 메타데이터와 상태 용량을 함께 표시:
  ```bash
  kubectl get <object> \
  -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'
  ```
- 깔끔한 컬럼 출력:
  ```bash
  kubectl get <object> -o=custom-columns=<COLUMN_NAME>:<JSON-PATH>
  ```
- 특정 Path 기준 정렬:
  ```bash
  kubectl get <object> --sort-by=<JSON-PATH>
  ```
