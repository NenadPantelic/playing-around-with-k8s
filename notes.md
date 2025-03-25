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
- `minikube addons disable ingress`

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

### Sealed secret

- sometime there is a need to share the configuration to version control system (git)
- configmap/secret is just a plain format, meaning if we leave the db password there, secret will leak
- encrypt K8s secret
- not a built-in feature
  - example: bitnami sealed secret
- K8s eventually saves the secret (unsealed, plain text value)
- Example:

  - `my-config-file.yml`
    `animal: Turtle`
  - `my-secret-file.yml` (local file, not in K8s)
    - create secret using parameter `--dry-run` (not uploaded to K8s cluster)
  - `my-sealed-secret-file.yml` (encrypted)
    - create sealed secret using `kubeseal` utility
    - save to be distributed (shared, version controlled...)
    - can only be applied to K8s cluster with the matching secret controller

- `helm upgrade --install sealed-secrets sealed-secrets --set-string fullnameOverride=sealed-secrets-controller --repo https://bitnami-labs.github.io/sealed-secrets --namespace kube-system`
- install bitnami sealed secrets client
- `kubectl create secret generic my-secret -n devops -o yaml --dry-run=client --from-file my-config-file.yml` -- dry run means that the file secret will be encrypted and displayed, but not stored in K8s cluster
- fetch the public key cert and store it - `kubeseal --controller-name=sealed-secrets-controller --controller-namespace=kube-system --fetch-cert > mycert.pem`
- seal the secret - `kubeseal -o yaml --cert mycert.pem < my-secret-file.yml > my-sealed-secret-file.yml`
- the encrypted data is stored, but when we apply the secret, it will decrypted and stored in plain text

## Observability on Kubernetes

### Resource monitoring with Metrics server

- each pod requires resources (CPU, memory)
- we can monitor the resource usage for every pod using the metrics server
- cloud flavors usually comes as a built-in feature, while minikube has an addon
- Instructions:

  1. stop the minikube cluster - `minikube delete`
  2. `minikube start --extra-config=kubelet.housekeeping-interval=10s` (a known issue on minikube at the moment)
  3. `minikube addons enable metrics-server`

- Demo:

  - enable ingress: `minikube addons enable ingress`
  - when it's ready, open a minikube tunnel - `minikube tunnel`

- Commands:
  - `kubectl top node` - monitors the node
  - `kubectl top -n <namespace>` - monitors pods

### Monitoring with Prometheus stack

- vizualization and the historical resource usage
- Prometheus & Grafana
- `kube-prometheus-stack` (includes custom dashboard)
- prerequisites: helm, ingress & metrics-server
- may not work as it should with minikube

```
helm upgrade --install my-kube-prometheus-stack --repo https://prometheus-community.github.io/helm-charts kube-prometheus-stack --namespace monitoring --create-namespace --values values-monitoring.yaml
```

- Nginx + Prometheus + Grafana - monitor Nginx traffic with Grafana
- reconfigure ingress-nginx: `helm upgrade ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --values values-ingress-nginx.yml`
- check the configuration `helm get values ingress-nginx --namespace ingress-nginx`
- `helm upgrade my-kube-prometheus-stack kube-prometheus-stack --repo https://prometheus-community.github.io/helm-charts --namespace monitoring --values values-kube-prometheus.yml`

## Ingress combination

- Example

  1. ingress controller (`https://api.devops.local/...`) with the global nginx configmap (add request and response header)
  2. two pods
     1. Ingress yellow at `/devops/yellow`
        - Nginx yellow annotation -> redirects all traffic to google.com
     2. Ingres blue at `/devops/blue`
        - Nginx blue annotation -> Rate limit (30 request/minute) and HTTP Strict Transport Security (HSTS) header

- `kubectl create secret tls api-devops-local-cert -n devops --key api-devops.local-privateKey.key --cert api-devops.local.crt`
- `helm upgrade --install my-ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace`

- Helm release cannot be restarted, but under the hood, Helm creates a deployment which can be restarted
- Nginx configmap is applied globally, while the annotations are applied only to a particular pod
- Ingress is usually used as a main entry point; use Nginx ingress controller

## Autoscaling & Stateful set

### Horizontal autoscaling

- K8s can create/remmove replica based on threshold (e.g. if CPU usage is over 500 millicores)
- this feature is called Horizontal Pod Autoscaler (HPA)
- prerequisite: metrics-server (to monitor the resource usage)
- Example:
  - let's have 1 to 3 replicas
  - K8s will scale up/scale down based on the CPU metric - it will periodically query the metrics server
  - when the resource usage is above the threshold, it will scale up and opposite, when it decreases below some point, it will scale down
- HPA has a predefined default policy where we only need to set the threshold value
- when the metric value goes above the threshold, K8s immediately scales up
- when the metric value goes below the threshold, K8s waits a few minutes to be sure that that value is not a temporary thing beforing scaling down

### StatefulSet

- Deployment
  - pod replica has a random suffix (`pod-jfa9-ua7s`, `pod-n27f-pl5m`)
  - volume behavior vary (data even can be loss if a pod is recreated, empty dir)
- Use case

  - Pod with fixed naming -> predictable, fixed DNS
  - Pod always uses the same volume
  - Example: RabbitMQ or Elasticsearch cluster

- StatefulSet
  - configuration like Deployment
  - can add a volume template
  - pod name follows a template: pod-0, pod-1, pod-2
    - naming is predictable and always starts from 0
    - pod name is the same even after a restart
    - when deleting, K8s first deletes pod with the highest index in name
  - pod-0 will get volume-0, pod-1 will get volume-1 etc.
  - deleting/scaling down will not delete the volume
- Demo:

  - microservices are rarely combined with the statefulset, since microservice usually refers to REST API, which is by nature stateless (every API request is independent)

- Deployment vs StatefulSet

  - Deployment:
    1. random suffix in pod name
    2. 1 volume, 1 volume claim for all pods
    3. provisioned together
  - StatefulSet:
    1. index (0-based) suffix in pod name
    2. 1 volume, 1 volume claim for each pod
    3. provisioned from the lowest index

- NOTE: when stating multiple arguments in `get` command, use `,` - `kubectl get pod,pvc,pv -n <namespace>`
- standard storage class = dynamic storage provisioning -> a volume will be created when needed (minikube support)
- headless service - a service doesn't even have a cluster IP -> used to provide the network identity for stateful pods
- volume template - used to create a persistent volume claim
- one replica, one volume. If we have three replicas and two volumes defined, standard dynamic storage will provide one more volume for the third replica

- data is distributed among persistent volumes - we cannot retrieve the entire dataset from all volumes
- Example:

  - ingress controller
  - 3 pods
  - 3 volumes:
    - volume-0 (100 data records, replicated) used by pod-0
    - volume-1 (100 data records, replicated) used by pod-1
    - volume-1 (100 data records, replicated) used by pod-2
  - if two volumes fail, the one that is still alive will have the entire data set and newly written records will go there
  - once they are back, the replication will happen and all records will be mirrored to recovered replicas

- `helm upgrade --install my-rabbitmq --repo https://charts.bitnami.com/bitnami rabbitmq --namespace rabbitmq --create-namespace --values values-rabbitmq.yml`

## Quota & Service account

### Resource quota

- each container can have resource quota
  - resource: CPU, memory, storage
- it is an optional config, but it is a good idea to have it to avoid resources being eaten up by a single app

- Two types of quota:
  1. request
     - minimum resource required
     - required for a pod to be alive
  2. limit
     - maximum resource allowed
     - can use only up to a limit (no more than that)

### Namespace quota

- Virtual area which logically splits the entities within a cluster
- pods (and accompanying resources) are logically groupped together to the same namespace
- no fixed standard for the namespace naming
  - environment (dev, test, production)
  - project name
  - team name
- there are two built-in namespaces
  - `default` -> created with the cluster
  - `kube-system` -> pods relevant to K8s system itself
- resource in namespace must be unique, meaning only one "hello-world" deployment in the namespace X
  - we can have multiple deployments with the exact same name, but in different namespaces
- resources can be limited on a namespace level
  - hardware resources
  - K8s resources (pod, configmap, pvc...)
- To create a quota for resource use control we can create a `ResourceQuota` object
- Namespace quota

  - Hardware resources
    | Configuration key | Meaning |
    |--- |--- |
    | `requests.cpu` | Sum of all `resources.requests.cpu` across all containers|
    |`requests.memory` | Sum of all `resources.requests.memory` across all containers|
    | `requests.storage`|Sum of all `resources.requests.storage` across all containers |
    | `limits.cpu`|Sum of all `resources.limits.cpu` across all containers |
    | `limits.memory`| Sum of all `resources.limits.memory` across all containers|
    | `limits.storage`|Sum of all `resources.limits.storage` across all containers |

  - Kubernetes resources
    | Configuration key | Meaning |
    |---|---|
    |configmaps | Total number of `ConfigMap` in a namespace |
    | persistentVolumeClaims | Total number of `PersistentVolumeClaim` in a namespace|
    | pods |Total number of `Pod` in a namespace |
    | replicationcontrollers | Total number of `ReplicationController` in a namespace|
    |resourcequotas | Total number of `ResourceQuota` in a namespace|
    | services | Total number of `Service` in a namespace|
    | services.loadbalancer| Total number of `Service` of type `LoadBalancer` in a namespace|
    | services.nodeports | Total number of `Service` of type `NodePort` in a namespace|
    | secrets| Total number of `Secret` in a namespace |

### Service account

- There are two types of users/accounts in Kubernetes

1. User account (human)
   - use `kubectl`
2. Service account (non-human)
   - authenticate from a pod within a cluster
   - service account is Kubernetes object
   - mounted in a pod for communication
   - specific per namespace
   - each namespace has a default service account

## Secure pod & repository

### Security context

- Least privilege principle
  - users/applications should only have the necessary privileges to complete their tasks
- Security context - defines privileges/permission for individual pods/containers
- Add `securityContext` within a pod or container configuration (or both)
- If `securityContext` defined on both, the one on container will take the priority
- this possibility (of having a pod-level security context) is useful when having multiple containers in pod

  - the default one can be defined on a pod level
  - override on a container level

- Security context example

```
securityContext:
  runAsNonRoot: true
  runAsUser: 10000
  runAsGroup: 10000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
```

- `runAsNonRoot`
  - set to true to enforce the use of non-root users for pod/container to limit the access to resources that might mistakenly get exposed to the container
  - applicable to: pod/container
- `runAsUser` and `runAsGroup`
  - to enforce specific the specific runtime user and group
  - these IDs must exist in the container image
  - applicable to: pod/container
- `readOnlyRootFilesystem`
  - set to true whenever possible
  - restricts the use of filesystem to read-only, which prevents the attacker to install malicious software of change configurations
  - applicable to: container
- `allowPrivilegeEscalation`
  - when set to true, container will run in a privileged mode
  - processes in privileged containers are essentially equivalent to root on the host
  - the default value is **false** and it is not recommended to change it
  - this config controls whether a process can gain more privileges that its parent process
  - in most cases we should explicitly set value to false to prevent processes from attaining higher privileges, for example via sudo command
  - applicable to: container
- `capabilities`
  - kernel level permissions that allow for more granular controls, which include system-wide administration functions
  - K8s provides a way to drop or add capabilities
  - drop all capabilities, then only add back the ones your application needs
  - in many cases, applications don't really need any capabilities, so you can drop them all, test the application and if needed, add just the ones which are needed
  - applicable to: container
  - `procMount`
    - don't change **procMount** from the default settings, unless you have a very specific requirement like nested containers
    - applicable to: container
  - `sysctls`
    - don't change **sysctls** from the default settings, unless you have a very specific requirement
    - can destabilize the host operating system
    - applicable to: pod

### Private repository

- Steps for using a private docker repository:

1. Create a namespace - `kubectl create namespace <namespace>`
2. Create a secret - `kubectl create secret -n <namespace> docker-registry dockerhub-secret --docker-server=https://index.docker.io/v1` --docker-username=your-username --docker-password=your-password --docker-email=your-email@email.com`

## Using and creating Helm charts

- Helm chart

  - generic template
  - create templates for K8s - all parameters are generic as template values
  - package the template as Helm chart
  - on release we define value or use default value for those parameters
  - install resource using a single command
  - Helm charts are versioned, so they can be updated
  - each application type has a different template, i.e.
    - Python app has a different template from a Java app template
    - Java application written in framework A will have a different template than the Java app written in framework B

- `helm create my-chart` -> Helm will create several required files and folders

```
[folder] my-chart

  [folder] templates

    _helpers.tpl - internal helper file, no need to change it

    deployment.yaml -> Kubernetes configuration template. Can be customized/added (e.g. with variables). Use Helm template language (based on Go) with the template variables. When running `helm install`, those variables are populated and K8s objects are created
    hpa.yaml
    ingress.yaml
    serviceaccount.yaml

  Chart.yaml - contains metaata for our Helm chart: name, description, version...
  values.yaml - contains key-value pairs to be used as parameters in Helm release; can be overriden upon release creation. These kv pairs will be eventually used on yaml files under folder `templates`
```

- to create a Helm release and use a local folder

#### Check the template file and injected values

- go to folder where the chart is located (chart folder) - `helm-charts`
- execute Helm installtion using a path to values file - `values-spring-boot.yml`
- what if template/values file has a mistake
- we can render template + values without creating a release -> `helm template`

- Template command: `helm template helm-yellow-01 spring-boot-rest-api --namespace devops --create-namespace --values /home/nenad/Documents/Learning/kubernetes-istio-google-cloud/examples/kubernetes/024-helm-spring-boot-rest-api-01/values-spring-boot.yml`
  - if we want to save the output files of this command
  - add `--output-dir <output-dir-location->`

#### Simulate/validate

- The YAML structure may be correct, but executing the release on K8s may give some errors
- we can execute a dry run - `kubectl apply --dry-run=client -f xxx.yml` (the argument file can be creat )
- it simulates the installation without actually applying the config file

#### Run the release

- `helm list -n devops`
- `helm upgrade --install helm-yellow-01 spring-boot-rest-api --namespace devops --create-namespace --values /home/nenad/Documents/Learning/kubernetes-istio-google-cloud/examples/kubernetes/024-helm-spring-boot-rest-api-01/values-spring-boot.yml`
- `helm uninstall -n devops helm-yellow-01`

### Template syntax

![](images/helm_template_syntax_1.png)<br>
![](images/helm_template_syntax_2.png)<br>
![](images/helm_template_syntax_3.png)<br>

### Chartmuseum

- we can (re)use charts using Helm chart repository
- we can use the cloud storage (AWS S3, GCP)
- we can also use the chartmuseum
  - can use various storages (local, cloud)
- `helm install my-chartmuseum --namespace chartmuseum -f values-chartmuseum.yml chartmuseum/chartmuseum`
- Navigate to folder where the chart is and run `helm package spring-boot-rest-api`. This will create a tar file.
- we can upload a tgz file to chartmuseum using an API

```
curl --location 'http://chartmuseum.local:8080/chartmuseum/api/charts' \
--header 'Content-Type: application/octet-stream' \
--header 'Authorization: Basic Y2hhcnRtdXNldW06cGFzc3dvcmQ=' \
--data-binary '@filepath/spring-boot-rest-api-0.1.0.tgz'
```

- now we can use that chart - `helm upgrade --install helm-yellow-02 spring-boot-rest-api --repo http://chartmuseum.local:8080/chartmuseum --username chartmuseum --password password --namespace devops --create-namespace --values values-spring-boot.yml --version 0.1.0`

### Multiple configurations

- we can use multiple-value file:

  - one for the general release (values.yml)
  - one for each environment (`values-dev.yml`, `values-test.yml`, `values-prod.yml`)
  - pass multiple files on release

- `helm upgrade --install helm-blue-03 spring-boot-rest-api --repo http://chartmuseum.local:8080/chartmuseum --username chartmuseum --password password --namespace devops --create-namespace --version 0.1.0 --values values.yml --values values-dev.yml`
- if there is the same key in multiple values, the last given file has the priority

### GitHub as Helm Chart repository

- use public or private repository

### Multiple Helm charts

- we might need multiple charts as a single unit
- image we have a service S that requires services A and B and Redis
- S should run on K8s using Helm
- Create a Helm chart for S
  - it will not have any K8s objects
  - instead, it will declare dependencies to A, B and Redis
  - S will then override Helm chart values (with values files)
- S will contain the `Chart.yaml` file, but no object templates
- we can create only `Chart.yaml` without using Helm

- Demo:
  - go to `helm-spring-boot-rest-api-04/chart-with-dependencies` and run `helm dependency update` (will download charts as tgz files)
  - `helm upgrade --install helm-blue-04 chart-with-dependencies --namespace devops --create-namespace --values values.yml --values values-dev.yml` (install charts)

## GitOps

- at some point there is need for transition between the engineering and devops team - when the deliverable is released
- build + push of a Docker image can be automated -> CI pipeline (Continuous integration)
- automatic deployment -> CD (Continuous deployment/delivery)

- GitOps

  - an engineering team updates an image & predefined values and pushes the changes
  - once chart & values are on GitHub, the devops team takes care of pulling the updated values and deploy these to K8s via Helm

- Development procedure

  1. an engineer has a task
  2. he/she creates a git branch - `feature`
  3. makes some commits and pushes them to `feature` branch
  4. creates a pull request to `main` branch
     --- CI pipeline runs ----
     Lints (formats) the source code
     Compiles
     Do static code analysis
     Do security scan
     Checks the code quality gate
     Runs unit tests
     ***
  5. Code review & discussion
  6. Merged to master
     --- CD pipeline runs ---
     Builds a Docker image
     Pushes it to repository
     Deploy the container on a server (Docker or K8s)

- GitHub repository
  - CI pipeline runs here
  - CD pipeline runs here
  - CD pipeline does not deploy images to the server
  - CD pipeline updates metadata in the application deployment repo
- GitOps
  - monitors the repo - if it changes, that's a new target state. The idea is to transform the K8s cluster from an actual state to target state
  - Synchronize any change to K8s cluster, using GH repo as a source of truth
- GitOps = git PR + CI/CD + IaC (Infrastructure as code)
- GitOps tool - ArgoCD
- `minikube start --memory 5000 --cpus 2`

- `helm upgrade --install my-argocd argo-cd --repo https://argoproj.github.io/argo-helm --namespace argocd --create-namespace --values values-argocd.yml`
- Credentials: admin/password
- `values-xxx.yml`
  - separate per environment, keep the structure
  - `xxx` is environment (`values-dev.yml`, `values-test.yml`, `values-prod.yml`)
  - the same key can have different value (e.g. database url, username/password, API key,...)
  - generic values can be put to `values.yml`
- what if someone deletes the object monitored by ArgoCD?
  - that means that the desired is not the same as the actual one
  - the same goes if some config parameter is changed
  - ArgoCD will synchronize and make the cluster go to the desired state
  - if an automatic sync feature is turned off, then it will do nothing since it will not automatically synchrnoize the state (it can be manually synchronized)

### Kubernetes CRD (Custom Resource Definition)

- built-in K8s object definition (deployment, pod, ingress...)
- application might need to use resources for the custom requirement
- use CRD (Custom Resource Definition)

- ArgoCD and sensitive data
  - `helm create sealed-secret`
  - `kubeseal --controller-name=sealed-secrets-controller --controller-namespace=kube-system --fetch-cert > mycert.pem` (need a sealed-secrets Helm chart for this)
- if we change a secret, ArgoCD will update Helm for sealed secret, but that will not be treatened as an application change, therefore an application will not be synchronized and it has to be manually restarted
- a solution is to use a tool called `reloader`

### Reloader

- reloader will monitor changes in configmap and sealed secrets and will update the deployment/pods (restart them when the changes are detected)
- simple case
  - configmap & sealed secret
- `helm upgrade --install my-reloader reloader --repo https://stakater.github.io/stakater-charts --namespace reloader --create-namespace`
- Seal a secret: `kubeseal --controller-name=sealed-secrets-controller --controller-namespace=kube-system --fetch-cert > mycert.pem`
- `kubeseal --cert mycert.pem -o yaml < devops-reloader-secret-plain.yml > devops-reloader-secret-sealed.yml`
- Apply the sealed secret and configmap
  - `kubectl apply -f devops-reloader-secret-sealed.yml`
  - `kubectl apply -f devops-reloader-configmap.yml`

## Managing certificates on K8s

### Cert manager

- secure HTTP using TLS (HTTPS protocol)
- K8s cert manager
  - adds certificate & certificate issuer to K8s
  - simplify the TLS usage & renewal
  - various issuers supported (e.g Let's encrypt)
  - ensures the certificate validity
  - renews certificates before they expire
- does not have any vital effect to local K8s cluster - everything is in localhost
- it requires a registered domain for certificate validation
- Install cert manager: `helm upgrade --install cert-manager cert-manager --repo https://charts.jetstack.io --namespace cert-manager --create-namespace --set installCRDs="true"`
- Cert Manager CRD

  - provides CRD (Custom Resource Definition)
  - **Issuer**
    - how cert manager requests the TLS certificate
    - specific for a namespace
    - ensure that the `Issuer` is in the same namespace as TLS certificate
  - **ClusterIssuer**
    - cluster-wide version of `Issuer`
    - one setup for all namespaces
  - `Certificate`
    - defails of TLS to be requested
    - refer an `Issuer`/`ClusterIssuer`

- ACME protocol (Automated Certificate Management Environment)
  - protocol to automate the certificate lifecycle - issuance and renewal
  - free to use
  - little time to configure
  - Let's Encrypt as ACME
  - ACME entity has to validate that we own the domain, so if the domain is not reachable over the Internet (not a public domain), cert will not be deployed successfully
  - generally, cert manager creates a token and exposes it, so it can be validated by ACME
  - still, the certificate will be issued (will have an invalid mark on browsers)

## Istio Service Mesh for East-West traffic

### East-West traffic

- North-south traffic: client outside the cluster and K8s cluster (client <-> ingress <-> service(pod))
- East-west traffic:
  - sometimes one K8s service (pod) has to call another service (pod)
  - client <-> ingress <-> service A <-> ingress <-> service B
  - an alternative, why would not service A call directly service B without going through an ingress object, i.e.
    - client <-> ingress <-> service A <-> service B; this is called the east-west traffic
  - the question is which address calls service A when reaching service B?
    - each service is assigned an IP address
    - each service is also assigned a domain - `service-name.namespace.svc.cluster.local`
- Example:
  - 3 services: blue, yellow and white
  - main entry (north-south via ingress): blue
  - API chain calls:
    - `/api/chain/{response-status-code}/one` -> blue calls yellow
    - `/api/chain/{response-status-code}/two` -> blue calls yellow and blue calls white
    - `/api/chain/{response-status-code}/three` -> blue calls yellow and yellow calls white
    - `/api/chain/{response-status-code}/four` -> one-level call
    - `/api/chain/{response-status-code}/one` -> two-level call

#### Automatic service DNS:

- service:
  - name: `devops-yellow-svc`
  - port: 8112
  - namespace: devops

| Recognized DNS                                    | caller (blue) in the same namespace | caller (blue) not in the same namespace |
| ------------------------------------------------- | ----------------------------------- | --------------------------------------- |
| `devops-yellow-svc.devops.svc.cluster.local:8112` | ✔️                                  | ✔️                                      |
| `devops-yellow-svc.devops.svc:8112`               | ✔️                                  | ✔️                                      |
| `devops-yellow-svc.devops:8112`                   | ✔️                                  | ✔️                                      |
| `devops-yellow-svc:8112`                          | ✔️                                  | ❌                                      |

- service:
  - name: `devops-white-svc`
  - port: 8113
  - namespace: devops

| Recognized DNS                                   | caller (blue/yellow) in the same namespace | caller (blue/yellow) not in the same namespace |
| ------------------------------------------------ | ------------------------------------------ | ---------------------------------------------- |
| `devops-white-svc.devops.svc.cluster.local:8113` | ✔️                                         | ✔️                                             |
| `devops-white-svc.devops.svc:8113`               | ✔️                                         | ✔️                                             |
| `devops-white-svc.devops:8113`                   | ✔️                                         | ✔️                                             |
| `devops-white-svc:8113`                          | ✔️                                         | ❌                                             |

### GKE

- `gcloud auth login` - to login with gcloud CLI

#### Observability and control

- one failure in the east-west traffic can cause an error to the entire east-west call chain (cascading failure)
- what if we have a problem with a particular service
  - a problem that cannot be fixed or a security hole
  - we should disconnect it
  - disconnecting it may not be easy - e.g. we have communication from node P to node Q and we want to close that traffic in east-west traffic, but leave open the north-south traffic calls to Q
- Tools:
  1. kube-prometheus stack - `helm upgrade --install kube-prometheus-stack --repo https://prometheus-community.github.io/helm-charts kube-prometheus-stack --namespace istio-system --create-namespace --values values-kube-prometheus.yml`
  2. nginx ingress controller - `helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace --values values-ingress-nginx.yml`
  3. istio base (CRD) and istio deamon (istiod)
  4. enabling sidecar injection
  5. Kiali as Istio UI
  6. Jaeger as distributed tracing UI

### Istio Service Mesh

- ingress traffic (north-south): clients to our data center; managed by ingress controller
- egress traffic: our data center to external APIs/resources
- internal traffic (east-west): communication between resources/services within a cluster/data center; use service mesh

- East-west problem
  - no HTTPs when services communicate between each other
  - no retry/failover when service calls fail
  - no metrics, no visibilty over the communication quality
  - no access control, all services can communicate between each other
  - simple routing (e.g. round robin). If we have 3 instances of service B, calls to service B will be routed in a simple way
  - these functionalities can be added to each service, but that will make some effort to an engineering team
- `Service mesh` to the rescue
- the idea is to route all calls through the service mesh rather than directly calling the service/pod
- Mesh can intercept the traffic and add some logic (encryption, retry, timeouts, metrics, logs...)
- Instead of `service A <-> service B`, we have `service A <-> mesh <-> service B`
- Mesh logic is pluggable, no coding needed

- Service mesh abilities:
  - observe & monitor the communication
  - secure connection
  - circuit breaker
  - traffic management/routing (service A v1 calls service B v2 etc.)
  - enables A/B testing or canary deployment
- No code change is needed, Istio helps with that by using sidecar proxy approach
- pods will have the service container + sidecar proxy container
- traffic between microservices will then go through each microservice's sidecar proxy, instead of direct communication between microservices (pod service applications). So, we can gather metrics, because proxy containers are intercepting traffic on both sides;
  - instead of `service A -> service B` we will have `service A -> sidecar proxy A -> sidecar proxy B -> service B`
- Istio is using `envoy` as sidecar
- all sidecar proxy containers make the service mesh **data plane**
- another part is the **control plane** which provides logic and config for the sidecar proxy
- control plane is using `istiod` (Istio deamon)
- Installation:
  - Istio base (CRD):
    ```
    helm repo add istio https://istio-release.storage.googleapis.com/charts
    helm repo update
    helm upgrade --install istio-base istio/base --namespace istio-system --create-namespace
    ```
  - Istio daemon - `helm upgrade --install istiod istio/istiod --namespace istio-system`
- built-in Istio provides port 15090 where Istio metric can be retrieved

### Enabling sidecar injection

- add a label to a namespace where we want to enable the sidecar proxy
- it should be done once for each namespace
- add this label - `istio-injection: enabled`
- **NOTE:** sidecar injection will work only for pods created after the sidecar proxy enablement
- to get the sidecar proxy container in other pods, restart the deployment - `kubectl rollout restart deployment...`
- prometheus path: `http://<kube-prometheus-stack-prometheus-cluster-ip/prometheus/graph`

### Kiali

- visualize Istio
- gathers Istio data & visualize it as graph (connection between nodes, traffic flow...)
- Kiali allows also some config for Istio (includes the node disconnection), so access to it should be put behind the credentials; by default use the service account token to secure itself
- Kiali takes data from Prometheus, so `PodMonitor` and `ServiceMonitor` must already be in running state to scrape data used by Kiali
- Install Kiali: `helm upgrade --install kiali-server kiali-server --repo https://kiali.org/helm-charts --namespace istio-system --create-namespace --values values-kiali-server.yml`

## Distributed tracing

- we have a call chain
  - we got the 413 response (payload too large)
  - we got the good response, but it took 20s
- to trace the call chain, we can have some correlation ID which every service will generate, so a request to that service that is part of a particular call chain can be tracked; we can then easily associate the response status code and latency to every request in that call chain
- two common tracing tools are Zipkin and Jaeger
- Kubernetes operator
  - complementary to Helm chart
  - similar functionalities
- Jaeger's Helm chart is available, but requires a lot of resources (local cluster might not be enough)
- K8s application can become more complex
- the challenge is customizing Helm yaml template
- instead, we can use K8s operator
- Helm chart is usually enough

- Install Jaeger K8s operator

```
kubectl create namespace observability
kubectl create -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.65.0/jaeger-operator.yaml -n observability
```

### Header propagation

- HTTP request header: `x-b3-traceid|x-b3-spanid|x-b3-parentspanid`

  - Trace id is formed from one or more spans
  - One span is called to one node; each span will have a unique id and parent span id

- Generated by code & passed along all nodes
- For an example, if we had a call chain going through three services, we would have:

  - Service A: t-11 | s-85 | null
  - Service B: t-11 | s-86 | s-85
  - Service C: t-11 | s-87 | s-86

- Another example, service A calls service B and service C
  - Service A: t-11 | s-85 | null
  - Service B: t-11 | s-86 | s-85
  - Service C: t-11 | s-87 | s-85
- Tracer

  - implement tracer in code
    - `opentracing.io`: older, already in archived state
    - `opentelemetry.io`: newer, suggested

- Image: 1.0.0 - no open tracing
- Image: 1.0.1 - with open tracing
- Jaeger is newer than zipkin and compatible with it
- sending trace/span to Zipkin (while using Jaeger) menas sending to zipkin-compatible Jaeger receiver
- usually port 9411
- Trace = journey, span = a milestone on that journey
- Sidecar proxy interceptor also makes some impact on the latency, but that is very small, especially compared to the value it gives us
- Opentelemetry does not require any code change -> an Opentelemetry agent is run along the code to push metrics
  - also there is an Opentelemetry sidecar proxy
- Install OT: `kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml` (requires cert-manager)
- Istio is needed for Opentelemetry and Jaeger

#### Metrics

- Effect of annotations
  - `prometheus.io/scrape: "true"`
  - `prometheus.io/port: "8111"`
  - `prometheus.io/path: /.../actuator/prometheus
- Prometheus hits XXX (on sample above `/.../actuator/prometheus`)
- Application responsibility to provide that an endpoint
- Grafana queries metrics and visualize them
- Istio collector -> Prometheus endpoint to scrape metrics
  - `podMetricsEndpoint`:
    - path: /stats/prometheus
    - port: "15020"
- Taffic flows through the sidecar
- There are two types of metrics - Java app prometheus & Istio sidecar prometheus
- How Prometheus gathers these two metrics

- Istio exposes `stats/prometheus`; port: 15020
  - Istio metric 1, Istio metric 2....
- Application exposes: `app/prometheus`; port 8080

  - App metric 1, App metric 2, App metric 3, ...
  - with the following annotations:
    - `prometheus.io/scrape: "true"`
    - `prometheus.io/port: "8080"`
    - `prometheus.io/path: "/app/prometheus"`

- These metrics are gathered and Istio exposes them at: port = 15020, URL = stats/prometheus
- Secure Kiali server: `helm upgrade --install kiali-server kiali-server --repo https://kiali.org/helm-charts --namespace istio-system --create-namespace --values values-kiali-server-secure.yml`

## Traffic management using Istio & Envoy proxy

### Traffic routing

- what if we have two versions of the same application - `1.0.0` and `1.0.1` running at the same time between the same service
- routing rule can be:

  - if a request contains `x-redirect-to` = `pod-white-1.0.1` header then we route the request to 1.0.1 version
  - otherwise, the request should go to `1.0.0`

- Istio traffic routing
  - it uses `DestinationRule` & `VirtualService`
  - these two are K8s objects
  - we can use Kiali wizard to auto-generate YAML definitions and apply them; we can adjust those auto-generated definitions
- For this configuration to work, we have to add:
  - istio sidecar proxy to nginx pod
  - certain annotations on the ingress
- Istio objects

  - `Gateway`: load balancer/ingress
    - no need for it when we have nginx controller
  - `VirtualService`: traffic routing rules, how the traffic will flow within the service mesh
  - `DestinationRule`: policies to be applied to traffic

- Imagine we have a situation where we are sending a package from Tokio to London
  - the logistics company takes care of that
    - an airport - `Gateway`
    - routing directions (whether it goes to Liverpool, Manchester or London once it arrives to England) - `VirtualService`
    - logistic policies (which type of transport to use, should they rotate vans used to deliver packages, the delivery protocol) - `DestinationRule`
- these definitions are pushed to Envoy proxy

### Load balancer

- distributes traffic
- LB algorithms:
  - Round Robin (Simple)
  - Least request/least connection - aims the pod replica which is least busy; it serves less number of clients
  - Sticky load balancing (ConsistentHash) - the request will go to the same pod if the request value is the same. Supports hash by: HTTP header, cookie, source IP, query parameter

### Canary release

- releasing new features only to a subset of users
- route x% traffic to old feature + y% traffic to new feature
- x: 90%, y:10% (maybe a new feature is not stable yet)
- how it gradually becomes more and more stable, we reduce the old feature traffic to 80%, 70%... and increase the y% towards 100%
- Canary can be done without Istio
  - we would have a single service and v1 pods (old feature) and v2 pods (new feature)
  - e.g. if we want to route 25% to new feature, we would need 4 pods behind a service, 3 pods v1 and 1 v2 pod
  - service would then apply the Round robin routing
  - what if we want 10% of traffic to go to new feature pod v2 -> we would need 10 pods, 9 v1 and 1 v2 pod
  - problems
    - too many pods - provisioning idle pods just for Canary percentage
    - low traffic means some pods are underutilized
    - resource usage is too big
- Istio needs only one pod of each version and by using the `VirtualService` and `DestinationRule` objects it splits the traffic as wanted
- just tell it the percentage share and it will do it
- Istio traffic distribution seen in Kiali may not be 100% precise, since it gets data from Prometheus and gathering/rendering them might face some delay

### `VirtualService`

- Kubernetes object
- Example:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-virtual-service # Name of the Istio VirtualService resource
spec:
  hosts:
    - "example.com" # Host for which this VirtualService configuration applies
  gateways:
    - "my-gateway" # Gateway to associate with this VirtualService
  http:
    - route:
        - destination:
            host: "my-service" # Destination service for this route
            subset: "v1" # Subset of the destination service
          weight: 80 # Weight for this route (80%)
        - destination:
            host: "my-service" # Destination service for this route
            subset: "v2" # Subset of the destination service
          weight: 20 # Weight for this route (20%)
```

- K8s service and `VirtualService` are not the same thing
- these are like partners; `VirtualService` is a partner whose task is to split traffic

### `DestinationRule`

- `VirtualService` defines **where** the traffic goes
- `DestinationRule` defines **how** it goes there
- these two can work together, but that is not a must -> `VirtualService` needs the `DestinationRule`, while the `DestinationRule` can work without the `VirtualService`

- Example:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  # Destination service for which this rule applies
  host: my-service

  subsets:
    - name: v1 # Subset for the first route in the VirtualService
      labels:
        version: "v1"
    - name: v2 # Subset for the second route in the VirtualService
      labels:
        version: "v2" #label must match the pod labels

  trafficPolicy:
    retries:
      # Number of retry attempts
      attempts: 3
      # Timeout for each retry attempt
      perTryTimeout: 2s

    connectionPool:
      tcp:
        # Maximum number of TCP connections
        maxConnections: 100
        # Timeout for establishing a TCP connection
        connectTimeout: 2s

  outlierDetection:
    # Number of consecutive errors before considering an instance as unhealthy
    consecutiveErrors: 5
    # Time interval between outlier detection checks
    interval: 30s
    # Minimum ejection duration for an unhealthy instance
    baseEjectionTime: 60s
    # Maximum percentage of unhealthy instances allowed
    maxEjectionPercent: 30
```

### Dark launching

- Canary is all about percentage
- what if we want to rearrange the traffic distribution so that v2 API pod serves clients between the age 30-40
- we cannot predict the percentage, nor we can apply such logic in plain Canary
- if we know the user age (they have to log in); then we apply the dark launching logic
  - instead of saying: "Hey Istio, please redirect 50% of traffic here, and 50% there" we can say
  - redirect the traffic based on the `x-target-market` header - true to v2 API pod, false or not exist to v1 API pod
  - we can set this header upon logging in
  - the user is not aware that we are testing a new feature - that's why it's called the dark launch
  - Istio supports using HTTP headers, URI, scheme, method, port, query parameters as the decision parameters in dark launch
  - it also supports the TCP route criteria matching

### Fallacies of Distributed system

- one request can contain multiple east-west requests (microservice to microservice, call to database, cache, file storage...)
- nodes can be distributed to many machines
- the entire system must be reliable according to our SLA

- Fallacies of distributed systems

1. Network is always reliable
   - network communication can fail (it can fail even without we know that)
   - reasons: target server is down, internet provider failure, broken hardware, something else...
   - application has to deal with this
   - retry failures, e.g. transactional outbox pattern (store an attempt and retry unprocessed requests)
2. Latency iz zero
   - Latency is always there, even it is very small (1ms) or bih (n seconds/minutes)
   - Latency = time needed for a service to send a request and get a response; well it's not zero
   - related to the network speed & source-to-target distance
   - we can use CDN (Content Delivery Network) to make the target closer to caller by storing a duplicate of data at the server which is part of CDN
3. Bandwidth is infinite
   - Bandwidth is limited and might be shared
   - it is measured in bps (bits per second)
   - it's like a pipe in which the water flows
     - the same pipe might be shared between many water sources
   - if we have a 1 gigabit/s bandwidth, that doesn't mean that our service can use the whole bandwidth, but the network admin can allocated 100 megabits/s to an application and rest is shared
   - an example is a 1-hour video. Netflix does not send it all at once, because that could take 4 GB, but rather it sends it in smaller chunks - streaming
4. The network is secure
   - security hole might be there, from various source
   - we might not know about those holes yet (ever)
   - OS, library, application, hardware security holes
   - use tools to scan CVE (Common Vulnerability Exposure) and fix it
   - use encrypted communication (e.g. TLS)
   - penetration testing team
   - bug bounty
   - maybe future technology can exploit, but not right now - we need a routine check
5. Topology doesn't change
   - nodes/applications in topology may be added, removed, changed
   - the network structure can change - e.g. we add a firewall in front of an application, change the configuration, we remove a node...
   - there is no an easy way to create a system completely resilient to this
6. There is one administrator
   - several administrators with the same privilege may cause conflicting, undocumented changes which can break the system
   - usually on infrastructure - e.g. John allocated an IP range for a service, while Lisa allocated an IP range for service B which conflicts (intersect) with the one that John chose, these services can then behave unexpectedly
   - these changes are not documented and it's hard to trace who did them -> we need a way to manage & document the infrastructure configuration
   - we can use IaC (Infrastructure as code) for this purpose, e.g. Terraform (config file can be subject to code review); it also ensures that you provision and configure the same environment every time
7. Transport cost is zero

   - transporting data (both in and out) costs money
   - in cloud the database is free, but the data costs money (data storage and transport)
   - ingress cost (into our network) and egress (from our network)
   - small cost per GB, but accumulated -> TBs or PBs
   - be aware of network cost - e.g. a picture might sized down when sending data from a phone to server
   - a JSON response will have the bigger data size compared to binary protobuf or avro, yet these two are less common in REST API responses and we have to satisfy the needs of our clients (if that's a public API)

8. The network is homogeneous
   - various network nodes, so make system that has high interoperability
   - e.g. create applications that are platform independent (of a filesystem e.g. - windows vs linux)

### Fault injection

- use Istio to test some fallacies
- purposely cause system to misbehave
- application developer gets a simulation result from Istio and so he/she can improve an application resiliency
- easily add/remove/configure failure from Istio
- Examples:
  - Network is reliable -> terminate a request without passing it to application (502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout, 429 Too Many Requests)
  - Latency is zero -> adding some delay to service response; high latency due to network issue, busy service, high I/O
