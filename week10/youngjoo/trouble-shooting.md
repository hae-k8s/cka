# Troubleshooting

## Application Failure

```
# Check Service Status
curl http://web-service-ip:node-port
kubectl describe service web-service
# Check pod
kubectl get pod
kubectl describe pod web
kubectl logs web -f --previous
# Check db service
# Check db pod
```

## ControlPlane Failure

```
# Check nodes and pods
kubectl get nodes
kubectl get pods
# Check sevice
service kube-apiserver status
service kube-controller-manager status
service kube-scheduler status
service kubelet status
service kube-proxy status
# View logs
kubectl logs kube-apiserver-master -n kube-system
sudo journalctl 0u kube-apiserver
```

## Worker Node Failure

```
# Check node status
kubectl get nodes
kubectl describe node worker-1
# Check memory
top
df -h
# Check kubelet
service kubelet status
sudo kournalctl -u kubelet
openssl x509 -in /vr/lib/kubelet/worker-1.crt -text
```
