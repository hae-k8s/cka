# Other Topics

## Advanced Kubectl Commands

1. JSON 형태로 출력
    
    ```bash
    kubectl get nodes -o json
    kubectl get pods -o json
    ```
    
2. JSONPath 옵션으로 실행
    
    ```bash
    kubectl get nodes -o jsonpath='{.items[0].spec.containers[0].image}'
    ```
    
- 예시
    - 노드 이름 조회
    
    ```bash
    kubectl get nodes -o jsonpath='{.items[0].spec.containers[0].image}'
    ```
    
    - 노드 아키텍처 조회
    
    ```bash
    kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.architecture}'
    ```
    
    - 노드 CPU 개수 조회
    
    ```bash
    kubectl get nodes -o jsonpath='{.items[*].status.capacity.cpu}'
    ```
    
    - 여러 정보 한 번에 조회 (포맷팅)
    
    ```bash
    kubectl get nodes -o jsonpath='{.items[*].metadata.name}{"\n"}{.items[*].status.capacity.cpu}'
    ```
    
    - 반복문 사용 (`range`로 반복 시작, `end`로 반복 종료)
    
    ```bash
    kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'
    ```
    
    - 커스텀 컬럼 사용
    
    ```bash
    kubectl get nodes -o custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu
    ```
    
    - 정렬 기능
    
    ```bash
    kubectl get nodes --sort-by='.metadata.name'
    kubectl get nodes --sort-by='.status.capacity.cpu'
    ```