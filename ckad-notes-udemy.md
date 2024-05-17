# Core Concepts
### Basics
1. Nodes - worker, where kube is installed.
2. Cluster - group of nodes
3. Master - prolly control plane??

### Components
1. API Server - frontend of kube
2. etcd - keystore, for management of cluster
3. Scheduler - distributor of load to workers 
4. controllers - brain behind orchestration, aware when containers go to shit
5. runtime - software for running containers like docker
6. kubelet - agent that runs on nodes, makes sure that containers 
running as expected

### Master vs Worker Nodes
#### Worker Has:
1. container runtime (docker)
2. kubelet - this is why it is the worker

#### Master Has: 
1. kube-apiserver - this is why it is the master
2. etcd
3. controller
4. scheduler

### kubectl
1. cli tool
2. kubectl cluster-info 

### docker vs containerD

1. kube -> CRI -> containerd -> docker
2. kube -> CRI -> containerd
3. ctr -> debugging containerd
3. nerdctl - CLI for containerD, runs like docker in cli
4. crictrl - CLI for CRI compatible container runtimes (for container debugging)
5. crictrl - aware of pods

#### Summary
||ctr|nerdctl|crictl|
|-|-|-|-|
|Purpose|Debugging|General Purpose|Debugging|
|Community|ContainerD|ContainerD|Kubernetes|
|Works With|ContainerD|ContainerD|All CRI Compatible Runtimes|

### Pods recap
- kubectl nginx --image nginx
- k get pods

### YAML recap
```yml
apiVersion: v1          # String values
kind: Pod
metadata:               # dictionary (2 spaces for tab-ish)
  name: myapp-pod
  labels:
    app: mypod
    type: front-end
spec:
  containers:                   # List/Array (of containers)
    - name: nginx-container     # dash indicates it is item of list
      image: nginx
```
|Kind|Version|
|-|-|
|Pod|v1|
|Service|v1|
|ReplicaSet|apps/v1|
|Deployment|apps/v1|
#### Yaml notes
- cant add more to metadata than what k8s supports
- labels are for keyval pairs user decides 
- k run for pods
- k create for others

#### Practical exam on Pods notes
- If you are not given a pod definition file, you may extract the definition to a file using the below command:
```
kubectl get pod <pod-name> -o yaml > pod-definition.yaml
```
Then edit the file to make the necessary changes, delete, and re-create the pod.

- To modify the properties of the pod, you can utilize the kubectl edit pod <pod-name> command. Please note that only the properties listed below are editable.

  - spec.containers[*].image

  - spec.initContainers[*].image

  - spec.activeDeadlineSeconds

  - spec.tolerations

  - spec.terminationGracePeriodSeconds

### ReplicaSet











