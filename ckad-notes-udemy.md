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

- Ensures that there is always the node is running specified pods 
- In case of crashes, it will try to maintain state by bringing up new instance
- helps load balancing
- Replication Controller - older tech than Replica Set

#### Sample Replication Controller
```yml
apiVersion: v1          # NOTE diff from ReplicaSet
kind: ReplicationController
metadata:               # dictionary (2 spaces for tab-ish)
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:                     # define a pod template
    metadata:                   # dictionary (2 spaces for tab-ish)
      name: myapp-pod
      labels:
        app: mypod
        type: front-end
    spec:
      containers:                   # List/Array (of containers)
        - name: nginx-container     # dash indicates it is item of list
          image: nginx
  replicas: 3
```

#### Sample ReplicaSet
```yml
apiVersion: apps/v1          # NOTE this diff from Replication Controller
kind: ReplicationController
metadata:               # dictionary (2 spaces for tab-ish)
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:                     # define a pod template
    metadata:                   # dictionary (2 spaces for tab-ish)
      name: myapp-pod
      labels:
        app: mypod
        type: front-end
    spec:
      containers:                   # List/Array (of containers)
        - name: nginx-container     # dash indicates it is item of list
          image: nginx
  replicas: 3
  selector:                         # MAJOR DIFF from replication controller 
    matchLabels:
      type: front-end
```
#### NOTES:
- error when "v1" is used for ReplicaSet
  - error: unable to recognize "<yaml def name>.yaml": no matches for /, Kind=ReplicaSet
- Needs selector because ReplicaSet can also manage pods that were not created as p:w
art of the ReplicaSet.
  - i.e. pods that were created before the creation of ReplicaSet with matching labels

#### Labels and Selectors
- Example: 
  - current deployed pods of label: front-end is 2, 
  - replicaSet is created with 3 replicas in spec.
  - replicaSet brings up 1 more pod to match its spec
  - To know which kind of pod to bring up, it will need the pod label.
- Acts as filters for ReplicaSet for pod monitoring

#### Scaling with ReplicaSet
- increase replicas in ReplicaSet spec. Then k replace -f
- k scale --replicas=N filename OR TYPE NAME
- using filename will not update the replicas in file.

### Deployments
- Rolling Updates, Rollback, Canary and Blue/Green, pause and play changes
- Containers are contained in pods, pods monitored by ReplicaSet, Deployment is higher

#### Sample Deployment
```yml
apiVersion: apps/v1
kind: Deployment
metadata:               # dictionary (2 spaces for tab-ish)
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:                           # Very much like a ReplicaSet spec
  template:
    metadata:
      name: myapp-pod
      labels:
        app: mypod
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:                         # MAJOR DIFF from replication controller 
    matchLabels:
      type: front-end
```

### Namespaces
- default: user defined objects default namespace
  - should be fine for most playground stuff
- kube-system: networking stuff for k8s. must be away from user
- kube-public: resources for all users
- create new namespaces for things like Environment isolation.
- DNS
  - db-service for local namespace
  - db-service.dev.svc.cluster.local
  - ".dev.svc.cluster.local" DNS entry added in this format when service is created
    - default domain name - cluster.local
    - service - svc
    - namespace - dev
    - service name - db-service

```
k get pods
k get pods --namespace=namespace-name
```
```yml
metadata:
  namespace: namespace-name           # for pods
```

#### Resource Quota sample
```yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

### Practice on imperative commands
- notable part of practice labs was when a pod needs to have a clusterIP service also exposing its port, use --expose
- how i solved this though is by seeing that the label for pod is "run=httpd" while selector of svc is "app=httpd"

# Configuration

### Container images

- Didnt feeling noting a lot here.
- only notable thing I stumbled upon is this:
  - To get the base OS of an image (has to be unix-based ig.)
    ```bash
      $ docker run <image_name> cat /etc/*release*
    ```
- other notable quirks between docker CLI and k8s (kubectl)
  - order of placing command options, so always consult --help
  - no `--dry-run=client` to save you here in docker land
  - `docker run -p HOST:CONTAINER IMAGE_NAME` - not much of guide in help menu about this.

- Override default command... I'm thinkin' busybox here.
    ```bash
      $ docker run <image_name> [command]
      $ docker run ubuntu sleep 5
    ```
    OR
    ```docker
      FROM Ubuntu
      CMD sleep 5 # CMD command param
    ```
    OR
    ```docker
      FROM Ubuntu
      CMD ["sleep", "5"] # CMD ["command", "param"]
    ```
- Parameterized way (ENTRYPOINT)
    ```docker
      FROM Ubuntu
      ENTRYPOINT ["sleep"]
    ```
    then
    ```bash
      $ docker run <image_name> [ARGS]
      $ docker run ubuntu-sleeper 10
    ```
- What if no args, need default args
    ```docker
      FROM Ubuntu
      ENTRYPOINT ["sleep"]
      CMD ["5"]
    ```
    same as 
    ```bash
      $ docker run ubuntu-sleeper 5  #default
      $ docker run ubuntu-sleeper 10 #args provided
    ```
- if one needs to override the entry point during runtime
    ```bash
        #sleep2.0 is an imaginary command
      $ docker run --entrypoint sleep2.0 ubuntu-sleeper 10
    ```
- Kubernetes relation to Docker commands:
    ```bash
        #sleep2.0 is an imaginary command
      $ docker run --entrypoint sleep2.0 ubuntu-sleeper 10
    ```
    in pod.yaml
    ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: ubuntu-sleeper-pod
        specs:
          - name: ubuntu-sleeper
            image: ubuntu-sleeper
            command: ["sleep2.0"]
            args: ["10"]
    ```

- Lab Notes:
    - On "what commands run at pod startup":
        - doesnt matter what the dockerfile say, look and pod spec
        - if only has command or has args, that will be the answer.
        - command always override image default CMD & ENTRYPOINT










