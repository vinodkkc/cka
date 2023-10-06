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
```

- filter with kubectl
```bash
kubectl get pods -n default -l tier=db
NAME         READY   STATUS    RESTARTS   AGE
db-1-d2hlf   1/1     Running   0          10m
db-2-hpnnx   1/1     Running   0          10m
db-1-fcr54   1/1     Running   0          10m
db-1-b79wp   1/1     Running   0          10m
db-1-c76z8   1/1     Running   0          10m

kubectl get pods -n default -l env=dev
NAME          READY   STATUS    RESTARTS   AGE
db-1-d2hlf    1/1     Running   0          10m
app-1-7tmj2   1/1     Running   0          10m
db-1-fcr54    1/1     Running   0          10m
app-1-qcr4m   1/1     Running   0          10m
app-1-tz9qx   1/1     Running   0          10m
db-1-b79wp    1/1     Running   0          10m
db-1-c76z8    1/1     Running   0          10m
```
- describe pod to list the labels on a pod
  
```bash
kubectl describe pod/db-1-d2hlf -n default
Name:             db-1-d2hlf
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.2.163.8
Start Time:       Tue, 26 Sep 2023 03:05:48 +0000
Labels:           env=dev
                  tier=db
Annotations:      <none>
Status:           Running
IP:               10.42.0.14
IPs:
  IP:           10.42.0.14
Controlled By:  ReplicaSet/db-1
```

- filter --no-headers
```bash
kubectl get pods -n default -l env=dev --no-headers | wc -l
```
- filter multiple labels
```bash
kubectl get all -A -l env=prod,bu=finance,tier=frontend
```

- Taints & tolerations
```bash
kubectl taint nodes node1 key=value:NoSchedule
kubectl taint nodes node1 key=value:NoExecute
kubectl taint nodes node1 key=value:PreferNoSchedule
```
- Noschedule: No new pods will be scheduled on the node but existing pods will continue to run
- NoExecute: No new pods will be scheduled on the node and existing pods will be terminated (evicted) if they do not tolerate the taint
- PreferNoSchedule: Kubernetes will try to avoid scheduling new pods on the node but there is no guarantee

- Taints & tolerations example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest  # Fixed the indentation here
  tolerations:             # tolerations are used to tolerate the taints
    - key: "key"           # key of the taint
      operator: "Equal"    # Fixed the indentation for the following lines
      value: "value"       # value of the taint
      effect: "NoSchedule" # effect of the taint
```

    
