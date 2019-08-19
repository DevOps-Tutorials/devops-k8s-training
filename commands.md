```
kubectl get pods -o wide --all-namespaces
kubectl describe pods/kube-dashboard-aksjdhfl-uuiuoyb -n kube-system
kubectl logs kube-dashboard-aksjdhfl-uuiuoyb --follow
kubectl exec -it shell-demo -- /bin/bash
kubectl exec shell-demo cat /proc/1/mounts
```