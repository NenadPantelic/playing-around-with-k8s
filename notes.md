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

- image: `timpamungkas/devops-blue:1.0.0`
- `kubectl expose deployment my-devops-blue --type LoadBalancer --port 8111 --name my-devops-blue-lb` - creates a LoadBalancer service
- we can also access the pod if we use the virtual IP that can be fetched with `kubectl describe service <service-name>`
- even if we have multiple replica pods, all of them are access through one service endpoint; the load is dynamically distributed based on the number of available pods
- `kubectl scale deployment my-devops-blue --replicas 3` - to scale deployment to use 3 pods
- `kubectl get pod -o wide` - for a wide output that contains IP addresses
