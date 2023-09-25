# Certified kubernetes Administrator (CKA) Exam Preparation

## Below Commands help to prepare for CKA exam

- kubernetes cheat sheet   https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- kubernetes documentation https://kubernetes.io/docs/home/

- Important Commands

```bash
kubectl get replace --force -f <file-name>
kubectl apply -f <file-name> --force
```

- Schedule a pod on a specific node

```bash
kubectl label node <node-name> <label-key>=<label-value>
kubectl taint node <node-name> <taint-key>=<taint-value>:<taint-effect>
```
- Schedule a pod on a specific node

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
    nodeName: <node-name>
    containers:
    - name: nginx
```