# Logging & Monitoring

## Monitoring

- 자체적으로 탑재된 솔루션은 없음
- Metric Server, Prometheus, Elastic Stack, ..
- Metric Server
  - In Memory service

```
// minikube
minikube addons enable metric-server

// other
git clone + kubectl create ...

// monitor
kubectl top node
kubectl top node
```

## Logging

```
// docker
docker run kodekloud/event-simulator

docker logs -f ecf

// event-simulator.yaml
apiVersion: v1
    kind: Pod
metadata:
    name: event-simulator-pod
spec:
    containers:
        - name: event-simulator
        image: kodekloud/event-simulator

// kube (pod 이름 명시해야함)
kubectl create -f event-simulator.yaml
kubectl logs -f event-simulator-pod event-simulator
```
