# Kubernetes & Istio with Google Cloud

## DevSecOps

- Company Loan Pig (2015) - car loan business
- they have on-premise servers in their own data centers storing their offline shop data; managed by vendors
- in 2017, they are creating a mobile app; vendor does not have capability to create a mobile application
- additionally, hardware cost is big - hardware itself, electricity, installation, configuration, maintenance
  - their decision is to move to cloud provider; they reorganize their IT team - they now have a cloud infrastructure team which is in charge of preparing Linux server, Postgresql management on AWS, maintaining other infrastructure like firewall, DNS, etc...
  - they hire a frontend engineering team to create Android and iPhone apps and a backend engineering team to create a server application in Java
  - they also hire a monitoring team to ensure no significant downtime, resource utilization, deployment
- in 2019, the company expands the business, they acquire other loan products, houses, electronic loan, peer-to-peer lending

  - for that purpose they form multiple software development teams
  - because the application that they purchased from vendor is not enough anymore (it doesn't meet all requirements), an internal team must build it from scratch with additional features
  - let's say that every first week of the month, an increased payment rate happens, so two new servers are spawned where the whole application is brought up
    - to do this manually every month - risky and a lot of manual repetitive work (to spawn two servers, install and configure them, let them work for a week and then shut them down for the rest of month)
    - let servers work all the time despite the fact that they are being utilized only every first week of the month - expensive
  - they introduce a microservice architecture to handle this many features as the monolith would be difficult to extend, maintain and scale up/down
  - with microservices, a deployment cycle is shorter since the deployments should be independent (and release changes are usually smaller) as teams which develop/maintain them; also this requires more attention of an operational team - multiple instances and servers are maintained

- DevOps cycle:

1. Plan - what to build?
2. Code - implement the code
3. Build - package to binary
4. Test - test quality
   ---- dev(elopment) -----
5. Release - release new version
6. Deploy - deploy to production
7. Operate - new release used
8. Monitor - monitor & feedback
   ---- ops(operations) ----

- DevSecOps
  - DevOps with the security details planned and implemented

1. Plan - when to use OTP (one-time password)
2. Code - write the secure code to avoid SQL injection, XSS
3. Build - scan libraries for known CVE
4. Test - penetration testing
5. Release - security change log
6. Deploy - scan deployable binary
7. Operate & monitor - spiking API traffic (possible DOS attack), many unauthorized accesses

- DevOps and DevSecOps is not a tool, but a set of working processes, tools, culture where the development and operations work together

## Container technology

- microservices: every microservice can be written in a different technology, e.g.
  - Service A: Java, Spring Boot -> JAR file, requires Java runtime
  - Service B: Go, native lib -> Native, must run on Linux (native Go can be run only on environment where it was built)
  - Service C: Python, Pandas -> Not compiled, needs Python runtime
- all of these can be packaged as containers
  - Container A: Linux OS + JRE + JAR file
  - Container B: Linux OS + compiled go binary
  - Container C: Linux OS + Python interpreter + Python source code
- Microservices
  - when we have only a few services
    - one service has one dedicated server
    - own OS & environment
  - a lot of services:
    - a dedicated server per service might increase cost & waste resources
    - server not always busy
    - server configuration & management
    - more resources mean more staff
- Container:
  - Operating system (minimum version)
  - Runtime environment (Java, Python, .NET, Erlang...)
  - Mandatory libraries
  - Application
- E.g. RabbitMQ
  - Linux Debian
  - Erlang runtime
  - Mandatory libraries
  - Rabbitmq
- Java service container
  - Linux Alpine
  - JRE
  - Tomcat & other libs
  - Runnable JAR
- we can run multiple containers per physical or per virtual machine
- when one container dies, others are still operational

### Docker

- Image:
  - packages application & all things required
  - like a zip archive with metadata
- Container:
  - a running instance of an image
  - once container = one image instances
  - independent
  - resource can be limited
- Registry (repository)
  - public & private
- Docker image layers

  - images are packed and stored as layers
  - let's say that we have two images:
    - A consisting of:
      - Custom application A (20 MB)
      - Library Q version 9.1.5 (30 MB)
      - Library P version 2.0.0 (40 MB)
      - Java Runtime 17.4.1 (75 MB)
      - Linux Alpine 3.22.4 (60 MB)
      ***
      - Total: 225 MB
    - B consisting of:
      - Custom application B (17 MB)
      - Library Q version 9.2.2 (34 MB)
      - Library P version 2.0.0 (40 MB)
      - Java Runtime 17.4.1 (75 MB)
      - Linux Alpine 3.22.4 (60 MB)
      ***
      - Total: 226 MB
  - server will store the following layers:
    - Custom application A (20 MB)
    - Library Q version 9.1.5 (30 MB)
    - Custom application B (17 MB)
    - Library Q version 9.2.2 (34 MB)
    - Library P version 2.0.0 (40 MB)
    - Java Runtime 17.4.1 (75 MB)
    - Linux Alpine 3.22.4 (60 MB)
    ***
    Total: 276 MB

- `--restart always` if you want to start the Docker container automatically when the computer is (re)started
- in YAML, the order of the defintions is not important
- to manage multiple containers, we use orchestrators:
  - Docker Swarm
  - Apache Mesos
  - Kubernetes

## Kubernetes basics

- manages multiple containers within one or more hosts
- manages the availability
  - starts the container automatically
  - restarts the container when it crashes
- a pod - contains one or more containers, but the commmon practice is to have one container per pod
- Kubernetes cluster
  - master node (control plane) - K8s management processes
    - API server: front-end for K8s
    - scheduler: watches pods that are unassigned to any worker node and assign those podes to free worker node
    - ectd: key-value store that acts as K8s data storage
    - controller manager: manages K8s processes (pods, services, nodes, replications)
  - worker nodes - nodes with pods that have some responsibility/task (has more resources - CPU, memory and disk)
    - kubelet: make sure the container on pod is running
    - kube proxy: network & communication proxy
    - container runtime: runtime (e.g. Docker)
  - service - exposes a pod to outer world
  - replication is easy with K8s
  - K8s can monitor the load on pods and dynamically scale up/down the number of replicas based on the traffic load - Horizontal Pod Autoscaler
- pods can be grouped logically to namespaces
- pods can have labels
- K8s have many types of volumes
  - config map
  - local
  - secret
  - cloud storage (Google persistent disk, Azure disk, AWS EBS volume)
- K8s can allocated the same pod replicas to different worker nodes (they are reached via service)

- K8s in cloud:
  - Google: Google Kubernetes Engine (GKE)
  - AWS: Elastic Kubernetes Service (EKS)
  - Azure: Azure Kubernetes Service (AKS)
- locally: minikube

### Hello Kubernetes

1. `kubectl create deployment my-nginx --image nginx:stable` - creates a deployment using Nginx image
2. expose the deployment to outer world - `kubectl expose deployment my-nginx --type NodePort --port 80` -> on minikube an additional step is needed -> `minikube service my-nginx --url`. Use the given address to access Nginx.
3. `kubectl api-resources` - list down all resources available in K8s
4. `kubectl get` and `kubectl describe` requires a resource as a parameter, e.g.
   - `kubectl get pod abc`
   - `kubectl describe service xyz` or shorter `kubectl describe svc xyz`

#### Minikube

- `minikube start` - to start local K8s cluster (requires Docker as prerequisite)
  - `minikube start --memory 8192 --cpus 2`
- `minikube stop` - stops K8s, but doesn't delete resources
- `minikube delete` - stops K8s and deletes all deployed resources
- `minikube tunnel` - creates a network tunnel between K8s and host (able to access to local cluster from browser)
- `minikube addons enable/disable` - enables/disables minikube addons
- `minikube service <service-name>` - opens a service in browser

#### Scaling pods

- image: `npantelic/devops-blue:1.0.0`
- `kubectl expose deployment my-devops-blue --type LoadBalancer --port 8111 --name my-devops-blue-lb` - creates a LoadBalancer service
- we can also access the pod if we use the virtual IP that can be fetched with `kubectl describe service <service-name>`
- even if we have multiple replica pods, all of them are access through one service endpoint; the load is dynamically distributed based on the number of available pods
- `kubectl scale deployment my-devops-blue --replicas 3` - to scale deployment to use 3 pods
- `kubectl get pod -o wide` - for a wide output that contains IP addresses

## Declarative Kubernetes

### Declarative vs imperative

- create/read/update/delete using `kubectl`
  - **Imperative** approach
- in reality, use **declarative** approach

  - write Kubernetes configuration file
  - apply the configuration file

- imperative (procedural)
  - how to make a capuccino procedurally?
    1. prepare a cup for 500cc
    2. boil 200cc of water
    3. pour boiling water into a cup
    4. pour 2 tablespoons of ground coffee into a cup
    5. add 200cc of foam milk
    6. pour milk into coffee
    7. add liquid sugar if you like
- imperative - the exact steps on how to do something (order may be important) -> HOW
- imperative K8s -

  1. create a deployment from the image `devops-blue:1.0.0`
  2. create a service with type LoadBalancer to port 8111
  3. scale pod to 3 replicas

- declarative - use an automatic coffee machine and offload the knowledge on HOW TO EXACTLY MAKE, so you have to know WHAT to make, and not how
- Declarative - WHAT to make, automate HOW

- So imperative K8s - how to operate K8s deployment (a living person, a professional with K8s knowledge)
- declarative K8s - what configartion is needed, K8s will automate the HOW part
- K8s config file - YML/YAML format; YAML - consistent use of **n** spaces for indentation level
- apply the file, leave the rest to K8s
- using declarative approach is recommended

- YAML config file declaration leverages K8s REST API (it uses that REST API behind the scene)
- `kubectl explain <resource>` - to get the doc details about that resource. We can also use this command to explain the fields of a resource. E.g. `kubectl explain deployment.spec` or `kubectl explain deployment.spec.selector`

- K8s configuration file (manifest)

```
apiVersion: ...
kind: ...
metadata:
...
spec:
...
status:
...
```

- `apiVersion` - REST API version used
- `kind` - object type specified in the manifest file
  ---- these two values are fixed and we can get them from K8s API reference ----
- `metadata` - contains the basic information about the object instance, like its name, version, owner of the object, creation date etc.
  - fields in metadata are the same for all object types
- `spec` (important) - part in which the target state of the object is declared/specified
  - fields in spec section are different for different object types
  - e.g. for pod it will declare pod's containers, ports, volumes and other info
  - for service that could be service type, IP address spec etc.
- `status` (important) - contains the current actual state of the object
  - e.g. for pod this section contains the condition of the pod, the status of each of its containers, its virtual IP and other information that reveals what is happening to pod
- some object do not contain `spec` and `status`, like event object because such object usually contain static data
- `spec` is defined by the engineer - that is a desired state
- `status` is the current state and that is not defined by an engineer, but monitored
- `controller` is a K8s component that monitors the target state based on the spec section and reports the actual state, updating the status section
- controller will then perform the necessary operations to achieve the target state
- labels are the way to group resources; we can add as many as needed
- there are some common labels that are usually used like `app.kubernetes.io/name`
- `spec.selector.matchLabels` - the value of this label must match the label value in spec

- applying the K8s configuration manifest: `kubectl apply -f <filename.yml>`
- `kubectl get pod --show-labels` - get pods with labels output

- Deployment - high level specification on what to deploy, we mostly work with this yaml object
- Pod
  - the actual application that runs
  - pod specification is defined in yaml deployment
- ReplicaSet
  - internally created with the purpose to maintain the number of requested pods
- **Update**: if we have a deployment up & running and we change something in the deployment definition and apply again the same config, controller will do the necessary changes to reach the desired state (terminates the existing pods and create new ones)
- **Delete**: `kubectl delete -f <filename.yml>`
- we can create multiple object in the same manifest file. Objects should be separated wih `---` (three dashes)
- `kubectl get pod -n <namespace>` - get all pods belonging to a given namespace or in general `kubectl get <resource> -n <namespace>` for any resource type

## Operating Kubernetes

### Labels

- used to identify K8s objects
- one object can have multiple labels
- one label can be used for multiple objects
- use the label selector for select matching objects

- Example:
  - Pod 1
    - image: devops-blue:1.0.0
    - labels:
      - devops-label-on-pod
      - 1.0.0
  - Pod 2
    - image: devops-blue:2.0.0
    - labels:
      - devops-label-on-pod
      - 1.0.1
  - Service-v1 (9011)
    - labels:
      - devops-label-on-pod
      - 1.0.0
    - note: thus applicable to Pod 1
  - Service-v2 (9012)
    - labels:
      - devops-label-on-pod
      - 1.0.1
    - note: thus applicable to Pod 2
  - Service-v3 (9013)
    - labels:
      - devops-label-on-pod
      - 2.0.0
    - note: this service does not match any pod by labels
  - Service-all (9014)
    - labels:
      - devops-label-on-pod
    - note: this service expose both Pod 1 and Pod 2 since their labels match the label of the this service

### Annotation

- key-value pair like label used to annotate a Kubernetes resource
- label vs annotation
  - similar format: key-value
  - label is intended for the internal K8s engine (e.g. to identify a pod)
  - annotation is for human use or 3rd party application which is installed on the same K8s cluster
- metadata section:

```
metadata:
  ....
  labels:
    app.kubernetes.io/name: my-funny-app
    app.kubernetes.io/version: 2.0.4
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8113"
    prometheus.io/path: /metrics/prometheus
```

- e.g. Prometheus can filter pod annotated with the expected annotations and use to them to index metrics data

### Port forwarding

- we sometimes need to access the pod directly
- if we have a load balancer and multiple replicas that can be bothersome
- we can open (forward) a pod application port by mapping the host port to it
- `kubectl port-forward [pod-name] host-port:pod-port`

### Healthcheck

- let's say we have 3 replicas and a load balancer service
- one of the pods crash
- K8s will do:

  1. redistribute traffic to 2 healthy pods
  2. restart the crashed pod to achieve the replica set number
  3. once the third pod is healthy, it will redistribute traffic to 3 nodes

- How K8s does know when the pod is healthy?

  - it periodically (e.g. one minute) runs a terminal command/http/tcp/grpc
  - if the health check response is good, then a pod is "healthy"
  - otherwise (check response exceeds the "not good" threshold), then a pod is marked as "not healthy"

- Two states in the pod's life:
  - readiness: pod is ready to accept the client request. K8s will only send the request to such pods
    - redirect traffic to healthy pod
    - a pod is ready when all its containers are ready
  - liveness: when the pod is ready, K8s will check if the pod is still alive. If it is not, it will restart it.
- Probes
  - probe to gather state
  - readiness/liveness probes
  - startup probe: in (legacy) application which takes a long startup time
- Example:
  - devops-blue has:
    - `/actuator/health/readiness`
    - `/actuator/health/liveness`
- use health check probe without any authentication (or supply the correct authentication credentials)

- note about devops-blue headers:

1. `K8s-App-Version`: application version (will be equal to Docker image tag)
2. `K8s-App-Identifier`: application name (with random number) & virtual IP
3. `K8s-Pod-Name`: pod name

### Pod lifecycle

- pod status:
  1. **Running**
  - all containers have been created
  - at least one container is running
  2. **Pending**
  - still download the container image
  - another option is that the pod cannot be scheduled to run due to resource constraints (e.g. not having enough memory)
  3. **Succeeded**
  - all containers within pod have have been terminated and will not be restarted
- when using the deployment configuration, the pod restart policy is `Always` - the pod will always restart if it has crashed and thus pod should not get into this state 4. **Failed**
  - all containers within a pod terminated
  - at least one termination returned an error code
  5. **Unknown**
  - cannot determine the pod status
  - cannot communicate with the worker node hosting the pod (network issue or the worker node is down)
  6. **CrashLoopBackOff**
  - error, pod keeps restarting
  - e.g. due to deployment error or missing the runtime dependency
  7. **ImagePullBackOff**
  - Cannot pull an image (network problem, private repository)
  - troubleshooting: `kubectl describe pod [pod-name]` and delete the pod since K8s will try to pull the image again and recreate it

### Log

- `kubectl logs <pod-name>` - list down all logs
- `kubectl logs --select <selector-value>` - list down all logs of the pods matching the given selector

## Kubernetes UI

- When working with minikube
  - `minikube addons list`
  - `minikube dashboard` - to open a UI
- every cloud provider has its own UI

- an alternative tool: K8s Lens (k8slens.dev)

## Volume

- a pod can be destroyed (e.g. when we scale down or simply delete it); pod destroyed -> container destroyed -> data destroyed
- how to have permanent data?
  - use external service - database, object storage (S3), big data node...
  - use Kubernetes volume
- a volume is created outside of a pod
- one volume can be mounted by multiple pods and one pod can mount multiple volumes
- volumes can be:

  - local - attached to a worker node. Accessibility of such volumes depends on the worker node availability
  - dedicated - outside of a worker node; cloud disk (Google or AWS storage); Network file system (NFS)

- Data privacy
  - containers in the same pod cannot see each other's data
  - the same container in different replica cannot see each other's data

### Empty dir volume

- shared data between the containers of a pod (you define which containers can use it)
- this kind of volume is shared on pod's level (another replica cannot see data of another replica pod)
- this volume is not permanent. When a pod is destroyed, it's gone
- convenient for temporary data or application state

- Problematic scenario with empty dir:
  - an empty dir is defined and we have two pod replicas
  - let's say that we an API that stores files and we have a load balancer in front of those pods
  - since an empty dir volume is defined per pod, if an API stores file in one pod (its empty dir), the other pod
    won't be able to see that

### External volume

- Kubernetes persistent volume
- local, network or cloud disk
- control plan also has some volume types
  - configmap
  - secret
  - stored in etcd

### Local volume

- stored on a worker node (disk)
- so, all pods of that worker node can see such volume; replica pods that live in different worker node cannot use it
- if a pod is restarted or destroyed, the data is still there. But, if a worker node dies, the data is gone
- NOTE: minikube does not support this, but Docker Desktop Kubernetes does
- K8s makes sure that the pod will be provisioned at the same node with the local volume
- it survives the pod restart
- `nodeAffinity` - tells Kubernetes where to deploy something (on which node) by some selector criteria

### External disk

- on-prem network storage file server, cloud disk (GCP, AWS, Azure...)
- this approach is resistant to container crash, pod crash, worker node crash
- cons: additional maintenance (if we use an on-prem solution) and more expensive

- Volume `accessModess`

  - on persistent volume `spec.accessModes`:
    1. `ReadWriteOnce` - read and write are allowed by only one pod at a time
    2. `ReadOnlyMany` - multiple pods can perform read operations
    3. `ReadWriteMany` - multiple pods can perform read and write operations

- useful: `kubectl get pod -n <namespace> <pod-name> -o jsonpath='{.spec.containers[*].name}'`
- upon executing `exec` with kubectl, add `-c` to explicitly state on which container the command should be applied

### hostPath volume

- minikube supports volume named `hostPath`
- mounts the host filesystem into a pod
- if we have a multi-node cluster and the pod is restarted for some reason and now lives on another node, then the new pod will not be able to
  the old data
- thus this type of volume works well on a single node cluster
- Minikube is a virtual machine that runs on a laptop, so hostPath will use that virtual machine. To check the file we can go into the VM itself or use `minikube mount` -> `minikube mount <local-path>:<path-in-vm>`

## Application configuration in Kubernetes

### ConfigMap

- we need to configure some application parameters
  - URL
  - database settings
- we need a way to adjust the configuration without need to change the source code
- environment variables can be used (in containers) and we can set them with K8s
- the problem is that some parameters can be dynamically changed
- we can use the configmap and pass it to K8s environment variables
- how to change those parameters? Change the configmap, save it and restart the pod -> `kubectl edit configmap <configmap-name> -n <namespace>`
- configmap is accessible by any container, any pod, any worker nodes in K8s cluster

- Example
  - let's create an HTML page
  - color & text from configmap via env variables
  - no code changes, just the configmap changes
- check the content of a configmap - `kubectl get configmap <name> -o yaml`
- when the configmap is change, we don't have to delete the deployment, we can rollout it -> `kubectl rollout restart deployment -n devops <deployment-name>` -> restart with zero downtime -> K8s will start up the new pod and keep the old one until the new one is ready. Then it will terminate the old pod and redirect traffic to new pod.

- Creating ConfigMap

  1. Declarative configmap file

  - single file
  - separate file

  2. from terminal

- Example: `kubectl create configmap configmap-file-single -n <namespace> --from-file=<configmap-source-file.yml>`
- when passing multiple `from-file` parameter into on configmap, each filename will become one key at configmap and each file content as respective value
- Example: `kubectl create configmap <configmap-name> -n <namespace> --from-file=<source-1> --from-file=<source-2> --from-file=<source-3>`
- a simpler way to specify multiple files as configmap sources is to pack the files into a folder and specify the folder as an argument

### Secret

- like configmap, but the value is always base64-encoded
- it's not actually secret, just not human-readable (which is a bit misleading)
- Example:
  - an HTML page
  - color & some text from secret
  - some text from configmap -> mix the use of secret and configmap
- `kubectl get secret <secret-name> -n <namespace> -o (json or yaml)`
- Creating secret from terminal:

```
kubectl create secret generic <name> -n <namespace> --from-literal <key1>=<value1> --from-literal key.literal.two=<value2>
```

- if the value of a secret is given in plaintext, K8s will encode it to base64
- Example: `kubectl create secret generic secret-literal -n devops --from-literal key.literal.one="This is my secret value for the first key" --from-literal key.literal.two="While this is the secret value for second key"`

- Creating secret from the input file

```
kubectl create secret generic secret-file-single -n devops --from-file=secret-source.yml
```

- Creating secret from multiple files:

```
kubectl create secret generic secret-file-multi -n devops --from-file=secret-source.json --from-file=secret-source.properties --from-file=secret-source.txt --from-file=secret-source.png
```

- a secret can be also created from a folder

## Exposing Kubernetes pod

### Service

- service - exposes pod out of K8s world making it accessible
- Service types:
  - LoadBalancer - distributes traffic among pod replicas
    - access to it by its external IP
    - needs a dedicated IP assigned to load balancer (might incur costs on cloud)
    - Load balancer can use the low number port including 80 (HTTP) or 443 (HTTPS)
    - one load balancer can have multiple ports
  - NodePort - exposes it out of K8s
    - exposes pods via worker node IPs
    - no need for a dedicated IP assigned to a node port
    - NodePort range (default): 30000 - 32767 -> we can assign the port or let K8s automatically assign a free port in this range
    - one NodePort can have only one port
    - good for local development, not practical otherwise
  - ClusterIP - exposes it only to K8s cluster (default type)
    - only for internal access within the cluster
    - client cannot connect to cluster IP service; can do by using kubectl proxy, but the URL will change
    - meant only to expose service to its cluster, so only for debugging purposes
- Example:
  - two different pods ->
    - devops-blue, port 8111
    - devops-yellow, port 8112
  - one deployment => one pod => one container
  - 2 replicas
- To open a random port through which we can access the node port - `minikube service -n <namespace> <service-name>  --url`
- To access ClusterIP, we should open a kubectl proxy -> `kubectl proxy --port=8888`

## Ingress controller

- having many pods means many load balancers
- that means many IP addresses that has to be reserved and paid (in case of cloud deployment), the DNS management
- what if we can have only one IP exposed externally and maybe we can separate pod routes via URL path
- that is `ingress controller` (ingress in K8s is a routing rule book)
- it redirects traffic to services based on a set of rules, e.g. using path
- the service behind can be of any type, but we can easily use ClusterIP services since ingress controller expose them to outer world
- `Ingress`
  - set of rules
  - K8s object
  - defined via configuration file (yml)
- `Ingress controller`

  - installed separately
  - it is a load balancer, but more powerful one with a set of rules (defined in `Ingress`)
  - Nginx, AWS, GCP... have their own ingress controller product

- Routing by host
  - sample: api.devops.com -> 10.142.56.202
  - DNS translate domain name into raw IP
  - cloud: Google Cloud DNS, AWS Route 53
  - locally: host file DNS (`/etc/hosts`)
  - maps the human-readable address to IP address
  - HTTP request header: `Host`
    - URL: `https://somewhere.someday.com`
    - `Host` request header: `someday.com`
  - `blue.devops.local`
  - `yellow.devops.local`

### Ingress over TLS

- secure traffic using TLS/HTTPS
- set TLS on Ingress
- TLS certificate

  - issued by CA
  - self-signed (not trusted by most apps, so we should disable validation on local - browser, postman)
  - tool to generate it - regery

- `kubectl create secret tls api-devops-local-cert --key api-devops.local-privateKey.key --cert api-devops.local.crt`

## Helm - Kubernetes package manager

- Installs, configures, manages applications inside Kubernetes cluster
- Helm has 3 vital components:

1. Chart
   - Helm package format
   - contains and defines all K8s resources required to run an application
2. Repository
   - repository which purpose is to store & share charts
   - public (stable) and self-hosted repositories
3. Release
   - running chart instance
   - one release = one Helm chart instance
   - Running on K8s cluster

- Behind the scene, Helm is K8s application resource and application is container which is pulled down from Docker repository
- https://artifacthub.io/
- `helm repo list` - check added repos
- `helm list` - check releases
- `helm uninstall` - remove release

- Exercise:
  - `minikube addons disable ingress`
  - TODO: complete
