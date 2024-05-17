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
6. kubelet - agent that runs on nodes, makes sure that containers running as expected

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

