# Advanced Kubectl Commands

## JSON Path in KubeCtl

- 100개가 넘는 Node, 1000개가 넘는 pod를 다루려면 JSON Path를 다룰 수 있어야 함
- 그냥 kubectl describe와 같은 커맨드를 사용하면 너무 많은 정보가 출력됨
- json path를 이용하면 원하는 정보만 가져올 수 있음

```
k get pods -o=jsonpath='{.items[0].spec.containers[0].image}'
# 두개 이상 쿼리를 사용할 수도 있음
k get pods -o=jsonpath='{.items[0].spec.containers[0].image}{.items[0].spec.containers[0].name}'
# 원하는 형태로 출력도 가능
k get pods -o=jsonpath=`{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'
k get nodes -o=custom-columns=<COLUMN NAME>:<JSON PATH>
# Sort By
k get nodes --sort-by=<JSON PATH>
```
