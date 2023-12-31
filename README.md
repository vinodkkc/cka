# Certified kubernetes Administrator (CKA) Exam Preparation

## Below Commands help to prepare for CKA exam

- kubernetes cheat sheet   https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- kubernetes documentation https://kubernetes.io/docs/home/
- kubectl https://kubernetes.io/docs/reference/kubectl/conventions/

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

- Note: Taints tells node to except only those pods which have tolerations for the taints, but taints does not guarantee that pods will be scheduled on the node, it just tells node to except only those pods which have tolerations for the taints

- Note: if you have certain requirement to restrict a pod to certain node, it will achieve by using nodeaffinity.
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

- kube-scheduler decides where to place pods in the cluster (Note: kube-scheduler is not a pod, it is a process running on the master node & it does not run on the worker node, further it does not responsible for running the pod on the node)
- kublet is responsible for running the pod on the node & it runs on the worker node.
- kubeadm tool does not install kublet, it is the responsibility of the admin to install kublet on the worker node.(manually)

### Sample yaml pod file with kubectl commands
- Note: Must use --dry-run=client
```bash
kubectl run redis --image=redis:latest -n default --dry-run=client -o yaml > pod-definition.yaml
```

### kubectl help
```bash
kubectl run --help
kubectl create --help
kubectl expose --help
kubectl get --help
kubectl describe --help
kubectl delete --help
kubectl edit --help
kubectl label --help
kubectl annotate --help
kubectl scale --help
kubectl autoscale --help
kubectl rollout --help
kubectl rollout history --help
kubectl rollout undo --help
kubectl rollout status --help
kubectl rollout restart --help
kubectl rollout pause --help
kubectl rollout resume --help
```

- ReplicaSet
- Note: ReplicaSet is the next generation of Replication Controller, ReplicaSet is the new way of creating Replication Controller.
- Replicaset yaml
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      type: front-end
  template:
    metadata:
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
 ```
    
```bash
kubectl create -f replicaset-definition.yaml
kubectl get replicaset
kubectl get rs
kubectl describe replicaset myapp-replicaset
kubectl delete replicaset myapp-replicaset
kubectl replace -f replicaset-definition.yaml
kubectl scale --replicas=6 -f replicaset-definition.yaml
kubectl scale --replicas=6 replicaset myapp-replicaset
kubectl scale --replicas=6 rs/myapp-replicaset
kubectl edit replicaset myapp-replicaset
kubectl rollout status replicaset myapp-replicaset
kubectl rollout history replicaset myapp-replicaset
kubectl rollout undo replicaset myapp-replicaset
kubectl rollout undo replicaset myapp-replicaset --to-revision=2
kubectl rollout restart replicaset myapp-replicaset
kubectl rollout pause replicaset myapp-replicaset
kubectl rollout resume replicaset myapp-replicaset
```

- kubectl explain will list the fields of the resource, for example apiVersion, kind, metadata, spec, status etc.
```bash
kubectl explain pods    # list the fields of the pod
kubectl explain pod.spec # list the fields of the spec of pod
kubectl exaplain replicaset # list the fields of the replicaset
kubectl explain replicaset.spec # list the fields of the spec of replicaset
```

## YAML Creation from kubectl command

- Create an NGINX Pod
```bash
kubectl run nginx --image=nginx
```

- Generate POD Manifest YAML file (-o yaml). Don’t create it(–dry-run)
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

- Create a deployment
```bash
kubectl create deployment --image=nginx nginx
```

- Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run)
```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml 
```

- Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.
```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml 
```

- Make necessary changes to the file (for example, adding more replicas) and then create the deployment.
```bash
kubectl create -f nginx-deployment.yaml 
```

- In k8s version 1.19+, we can specify the –replicas option to create a deployment with 4 replicas.
```bash
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml 
```

#### ETCD – Commands (Optional)
- (Optional) Additional information about ETCDCTL UtilityETCDCTL is the CLI tool used to interact with ETCD.ETCDCTL can interact with ETCD Server using 2 API versions – Version 2 and Version 3.  By default it’s set to use Version 2. Each version has different sets of commands.

- For example, ETCDCTL version 2 supports the following commands:
```bash
etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set
```

- Whereas the commands are different in version 3
```bash
etcdctl snapshot save
etcdctl endpoint health
etcdctl get
etcdctl put 
```

- To set the right version of API set the environment variable ETCDCTL_API command
```bash
export ETCDCTL_API=3 
```

- When the API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don’t work. When API version is set to version 3, version 2 commands listed above don’t work.
- Apart from that, you must also specify the path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don’t worry if this looks complex.
```bash
--cacert /etc/kubernetes/pki/etcd/ca.crt
--cert /etc/kubernetes/pki/etcd/server.crt
--key /etc/kubernetes/pki/etcd/server.key 
```

- So for the commands, I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form.
```bash
kubectl exec etcd-controlplane -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key"
```

### Kubernetes Yaml fixation tools
- kubeval:
- Validates your Kubernetes configuration files against the official Kubernetes schemas, It can identify misconfigurations but doesn't necessarily automatically fix them.
```bash
 curl -L https://github.com/instrumenta/kubeval/releases/latest/download/kubeval-linux-amd64.tar.gz | tar xvz
sudo cp kubeval /usr/local/bin
kubeval my-k8s-deployment.yaml
```

- kube-score:
- kube-score is a tool that performs static code analysis of your Kubernetes object definitions. The output is a list of recommendations of what you can improve to make your application more secure and resilient.
```bash
curl -L https://github.com/zegl/kube-score/releases/download/v1.13.0/kube-score_1.13.0_linux_amd64.tar.gz | tar xvz
sudo cp kube-score /usr/local/bin
kube-score score my-k8s-deployment.yaml
```

- kube-linter:
- kube-linter is a static analysis tool that checks Kubernetes YAML files and Helm charts to ensure the applications represented in them adhere to best practices.
```bash
curl -L https://github.com/stackrox/kube-linter/releases/download/0.2.5/kube-linter-linux.tar.gz | tar xvz
sudo cp kube-linter /usr/local/bin
kube-linter lint my-k8s-deployment.yaml
```

- pluto:
- Pluto is a CLI tool to help discover deprecated apiVersions in Kubernetes. Pluto will examine Kubernetes manifests, cluster-wide, and report on deprecated apiVersions it finds.
```bash
curl -L https://github.com/FairwindsOps/pluto/releases/download/v5.18.5/pluto_5.18.5_linux_amd64.tar.gz | tar xvz
sudo mv pluto /usr/local/bin/
helm template -f values.yaml ./ | pluto detect -
```