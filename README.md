# Certified kubernetes Administrator (CKA) Exam Preparation

## Below Commands help to prepare for CKA exam

- kubernetes cheat sheet   https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- kubernetes documentation https://kubernetes.io/docs/home/

- Important Commands

```bash
kubectl get replace --force -f <file-name>
kubectl apply -f <file-name> --force
```

- Schedule a pod on a specific node Method 1 (Label the node)

```bash
kubectl label node <node-name> <label-key>=<label-value>
```
- Schedule a pod on a specific node

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
    nodeSelector:
      <label-key>: <label-value>
    containers:
    - name: nginx:latest
```


- Schedule a pod on a specific node Method 2 (Node name)

```kubectl
kubectl -n default run pod-name --image=nginx:latest --restart=Never --node=<node-name>
```

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

- Schedule a pod on a specific node Method 3 (Taints and Tolerations)

- Taint the node

```bash
kubectl taint node <node-name> <taint-key>=<taint-value>:<taint-effect>
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx:latest
  tolerations:
  - key: special-hardware
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

### Label and Selectors

- Important Note: labels mentioned in pod TEMPLATE are actual labels, which must be selected by deployment or service to connect with pod (connection), so deployment and services uses
```yaml
selector:
  matchLabels:
    app: App1   
