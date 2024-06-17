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

# ConfigMaps
- imperative:
    ```bash
        $kubectl create configmap \ 
            <config-name> --from-literal=<key>=<value>
        $kubectl create configmap \ 
            app-config --from-literal=APP_COLOR=blue
    ```

- declarative:
    ```yaml
        apiVersion: v1
        kind: ConfigMap
        metadata:
            name: app-config
        data:
            APP_COLOR: blue
            APP_MODE: prod
    ```

- view by: 

    ```bash
        $kubectl get configmaps 
        $kubectl describe configmaps 
    ```
- configmaps in Pods (env vs envFrom)
    ```yaml
        envFrom:
          - configMapRef:
              name: app-config #pertains to configmap resource "app-config"
    ```
    injecting just as single ENV variable 
    ```yaml
        env:
          - name: APP_COLOR
            valueFrom:
              configMapKeyRef:
                name: app-config #pertains to configmap resource "app-config"
                key: APP_COLOR
    ```
    inject from volume:
    ```yaml
        volumes:
        - name: app-config-volume
          configMap:
            name: app-config
    ```
- Labs practice notes:
    - Never forget when editing first parts of known list, put "-"
    - because I was editing the first entry within containers list, wont work without the "-"

# Secrets

- imperative:
    ```bash
        $kubectl create secret generic \ 
            <secret-name> --from-literal=<key>=<value>
        $kubectl create secret generic \ 
            app-secret --from-literal=DB_Host=mysql \
                       --from-literal=DB_User=root 
                       --from-literal=DB_Password=paswrd
        OR via file
        $kubectl create secret generic \ 
            app-secret --from-file=app-secret.properties
    ```

- declarative:
    ```yaml
        apiVersion: v1
        kind: Secret
        metadata:
            name: app-secret
        data:
          DB_Host=mysql
          DB_User=root
          DB_Password=paswrd
    ```
    ENCODE IT!!
    ```bash
        $echo -n 'mysql' | base64
    ```
    ```yaml
        apiVersion: v1
        kind: Secret
        metadata:
            name: app-secret
        data:
          DB_Host=bXlzcWw=
          DB_User=cm9vdA==
          DB_Password=cGFzd3Jk
    ```
- view by: 

    ```bash
        $kubectl get secrets
        $kubectl describe secrets #hides values
        $kubectl get secret app-secret -o yaml #shows values encoded
        $echo -n 'mysql' | base64 --decode #decode the encoded values
    ```
- configmaps in Pods (env vs envFrom)
    ```yaml
        envFrom:
          - secretRef:
              name: app-secret #pertains to secret resource "app-secret"
    ```
    injecting just as single ENV variable 
    ```yaml
        env:
          - name: DB_Password
            valueFrom:
              secretKeyRef:
                name: app-secret #pertains to secret resource "app-secret"
                key: DB_Password
    ```
    inject from volume:
    ```yaml
        volumes:
        - name: app-secret-volume
          configMap:
            name: app-secret
    ```
## Secrets Notes:
- Secrets are not Encrypted only Encoded.
- Never check-in secrets objects to SCM
- Secretes are not Encrypted in etcd
- Anyone who can create pods/deployments in the same namespace can access the secrets (duh?)
    - Consider Secrets - RBAC or 3rd party secrets store

# Docker Security
- Containers and host share the same kernel
- Containers are isolated using namespaces in Linux
- Host has its own namespae
- Containers' process are run on host but in container's namespace
- Docker processes can only see within container only
- Docker host has set of users
- By default, Docker processes within containers as root
- To run processes using another user:
    ```bash
        $docker run --user=1000 ubuntu sleep 3600
    ```
- Another way is to define user in Dockerfile to run as certain user during container creation
    ```docker
      FROM Ubuntu
      USER 1000
    ```
- root user in Docker container is different from root user on Host
- Docker limit the abilities of root user within the container
- Docker uses Linux capabilities to implement this
- See full list of Linux capabilities in:
    `/usr/include/linux/capability.h`
- By default Docker runs a container with limited set of capabilities, hence can't disrupt host
- Override this: 
    ```bash
        $docker run --cap-add MAC_ADMIN ubuntu
    ```
    OR
    ```bash
        $docker run --cap-drop KILL ubuntu
    ```
    OR 
    ```bash
        $docker run --priveleged ubuntu
    ```
# Security Contexts
- Add a security context on Pod level
```yml
apiVersion: v1          # String values
kind: Pod
metadata:               # dictionary (2 spaces for tab-ish)
  name: myapp-pod
  labels:
    app: mypod
    type: front-end
spec:
  securityContext:
    runAsUser: 1000
  containers:                   # List/Array (of containers)
    - name: nginx-container     # dash indicates it is item of list
      image: nginx
```

- Add a security context on container level
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
      securityContext:
        runAsUser: 1000
        capabilities:           # NOTE: capabilites are only available at container level NOT Pod
          add: ["MAC_ADMIN"]
```

- Lab notes:
    - When specified to run process as root user, no need to add any `runAsUser`, it is int64 field
    - Remember that capabilities only are for CONTAINER LEVEL!!

# Service Accounts
- 2 kinds of accounts in k8s - User Accounts and Service Accounts
    ```bash
        $kubectl create serviceaccount dashboard-sa
    ```
- When k8s create Service account, it first creates the object, then token for SA
- It then creates secret object and stores that token in that secret.
- Then the secret object is linked to the Service account


    ```bash
        $kubectl describe secret dashboard-sa-token-kbbm
        # lists the token from inside the secret
    ```
- In pod definition, add `serviceAccountName`
- When there is deployment, you can edit the pod directly and the deployment will handle pod deletion
- If no deployments, manually delete the pod before modifying the service account.

- Lab notes:
    - to create an authorization token for a created service account, do below:
    ```bash
        $kubectl create token <service-account-name>
    ```

# Resource Requirements
- kube-scheduler manages in which node the pod is allocated to depending on the resource available
- Resource requests - min amount of resource needed
    - for pod allocation
- 1 cpu = 1 vCPU

    in pod.yaml
    ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: simple-webapp-color
          labels:
            name: simple-webapp-color
        specs:
          containers:
          - name: simple-webapp-color
            image: simple-webapp-color
            ports:
              - containerPort: 8080
            resources:
              requests:
                memory: "1Gi"
                cpu: 1
              limits:
                memory: "2Gi"
                cpu: 2
    ```

### Notes
- in example above, limits is higher than requests
- Request will start pod with 1 Gi, 1 cpu resource
- Limits can be higher since ideally pod is housed in Node with way higher resources
- If cpu utilization exceed allocated (2 vCPU), pod will throttle to not go beyond cpu limit
- If memory usage exceed allocated (2 Gi), pod can go over a bit, 
    but if constantly hitting this, OOM kill will happen.
#### Scenarios
- (2 pods in a node)
##### CPU
- No Request, No Limit
    - 1 pod can starve other pods from even starting up.
- No Request, Limit
    - K8s will set Request = Limit
    - will guarantee resources for pods, but will not use full potential of a Node
- Request, Limit
    - will guarantee resources for pods, but will not use full potential of a Node
- Request, No Limit
    - (Most Ideal) will guarantee resources for pods, will maximize Node CPU cycles

##### Memory
- same idea with CPU scenarios
- except pod gets killed in the instance than pod exceeds max node memory

## Limit Ranges
- Guarantee defaults for pods created without a resource requests and limit
- applies to Namespace level.
- Enforced when pods are created - Does not affect existing pods!
- CPU:
    ```yaml
        apiVersion: v1
        kind: LimitRange
        metadata:
          name: cpu-resource-constraint
        specs:
        - default:
            cpu: 500m
          defaultRequest:
            cpu: 500m
          max:
            cpu: "1"
          min:
            cpu: 100m
          type: Container
    ```
- Memory
    ```yaml
        apiVersion: v1
        kind: LimitRange
        metadata:
          name: memory-resource-constraint
        specs:
        - default:
            memory: 1Gi
          defaultRequest:
            memory: 1Gi
          max:
            memory: 1Gi
          min:
            memory: 500m
          type: Container
    ```

## Resource Quotas
- Manage the resource allocation per Namespace
- I guess to not max out the Node resources
    ```yaml
        apiVersion: v1
        kind: ResourceQuota
        metadata:
          name: my-resource-quota
        spec:
          hard:
            requests.cpu: 4
            requests.memory: 4Gi
            limits.cpu: 10
            limits.memory: 10Gi
    ```

# Taints and Tolerations
- Analogy:
    - Taint is mosquito lotion. Apply to yourself
    - Mosquitoes, is intolerant to the lotion so it will not land on you
    - Say cockroaches, it will be tolerant to the lotion only meant for mosquitoes 
- In K8s:
    - Person is a Node, and bugs are Pods
    - Not exactly for security
    - Used to set restrictions on what pods can be scheduled in a node
    - By default, pods do not have tolerations, so any tainted Node will not schedule default pod
        - Since no pod can tolerant the tainted node
- Summary:
    - Taints are set on Nodes, Tolerations are set on Pods

- Setting Taint
    ```bash
      $ kubectl taint nodes node-name key=value:taint-effect
    ```
    ```
      Taint-effects: NoSchedule | Prefer | NoExecute
    ```
    ```bash
      $ kubectl taint nodes node1 app=blue:NoSchedule
    ```
- Setting Tolerations
    - Sample Taint:
    ```bash
      $ kubectl taint nodes node1 app=blue:NoSchedule
    ```
    - In pod spec:

    ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: myapp-pod
        specs:
          containers:
          - name: nginx
            image: nginx
          tolerations:
          - key: "app"
            operator: "Equal"
            value: "blue"
            effect: "NoSchedule"
    ```
    ```
        NOTE: Always encode the tolerations values with DOUBLE QUOTES!!!
    ```
- Understanding `NoExecute`:
    - Scenario where 2 pods running where 1 pod is tolerated and node just got tainted.
    - The other untolerated pod gets evicted from the tainted node

- Tolerated pod does not guarantee that it will be scheduled in a tainted Node
- Only that the Node will only accept specifically tolerated pod
- For guaranteed scheduling of pods into a node, NODE AFFINITY is concept to check

- So far these have been only dealt on worker nodes. How about the Master node?
- K8s scheduler does not schedule pods in the Master. 
- Because at cluster startup, the scheduler taints the Master taints the Master node.
    ```bash
      $ kubectl describe node kubemaster | grep Taint
    ```
