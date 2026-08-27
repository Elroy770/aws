<!-- Page 1 -->

# ‫לוחמים להייטק‬


### Diagram / Visual Elements:
- 7o'n? DANI?

<!-- Page 2 -->

# CKA-Certified Kubernetes

Administrator

<!-- Page 3 -->

# Agenda

- Exam Overview
- Core Concepts
- Kubernetes Architecture
- Workloads
- ConfigMaps & Secrets
- Networking Intro.
- Services
- Cluster Monitoring

### Diagram / Visual Elements:
- a,
- Cd
- * ConfigMaps & Secrets

<!-- Page 4 -->

# Exam Overview

<!-- Page 5 -->

# CKA Exam

What is the Certified Kubernetes Administrator (CKA) Certification?
The Certified Kubernetes Administrator (CKA) certification is designed to ensure that certification holders have the skills,
knowledge, and competency to perform the responsibilities of Kubernetes Administrators.
The CKA certification allows certified administrators to quickly establish their credibility and value in the job market, and also
allowing companies to more quickly hire high-quality teams to support their growth.
Exam Details
- The exams are delivered online and consist of performance-based tasks (problems) to be solved on the command line running
Linux.
- The exams consist of 15-20 performance-based tasks.
- Candidates have 2 hours to complete the CKA exam.
- The exams are proctored remotely via streaming audio, video, and screen sharing feeds..
Currently, the exam is on Kubernetes version 1.34

### Diagram / Visual Elements:
- CS

<!-- Page 6 -->

# Monolithic vs. Microservices Architectures

CKA candidate
handbook

### Diagram / Visual Elements:
- CKA Curriculum
- Storage
- * Implement storage classes and dynamic
- volume provisioning
- * Configure volume types
- and reclaim ps
- Manage persistent volumes and per
- volume claims
- Workloads and Scheduling
- * Configure workload autoscaling
- * Understand the primitives used to create
- robust, on deploy-
- ments
- * Co Vand scheduling
- finity, etc.)
- Servicing and Networking
- * Understand connectivity between Pods
- * Define and enforce Network Policies
- * Use ClusteriP, NodePort, Loz
- jalancer
- 1s and endpoints
- * Use the
- traffic
- ay API to manage Ingress
- * Kno
- Ing
- to use Ingress controllers and
- * Understand and use CoreDNS
- Installation ai
- oubleshooting
- Troubleshoot clusters and nodes
- Troubleshoot ¢ com
- ponents
- and applic:
- nage and evaluate container output
- services and networkir
- - Cluster Architecture,
- d Configuration
- Man: trol (RBAC)
- role based acces
- Prepare underlying infrastructure for installing
- aku
- netes clus!
- Create and manage Kubernetes clusters us-
- ing kubeadm
- Mani
- the lifecycle of Kuberr
- s clusters
- d configure a highly-available
- e Helm and Kustomize all cluster
- component
- Understand extension interfaces (CNI, CS!
- CRI, etc)
- xt
- Understand CRDs, install ai
- d configure
- operators

<!-- Page 7 -->

# General Concepts

<!-- Page 8 -->

# Monolithic vs. Microservices Architectures

A monolithic architecture is a traditional software development model that uses one code base to perform multiple functions.
All the software components in a monolithic system are interdependent due to the data exchange mechanisms within the system.
It’s restrictive and time-consuming to modify monolithic architecture as small changes impact large areas of the code base.
In contrast, microservices are an architectural approach that composes software into small independent components or services.
Each service performs a single function and communicates with other services through a well-defined interface.
Because they run independently, you can update, modify, deploy, or scale each service as required.

### Diagram / Visual Elements:
- la?)
- Ul
- 6 wicroservice | Microservce [s]
- Data Microservice | iu
- Business
- ie,
- Logic
- Layer S| | S| |S| |S
- Microservice | |_Microservice
- 1D
- 1D

<!-- Page 9 -->

# VMs vs. Containers

A VM is an isolated computing environment with its own CPU, memory, network interface, storage, and operating system,
emulating physical computers. Multiple VMs can run on a single server, with a hypervisor acting as a lightweight software layer
positioned between the physical host and the VMs. This hypervisor efficiently manages access to resources, allowing VMs to
function as distinct servers for greater flexibility and agility.
By placing many VMs on each physical server, traditional virtualization technology can make better use of hardware, leading to cost
savings. This orchestration abstracts physical resources (typically compute, network, and storage) so users can access them
through software.
A container is a standard unit of software that packages up code and all its dependencies, so the application runs quickly and
reliably from one computing environment to another. A Docker container image is a lightweight, standalone, executable package of
software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.
Unlike a virtual machine, a container doesn’t require a guest operating system. Most modern applications are made of multiple
containers that each perform a specific function.
Compared to VMs, containers are typically smaller (measured by the megabyte). Their smaller size makes them a faster and more
agile way to scale to match changes in demand.
Ubuntu server ISO file size is about 6 GiB. Ubuntu container image is about 190 MiB.

### Diagram / Visual Elements:
- G2 VMs vs. Containers

<!-- Page 10 -->

# VMs vs. Containers – cont.

By utilizing the same host kernel for different containers, we can run multiple containers, each with it’s own apps and
dependencies, without interference with each other.
The caveat is that you can’t run containers that do not share the host’s kernel. For example, you can’t run Linux containers on
Windows, and vice versa.
For that, we will have to create a VM for each kernel architecture and run our containers on it.
1 physical server host. 2 virtual machines, 1 Windows and 1 Linux. Many containers on each VM.
Hypervisor

### Diagram / Visual Elements:
- Tomcat System
- Container Container

<!-- Page 11 -->

# Containers Components

Dockerfile / Containerfile
The Dockerfile (or Containerfile) is a text document containing all the commands a user would call on the command line to
assemble an image.
- Instruction-Based: It uses keywords like FROM (base OS), RUN (installing packages), and COPY (adding your code).
- Automation: It allows for a repeatable, automated build process.
- Best Practices: Modern DevOps focuses on "Multi-stage builds" to keep the final image size small by separating the build
environment from the runtime environment.
Building Layers
A container image is not a single giant file; it is a stack of read-only layers. Each instruction in your Dockerfile (RUN, COPY, ADD)
creates a new layer.
- Storage Efficiency: If two different images both start with FROM ubuntu:22.04, they share those exact same base layers on the
disk, saving massive amounts of space.
- Caching: During the build process, if you haven't changed the first three lines of your Dockerfile, the builder will reuse the cached
layers, making subsequent builds nearly instantaneous.
- Immutability: Once a layer is created, it never changes. When a container runs, it adds a thin "writable layer" on top, but the
underlying image layers remain untouched.

### Diagram / Visual Elements:
- Lae)
- Cd
- Dockerfile / Containerfile
- assemble an image.

<!-- Page 12 -->

# Containers Components – cont.

Image
A lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime,
system tools, system libraries, and settings.
- Static Artifact: "Template" or the "Class", while the container is the "Instance" or the "Object."
- Portability: Because all dependencies are bundled inside, an image built on nodeA will run exactly the same way on nodeB
- Security: Images can be digitally signed and scanned for vulnerabilities before they are ever deployed to a cluster.
Container Registry
A Registry is a storage and distribution system for named container images.
- Public vs. Private: Docker Hub is the most famous public registry, but enterprises use private registries like Quay.io, Azure
Container Registry (ACR), or JFrog Artifactory to secure their proprietary code.
- The Pull/Push Workflow: Developers push images to the registry after building, and Kubernetes nodes pull those images when
it's time to run a Pod.
- Integration: In a DevOps pipeline, the CI tool (like GitLab CI) automatically pushes the successful build to the registry.
Versions (Tags)
In the container world, versions are managed via Tags. A tag is an alias for a specific image digest (a unique hash of the image
contents).
- The "Latest" Trap: The :latest tag is a pointer to the most recent build. In production, you should never use :latest because it
isn't deterministic; you want to use specific version tags like :v1.2.3.
- Immutable Tags: For ultimate reliability, some teams use the SHA256 Digest (e.g., image@sha256:8b3d...) to ensure that even if
a tag is overwritten in the registry, the cluster continues to run the exact bit-for-bit version intended.

### Diagram / Visual Elements:
- mY)
- Ce
- Versions (Tags)

<!-- Page 13 -->

# Container Image Lifecycle – illus.


### Diagram / Visual Elements:
- g DOCKERFILE
- DEY: FROM node: 10.24
- copy package.json ./
- run npm install
- expose 8080
- cmd ['npm’, “start"]
- SOURCE
- CODE
- PUSH TRIGGER ci/cD
- CODE BUILD
- VERSION BUILD TEST
- CONTROL
- REPOSITORY iE
- =)
- PUSH IMAGE
- snappy_web: 1.0
- snappy_photo: 2.4 | shappy_web: 1.0
- snappy_photo: 2.3 | | snappy_web: 0.9
- snappy_photo: 2.2 | | snappy_web: 0.8
- IMAGE REGISTRY

<!-- Page 14 -->

# What is Kubernetes (K8s)?

Kubernetes is an open source container orchestration engine for automating deployment, scaling, and management
of containerized applications. The open source project is hosted by the Cloud Native Computing Foundation
(CNCF).

### Diagram / Visual Elements:
- Cod
- Casi eS ene eerie
- me
- En — ee — ee
- SE OE
- ' ' '
- Container Host Platform
- Container Host Platform
- plication Jf Applic:
- Traditional Deployment Virtualized Deployment
- plication Applic
- Container Deployment

<!-- Page 15 -->

# Why Kubernetes?

Operational Domain
Legacy solutions + manual containers
Kubernetes
Fault toleration
Manual Remediation
Reconciliation Loop (Self-Healing)
Scalability
Static Provisioning
Autoscaling
Connectivity
Manual Networking
Dynamic Service Discovery
Lifecycle management High-Risk Deployments
Rolling Updates & Automated
Rollbacks
Resource Efficiency
Underutilized resources
Resource optimization
IaC
Imperative
Declarative

### Diagram / Visual Elements:
- la?)
- Cts
- lac

<!-- Page 16 -->

# Kubernetes Architecture

<!-- Page 17 -->

# Cluster Planes Overview

Kubernetes follows a master-worker architecture (now called Control Plane and Worker Nodes). Before diving into workloads, you
must understand where your applications run and what components manage them.
The 2 main planes are:
Control Plane (Master Nodes)
Worker/Data Plane (Worker Nodes)
- The “brain” of the cluster
- 
The ”muscle” of the cluster
- Makes global decisions about the cluster (scheduling,
- 
Run your actual application containers
- 
Receive instructions from the control plane
- Can run on multiple nodes for high availability
- 
Report status back to the control plane
- Does NOT run your application workloads (by default)
- 
Can be scaled horizontally (add more nodes for more
detecting/responding to events)
and should not.
capacity)

### Diagram / Visual Elements:
- la)

<!-- Page 18 -->

# Control Plane Components

The Control Plane consists of four critical components that work together to manage the
cluster:
kube-apiserver
- Entry Point: Serves as the singular entry point for the entire cluster; every internal component and external user (via kubectl)
communicates only through this API.Exposes the Kubernetes REST API
- Authentication & Authorization: Validates the identity of the requester and ensures they have the correct RBAC (Role-Based
Access Control) permissions before processing any request. Executes "Admission Controllers" to modify or reject requests based
on cluster policies (e.g., ensuring every Pod has resource limits defined).
- State Coordination: Acts as the only component allowed to talk to etcd, serving as a synchronized front-end that prevents data
corruption from multiple simultaneous writers.
- Horizontally scalable
etcd
- The single source of truth for the cluster
- State Persistence: Stores the complete configuration, metadata, and current status of every object in the cluster (Pods, Secrets,
ConfigMaps, etc.). Stores the desired state and the actual state of the cluster at any given point of time.
- Distributed Key-Value Store: Designed specifically for distributed systems where low-latency reads and highly reliable writes are
critical for cluster stability.
- Can be spread across multiple instances – Using Leader elections and write once read many config.
If etcd gets corrupt and you have no backup, your cluster configuration is gone.

### Diagram / Visual Elements:
- Lae)
- Cod
- ste iserver

<!-- Page 19 -->

# Control Plane Components – cont.

kube-scheduler
This component constantly monitors etcd for desired/actual state changes, and schedules new pod request to node.
It DOES NOT place the Pods directly onto the worker node; instead, it notifies the kube-apiserver to send a create Pod request to a
worker node via the Kubelet.
The kube-scheduler’s job is to decide on which node the Pod should be created, with constraints such as resources, affinity rules,
taints & toleration rules and manual requests.
kube-controller-manager
- Reconciliation Loop: Continuously runs a "watch" loop to observe the actual state of the cluster and takes corrective action to
move it toward the user-defined desired state.
- Node Controller: Responsible for monitoring node health; it marks nodes as "Unreachable” if need and initiates pod rescheduling
if a node stops sending heartbeats.
- Replication Controller: Ensures that the exact number of Pod replicas specified in a Deployment is running at all times (scaling
up or down as needed).
- Endpoints Controller: Populates the Endpoints object, effectively joining Services and Pods together so traffic can flow to the
correct container IPs.

### Diagram / Visual Elements:
- la?)
- Cd
- kube-scheduler
- up or down as needed).

<!-- Page 20 -->

# Control Plane - Analogy

etcd
ears +
mouth
kube-apis
erver
kube-contr
oller-mana
ger
kube-sch
eduler

### Diagram / Visual Elements:
- Q kube-apis
- or ’ kube-contr
- oller-mana
- er
- kube-sch

<!-- Page 21 -->

# Data Plane Components

The worker nodes consists of three critical components that work together to ensure correct containers start-up and
communication with the Control Plane components:
kubelet
- Pod Lifecycle Management: Primary responsibility is to ensure that the containers are running and healthy on the local node.
- CRI Integration: Communicates with the Container Runtime via the Container Runtime Interface (CRI) to pull images, start
containers, and stop them.
- Health Monitoring: Executes Liveness and Readiness probes to determine if a container is functioning or needs to be restarted.
- Secret/ConfigMap Mounting: Orchestrates the mounting of sensitive data and configuration files from the Kubernetes API into
the local container filesystem.
kube-proxy
- Service Abstraction: Implements the "Service" concept by managing the virtual IP addresses that represent a group of pods.
- IPTables/IPVS Management: Manipulates the node’s host-level networking rules (usually via iptables or ipvs) to route traffic to
the correct container.
Container Runtime Interface (CRI)
- Image Management: Responsible for pulling container images from private or public registries and managing local image
storage.
- Container Isolation: Leverages Linux kernel features like Namespaces and Cgroups to ensure containers are isolated from the
host and each other.

### Diagram / Visual Elements:
- Lae)
- Cod
- kube-proxy

<!-- Page 22 -->

# Kubernetes Architecture – illus.


### Diagram / Terminal Content:
```yaml
mY)
Ce
SB. Kubernetes Control Plane
Scheduler
eted a
(3) Controller
Manager
Vv
Master
API Server }—
Mm
“<J> | kubectl
[7
Node
> Kubelet
low Kube-proxy
```

<!-- Page 23 -->

# Kubernetes Architecture – illus.


### Diagram / Visual Elements:
- mY)
- Ce?
- Kubernetes Control Plane
- Scheduler
- el
- ret
- Vv
- etcd
- 7 xy
- ) Controller
- Manager
- wN ig)
- Master
- API Server }—
- mM
- eszeensennes ees on kubect!
- Node
- Se
- -~_| || B==ee
- [7
- Be ae Kube-proxy
- .\)

<!-- Page 24 -->

# Workloads

<!-- Page 25 -->

# Imperative vs. Declarative Approaches

in Kubernetes (and in general) we have 2 ways of interacting with the Kubernetes cluster; Imperative commands and Declarative
manifests. Both ways achieve the same outcome, but with practically different ways to achieve it:
Imperative Infrastructure
Imperative management relies on executing a series of specific, manual commands to reach a desired configuration, essentially
functioning as a "how-to" recipe. Because the user is responsible for managing the step-by-step, this method is prone to
configuration drift if commands are interrupted or executed in an incorrect order.
Declarative Infrastructure
Declarative infrastructure focuses on the desired end state, defined in configuration manifests like YAML, allowing the system to
reconcile the current environment with that target. Kubernetes thrives on this model: the control plane continuously runs a
reconciliation loop, observing the cluster, calculating the difference between the actual and desired state, and taking corrective
action to align them without manual intervention.

### Diagram / Visual Elements:
- mY)
- Cd

<!-- Page 26 -->

# Declarative vs. Imperative - example


### Diagram / Terminal Content:
```yaml
apiVersion: v1
kind: Pod
metadata:
name: nginx
spec:
te ted Explicit Instructions Describe the Outcome
— name: nginx
image: ng The system is stupid, The system is smart,
ports: you are smart you don’t care
- containerPort:
```

<!-- Page 27 -->

# Declarative Manifests in Kubernetes

Kubernetes uses declarative resources files to define what is the desired state. These are called ‘manifests’.
These are written in the .yaml file format.
Most Kubernetes resources .yaml manifests follow the next pattern:
apiVersion - Which version of the Kubernetes API you're using to create this object
kind - What kind of object you want to create
metadata - Data that helps uniquely identify the object, including a name string, UID, and optional namespace
spec - What state you desire for the object

### Diagram / Terminal Content:
```yaml
fy,
Ce
kind: Pod
name: nginx
containers:
— name: nginx
image: nginx:1.1
ports:
- containerPort:
```

<!-- Page 28 -->

# Kubernetes Pod

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.
A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the
containers.
Pods in a Kubernetes cluster are used in two main ways:
Pods that run a single container:
The "one-container-per-Pod" model is the most common Kubernetes use case.
In this case, you can think of a Pod as a wrapper around a single container.
Kubernetes manages Pods rather than managing the containers directly.
Pods that run multiple containers that need to work together:
A Pod can encapsulate an application composed of multiple
containers that are coupled and need to share resources.
These containers form a single unit.

### Diagram / Terminal Content:
```yaml
fe),
Cod
Persistent Storage
Volume Mount
apiVersion: v1 ry, (J
kind: Pod
metadata
waleess ae,
name: y = J {senses
y y
containers: Node
— name: nginx
ports:
- containerPort: 8@
Kubernetes Network (CND)
```

<!-- Page 29 -->

# Workload Types

ReplicaSet
A ReplicaSet is a low-level controller designed to ensure that a specific number of identical Pod replicas are running at any given
time. It acts as a stability mechanism.
- Pod Guarantee: Maintains the "Desired State" by constantly checking the number of running Pods against the defined count.
- Self-Healing: If a Pod is deleted or crashes, the ReplicaSet immediately triggers the creation of a replacement.
- Label Selectors: Uses selectors to identify which Pods it owns, allowing it to "acquire" existing Pods that match its criteria.
- Scaling: Allows for manual or automated scaling of Pod counts up or down.
- Foundational: Almost never used directly by users; it is primarily managed by the Deployment controller.

### Diagram / Terminal Content:
```yaml
12)
Cod
[admin@master1 examples]$ kubectl get replicasets
ee TCLS NAME DESIRED CURRENT READY AGE
rpibicen 3 frontend 3 3 3 6s
eelecron: [admin@master1 examples]$ kubectl get pods
matchL
tier: fron NAME READY STATUS RESTARTS
template: frontend-g6vgx 1/1 Running @
metadata:
Tia frontend-n9mzg 1/1 Running @
tier: f frontend-sw52k 1/1 Running @
containers
```

<!-- Page 30 -->

# Workload Types

Deployment
The most common resource for managing applications. It provides declarative updates for Pods and ReplicaSets, allowing you to
manage software versions.
- Rollouts and Rollbacks: Automates the transition from version A to version B without downtime using "Rolling Updates."
- Version History: Maintains a revision history, allowing you to revert to a previous stable state with a single command.

### Diagram / Terminal Content:
```yaml
: apps/v1 [admin@master1 amples]$ kubectl create deployment my-deployment --image nginx: latest \
nd: Deployment —-replicas 3
7 Brey deployment. apps/my-deployment created
[admin@master1 les]$ kubectl get deployments
: frontend NAME READY UP-TO-DATE AVAILABLE AGE
: [admin@master1 examples]$ kubectl get pods
: frontend NAME READY STATUS RESTARTS AGE
my—deployment-79b8f6589f-5v5xk 1/1 Running @ 10s
my-deployment-79b8f6589f-j76hh 1/1 Running @ 10s
my-—dep Loyment-79b8f6589f—q8t5n 1/1 Running 0 10s
[admin@master1 1$ kubectl scale deployment my-deployment --replicas 7
ss deployment.apps/my-deployment scaled
: frontend [admin@master1 examples]$ kubectl get pods
NAME READY STATUS RESTARTS AGE
my-dep Loyment-79b8f6589f—5v5xk 1/1 Running 0 31s
my-dep Loyment-79b8f6589f-87b4c 1/1 Running 8s
Si my-dep Loyment-79b8f6589f—gcdgf 1/1 Running 8s
: frontend my-dep loyment-79b8f6589f—hqrvl 1/1 Running 8s
my-dep Loyment-79b8f6589f—j76hh 1/1 Running 31s
my-dep Loyment-79b8f6589f—q8t5n 1/1 Running 31s
my-dep Loyment-79b8f6589f—zf65l 1/1 Running 8s
: my-frontend
: nginx: latest
3°
```

<!-- Page 31 -->

# nginx:1.

0
nginx:2.
0

<!-- Page 32 -->

# Deployment - Example

Deployment’s spec
Pod’s spec

### Diagram / Terminal Content:
```yaml
Deployment’s spec
replicas: 3
selector:
matchLabels:
metadata:
labels:
app: nginx
containers:
- name: nginx
image: nginx: latest
Pod’s spec
```

<!-- Page 33 -->

# Why Use Deployments Instead of Direct Pods?

Direct Pods
Deployments
No self-healing
Auto-replaces failed Pods
No scaling
Easy scaling up/down
No rolling updates
Zero-downtime updates
No rollback
Revision history & rollback
Manual management
Declarative management

<!-- Page 34 -->

# Workload Types – cont.

DaemonSet
Ensures that all (or some) Nodes run a single copy of a specific Pod. As nodes are added to the cluster, Pods are automatically
added to them.
Ideal for tasks that must happen at the hardware/OS level of every machine.
Some typical uses of a DaemonSet are:
- running a logs collection daemon on every node
- running a node monitoring daemon on every node

### Diagram / Terminal Content:
```yaml
12)
Ce
added to them.
* running anode monitoring daemon on every node
[admin@master1 examples]$ kubectl get daemonsets
NAME DESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE SELECTOR AGE
logging 2 2 2 2 2 <none> 78s
[admin@master1 examples]$ kubectl get pods
NAME READY STATUS RESTARTS AGE
logging-pwz57 1/1 Running @ 84s
logging-qhsqz 1/1 Running @ 84s
selector:
matchLabels:
iakeire
template:
metadat
label:
tier:
containers:
- name:
image:
```

<!-- Page 35 -->

# Workload Types – cont.

StatefulSet
A StatefulSet runs a group of Pods, and maintains a persistent identity for each of those Pods.
This is useful for managing applications that need persistent storage or a stable, unique network identity.
Example usage for StatefulSets: Database containers
- Ordered Deployment: Pods are created (and deleted) in a strict order (0, 1, 2...), waiting for the previous one to be "Ready."
- Stable Network ID: Each Pod gets a unique, persistent DNS hostname (e.g., db-0, db-1) that stays the same even after a restart.
- Graceful Termination: Ensures data safety by shutting down Pods in reverse order.

### Diagram / Terminal Content:
```yaml
12)
Cod
ErEMErSske0 [admin@master1 examples]$ kubectl get statefulsets
kind: St NAME READY AGE
metadata: : Imysql-db 0/3 12s
! me “ [admin@master1 examples]$ kubectl get pods
a INAME READY STATUS RESTARTS AGE
ae mysql-db-@ 0/1 ContainerCreating @ 20s
app: mysql
renee: 3 in@masterl examples]$ kubectl get pods
cenpiate ly NAME READY STATUS RESTARTS AGE
Sey mysql-db-@ 0/1 Error 4 (5@s ago) 2m34s
Sanne mysql-db-1 0/1 CrashLoopBackOff 2 (26s ago) 105s
spec: aa mysql-db-2 0/1 CrashLoopBackOff 2 (24s ago) 51s
```

<!-- Page 36 -->

# Workload Types – cont.

Job
Used for short-lived, finite tasks. It creates one or more Pods and ensures that a specified number of them successfully terminate.
- Run to Completion: Unlike other resources, the goal of a Job is to stop running once the task is done.
- Parallelism: Can be configured to run multiple Pods at once to finish a large queue of work faster.
- Success Tracking: Tracks successful completions; if a process returns a non-zero exit code, the Job retries until it succeeds or
hits a limit.
- Batch Processing: Ideal for one-off tasks like database migrations or generating a one-time report.
CronJob
A wrapper around a Job that runs it on a recurring schedule, much like a traditional Linux crontab.
- Time-Based Scheduling: Uses standard Cron syntax (* * * * *) to trigger Jobs.
- Concurrency Policy: Defines what happens if a new Job is triggered while the previous one is still running (Allow, Forbid, or
Replace).
- Job History: Keeps a record of successful and failed Job executions for auditing.

### Diagram / Visual Elements:
- hits a limit.

<!-- Page 37 -->

# Deployment Strategies

Deployments have 2 main strategies of replacing old Pods with new ones: Recreate and Rolling Update.
Recreate
All existing Pods are killed before new ones are created. Can result in downtime as old Pods are deleted and new Pods are still
being created/starting.
RollingUpdate
Gradually scale down the old ReplicaSets and scale up the new one.

### Diagram / Visual Elements:
- 12)
- Ce
- Anvesh Muppeda
- Administrator Administrator
- # Zero-downtime (slower, but always full capacity) Rolling 0 aaa
- Sete a a aa Recreate
- type: RollingUpdate Update
- maxSurge: 1
- maxUnavailable: 0
- Upgrade Upgrade
- Deployment Deployment
- # Fast rollout (brief reduced capacity)
- strategy:
- type: RollingUpdate
- maxSurge: 50%
- maxUnavailable: 50%
- Running Running Running Running Running Running
- # Recreate (downtime, but cl itch,
- tecreate (downtime, ut clean switch) v1.0 v1.0 v1.0 v1.0 v1.0 v1.0
- strategy: \ \
- type: Recreate # Kills all old Pods before creating new ones N V, X 4

<!-- Page 38 -->

# Deployment Rollbacks

Sometimes, you may want to rollback a Deployment.
For example, when the Deployment is not stable, such as crash looping.
By default, all of the Deployment's rollout history is kept in the system so that you can rollback anytime you want (you can change
that by modifying revision history limit).
A Deployment's revision is created when a Deployment's rollout is triggered.
This means that the new revision is created only if the Deployment's Pod template (.spec.template) is changed, for example if you
update the labels or container images of the template.
Other updates, such as scaling the Deployment, do not create a Deployment revision.
This means that when you roll back to an earlier revision, only the Deployment's Pod template part is rolled back.

### Diagram / Terminal Content:
```yaml
fy,
Ce
kubectl rollout undo deployment/nginx
kubectl rollout undo deployment/nginx --to-revision=2
kubectl rollout status deployment/nginx
```

<!-- Page 39 -->

# Namespaces – cont.


### Diagram / Terminal Content:
```yaml
Namespaces - cont.
1; apps/v1
: Deployment
: site-a-deployment
eS tce—a
: [admin@master1 1$ kubectl create namespace site-a
. b namespace/site-a created
s [admin@master1 1$ kubectl create namespace site-b
. 1 namespace/site-b created
i [admin@naster1 ]$ kubectl create -f deployment-different-namespaces.yaml
+ i ti
: 2 :
= . [admin@master1 nles]$ kubectl get pods -n site-a
: site—b-pod NAME READY STATUS RESTARTS AGE
a . site-a—deployment-6dcd8dff67-gxgsp 1/1 Running @ 16s
Site—b
: b
= : site-b-container
: nginx: latest
```

<!-- Page 40 -->

# ConfigMaps & Secret


### Diagram / Visual Elements:
- ConfigMaps & Secret

<!-- Page 41 -->

# ConfigMaps

A ConfigMap is an object used to store non-confidential data in
key-value pairs.
A ConfigMap allows you to decouple environment-specific
configuration from your container images.
ConfigMap does not provide secrecy or encryption.
If the data you want to store are confidential, use a Secret rather
than a ConfigMap.
There are four different ways that you can use a ConfigMap to
configure a container inside a Pod:
- Inside a container command and args
- Environment variables for a container
- Add a file in read-only volume, for the application to read
- Write code to run inside the Pod that uses the Kubernetes API to
read a ConfigMap

### Diagram / Terminal Content:
```yaml
Ce
apiVersion: v1
metadata:
J nr
If the data you want to store are confidential, use a Secret rather name: g ame-—-aqemo
than a ConfigMap.
configure a container inside a Pod: DATABAS Es PORT: lie 3
DATABASE_INFO: |
read a ConfigMap NY M
=sql-db-us-
AUTOSAVE=t rue
```

<!-- Page 42 -->

# Using ConfigMaps inside Pods

Using ConfigMaps as files from a Pod
With this method, each key=value pair in the config map becomes a separate file named ‘key’,
and the content of that file is ‘value’

### Diagram / Terminal Content:
```yaml
mY)
Ces
on: vi
ely ConfigMa : pod-with-configmap
: example
Noam
E : nginx: latest
: Banana i #specifying
: | - : configmap-vol
er r : /additional-files
Chiang Mai : -
Koh Samui 2! configmap-vol
ta: # base64 encoded es E
: d29ybGQK : my-configmap
admin@master $ kubectl get contigmaps
NAME DATA AGE
kube-root-ca.crt 1 2d16h
my-configmap 4 102s
[admin@master1 examples]$ kubectl get pods
NAME READY STATUS RESTARTS AGE
pod-with-configmap-as-volume 1/1 Running @ 44s
[admin@master1 examples]$ kubectl exec pod-with-configmap-as—volume -i -- /bin/sh
cd /additional-files
ls
BEST_PLACES
Fruit
cat BEST_PLACES
Chiang Mai
Koh Samui
```

<!-- Page 43 -->

# Using ConfigMaps Inside Pods – cont.

Using ConfigMaps as environment variables
The envFrom field instructs Kubernetes to create environment variables from the sources nested within it. The
inner configMapRef refers to a ConfigMap by its name and selects all its key-value pairs.

### Diagram / Terminal Content:
```yaml
mY)
Ce
apiVersion: v1
kind: ConfigMap
metadata:
name: my-configmap
data: # plaintext values
MY_NAME: Noam
Fruit: Banana
BEST_PLACES: |
Chiang Mai
Koh Samui
binaryData: # base64 encoded values
Hello: d29ybGQK
apiVersion: v1
kind: Pod
metadata:
spec:
containers:
- name: example
image: nginx: latest
name: my-configmap
[admine examples]$ kubectl get pods
NAME READY STATUS RESTARTS AGE
pod-with-configmap-as-env 1/1 Running @ 5s
[admin@ examples]$ kubectl exec pod-with-configmap-as—env -i —- /bin/sh
KUBERNETES_PORT=tcp: //10.96.0.1:443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=pod-with-conf igmap—as—env
Fruit=Banana
HOME=7 Toot
PKG_RELEASE=1~trixie
DYNPKG_RELEASE=1~trixie
ACME_VERSTON=0.3.1
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
NGINX_VERSION=1. 29.6
PATH= /usr/local/shin:/usr/Local/bin:/usr/sbin: /usr/bin:/sbin:/bin
BEST_PLACES=Chiang Mai
Koh Samui
KUBERNETES_PORT_443_TCP_PORT=443
NJS_VERSION=0.9.6
KUBERNETES_PORT_443_TCP_PROTO=tcp
NJS_RELEASE=1~trixie
MY_NAME=Noam
KUBERNETES_PORT_443 TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
```

<!-- Page 44 -->

# Using ConfigMaps Inside Pods – cont.

Sometimes a Pod won't require access to all the values in a ConfigMap. For this use case, you can use the env.valueFrom syntax
instead, which lets you select individual keys in a ConfigMap. The name of the environment variable can also be different from the
key within the ConfigMap.

### Diagram / Terminal Content:
```yaml
XN)
Cod
' vl admin@masteri iples]$ kubectl get pods
NAME READY STATUS RESTARTS AGE
pod-with-configmap-as—-env-specific 1/1 Running 2@ 5s
[admin@master1 ex s]$ kubectl exec pod-with-configmap-as-env-specific -i — /bin/sh
; cp://10.96.0.1:443
: Noam
: Banana
: | re ‘/ root
7 : PKG_RELEASE=1~trrixie
Cline MEE DYNPKG_RELEASE=1~t rixie
Koh Samui ACME_VERSION=0.3.1
Data: # bas TERM=xterm
: d29ybG0K KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
NGINX_VERSION=1. 29.6
PATH=/usr/local/sbin: /usr/local/bin:/usr/sbin: /usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
NJS_VERSION=0.9.6
KUBERNETES_PORT_443_TCP_PROTO=tcp
NJS_RELEASE=1~trixie
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_HOST=10.96.0.1
: MY_VALUE d «| PWD=/
: example
+ nginx: latest
: Fruit
```

<!-- Page 45 -->

# Secrets

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.
Secrets are similar to ConfigMaps but are specifically intended to hold confidential data.
Kubernetes Secrets are, by default, stored unencrypted in the etcd.
Anyone with API access can retrieve or modify a Secret, and so can anyone with access to etcd.
Additionally, anyone who is authorized to create a Pod in a namespace can use that access to read any Secret in that namespace.t.
To combat this, we can use any combination of the following:
- Enable Encryption at Rest for Secrets.
- Enable or configure rules with least-privilege access to Secrets.
- Restrict Secret access to specific containers.
- Consider using external Secret store providers.

### Diagram / Visual Elements:
- mY)

<!-- Page 46 -->

# Secret Types

Kubernetes provides several built-in types for some common usage scenarios.
These types vary in terms of the validations performed and the constraints Kubernetes imposes on them.
You can define a Secret type by assigning a ‘type: <type>’ in the manifest’s ‘type’ field.

### Diagram / Visual Elements:
- Opaque arbitrary user-defined data
- kubernetes. io/service—account-token ServiceAccount token
- kubernetes. io/dockerconfigjson serialized ~/.docker/config.json file
- kubernetes.io/basic-auth credentials for basic authentication
- kubernetes. io/ssh-auth credentials for SSH authentication
- kubernetes.io/tls data for a TLS client or server
- bootstrap. kubernetes. io/token bootstrap token data

<!-- Page 47 -->

# Secret - Opaque

Opaque
Opaque is the default Secret type if you don't explicitly specify a type in a Secret manifest.
This can be any number of key=values without any constraint.

### Diagram / Terminal Content:
```yaml
Secret - Opaque
[admin@master1 es]$ kubectl apply -f secret—opaque.yaml
: v1 secret/my—opaque-secret configured
[admin@master1 s]$ kubectl get secrets
: Secret NAME TYPE DATA AGE
Imy-opaque-secret Opaque 3 40s
aia [admin@master1 mp les]$ kubectl describe secret my-opaque-secret
my-opaque-secret
2: ~my-Opaque-secret a
Opaque <none>
: d29ybGQK
6 bytes
>: bm9hbQo=
r: yellow
hello: 6 bytes
name: 5 bytes
[admin@master1 1$ kubectl get secret my-opaque-secret -o yaml| head -n 10
apiVersion: v1
data:
color: eWVsbG93
hello: d29ybGQK
name: bm9hbQo=
kind: Secret
metadata:
3°
```

<!-- Page 48 -->

# Secret - dockerconfigjson

dockerconfigjson (in manifest file) / docker-registry (in imperative command)
Stores a serialized JSON that follows the same format rules as the ~/.docker/config.json file.
Secret ‘data’ field must contain a ‘.dockerconfigjson’ key for which the value is the content of a base64 encoded
~/.docker/config.json file.
If you don’t have a config.json file, you can create the secret imperatively.

### Diagram / Terminal Content:
```yaml
pay
[admin@master1 ~]$ kubectl create secret docker-registry my-docker-secret \
: Secret —-docker-username=noam \
--docker-password=123 \
[admin@master1 ~]$ kubectl get secrets my-docker-secret -o yaml
: secret-dockercfg
: kubernetes. io/dockerconfigjson
. - . . -dockerconfigjson: eyJhdxXRocyl6eyJyZwdpc3RyeSSvY3RvcHVzY3MuY 29t0j UwMDAi0ns
8 || ‘ TLoY URveE1qgIT0ifX19
<base64-encoded-file> kind: Secret
metadata:
"noam"
’H [admin@master1 ~]$ echo “eyJhdXRocyI6eyJyZWdpc3RyeS5vY3RvcHVzY3MuY29t0jUWMDA
"auth" :"bm9hbToxMjM="}}} 1c2NzLnNvbSIs nf 1dGq404)dbTLoY AveELGTTOAFX19" | base64 -d ; echo "
```

<!-- Page 49 -->

# Secret – auth credentials

basic-auth secret
This type is for storing credentials needed for basic authentication.
When using this Secret type, the data field of the Secret must contain one of the following two keys:
- username: the user name for authentication
- password: the password or token for authentication

### Diagram / Terminal Content:
```yaml
rsion: v1 [admin@naster1 2s]$ kubectl create secret generic my-auth-secret \
--type kubernetes.io/basic-auth \
rom—literal username i
--from-literal passwort
secret/my-auth-secret created
[admin@master1 1$ kubectl get secrets
NAME TYPE DATA AGE
my-auth-secret kubernetes. i0/basic-auth Zz 9s
my-docker-secret kubernetes.io/dockerconfigjson 1 92m
[admin@master1 ]$ kubectl get secret my-auth-secret -o yaml
apiVersion: v1
password: YWRtaW4=
username: YWRtaW4=
kind: Secret
metadata:
creationTimestamp: '"2026-@3-12T04:56:08Z"
namespace: default
resourceVersion: "373649"
uid: £3859e3f-9dd1-4610—b5b0-729d78484895
type: kubernetes.io/basic—auth
kubernetes. io/basic—auth
: admin
: admin
```

<!-- Page 50 -->

# Secret – ssh

ssh secret
This type is for storing data used in SSH authentication.
When using this Secret type, you will have to specify a ‘ssh-privatekey’ key-value pair in the data (or stringData) field as the SSH
credential to use.

### Diagram / Terminal Content:
```yaml
Secret — ssh
credential to use.
[admin@master1 examples]$ kubectl create secret generic my-ssh-secret \
--type kubernetes.io/ssh—auth \
—-from-literal ssh-privatekey=abcd1234
secret/my-ssh-secret created
[admin@master1 « 2s]$ kubectl get secrets
NAME TYPE DATA AGE
my-auth-secret kubernetes. io/basic-auth 2 7m17s
my-docker-secret kubernetes.io/dockerconfigjson 1 99m
my—ssh-secret kubernetes. io/ssh—-auth a 8s
[admin@master1 ]$ kubectl get secret my-ssh-secret -o yaml
apiVersion: v1
ssh-privatekey: YWJjZDEyMzQ=
kind: Secret
metadata:
creationTimestamp: '"2026-03-12T@5:03:17Z"
name: my-ssh-secret
namespace: default
resourceVersion: "374332"
uid: b@c7ea43-ee9f—44fd-b5e7—-bd2e6ac0c39F
pe:_kubernetes. io/ssh-auth
: kubernetes.io/ssh-auth
abcd1234
3°
```

<!-- Page 51 -->

# Secret - TLS

TLS secret
This type is for storing a certificate and its associated key that are typically used for TLS.
When using this type of Secret, the tls.key and the tls.crt key must be provided in the data (or stringData) field.

### Diagram / Terminal Content:
```yaml
a,
Cd
Secret - TLS
E : vi [admin@master1 ]1$ kubectl create secret tls my-tls-secret \
nd: Secret —-cert noam.crt —key noam.key
A secret/my-tls-secret created
. Byer ieeeecrer [admin@master1 examples]$ kubectl get secrets
. : NAME TYPE DATA AGE
: kubernetes.io/tls my-auth-secret kubernetes. io/basic—auth 2) 18m
my-docker-secret kubernetes.io/dockerconfigjson 1 110m
: "REPLACE_WITH_BASE64_CERT" my-ssh-secret kubernetes. io/ssh—auth 1 11m
"REPLACE_WITH_BASE64_KEY" my-tls-secret kubernetes. io/tls 2 8s
[admin@master1 examples]$ kubectl describe secret my-tls—secret
Name: my-tls-secret
.. Namespace: default
Labels: <none>
eco mue Ap elSe) Annotations: <none>
Type: kubernetes.io/tls
tls.crt: 1107 bytes
tls.key: 1675 bytes
3°
```

<!-- Page 52 -->

# Namespaces

In Kubernetes, namespaces provide a mechanism for isolating groups of resources within a single cluster.
Names of resources need to be unique within a namespace, but not across namespaces.
Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc.) and not for cluster-wide
objects (e.g. StorageClass, Nodes, PersistentVolumes, etc.).
Think of it as; People on Project A should not see/use resources of people on Project B, and vice versa.
Concept:
- Namespaces are intended for use in environments with many users spread across multiple teams, or projects.
- Namespaces provide a scope for names. Names of resources need to be unique within a namespace, but not across
namespaces.
- Namespaces cannot be nested inside one another and each Kubernetes resource can only be in one namespace.
- Namespaces are a way to divide cluster resources between multiple users (via resource quota).
Kubernetes starts with four initial namespaces:
default
Kubernetes includes this namespace so that you can start using your new cluster without first creating a namespace.
kube-node-lease
This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the
control plane can detect node failure.
kube-public
This namespace is readable by all clients (including those not authenticated). This namespace is mostly reserved for cluster usage,
in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this
namespace is only a convention, not a requirement.
kube-system
The namespace for objects created by the Kubernetes system.

### Diagram / Visual Elements:
- Cod
- kube-public
- kube-system

<!-- Page 53 -->

# Namespaces – cont.

To view/modify resources of a specific namespace, use ‘kubectl -n <namespace> get’.
If ~/.kube/config file was not changed, this will always default to ‘default’ namespace
This works for any namespaced resource.

### Diagram / Terminal Content:
```yaml
Namespaces - cont.
[admin@master1 ~]$ kubectl get namespaces
NAME STATUS AGE
default Active 3d23h
kube-flannel Active 3d23h
kube-node-lease Active 3d23h
kube-public Active 3d23h
kube-system Active 3d23h
\@naster1 ~]$ kubectl get pods -n kube-flannel
READY STATUS RESTARTS
kube-flannel-ds-jwgkn 1/1 Running 238 (3d3h
kube-flannel—ds-vhpfm 1/1 Running 238 (3d3h
kube-flannel—ds-x98tx 1/1 Running 238 (3d3h
[admin@master1 ~]$ kubectl get pods -n kube-system
NAME READY STATUS
coredns-66bc5c9577-dr2gh 1/1 Running
coredns-—66bc5c9577-qz8qv 1/1 Running
etcd-master1 1/1 Running
kube-apiserver-master1 1/1 Running
kube-controller-manager-master1 1/1 Running
kube-proxy-7 lpk2 v1 Running
kube-proxy-jhcjb 1/1 Running
kube-proxy-Lmh59 1/1 Running
kube-scheduler-master1 1/1 Running
[admin@master1 ~]$ kubectl get pods -n default
NAME READY STATUS
frontend-84db57ffdf-6w7r7 = 1/1 Running
frontend-84db57ffdf-8vd7b 1/1 Running
frontend-84db57ffdf-g6prr 1/1 Running
frontend-84db57ffdf-rtvb6 = 1/1 Running
frontend-84db57ffdf-vdwj4 1/1 Running
RESTARTS.
AGE
3d23h
3d23h
3d23h
ago)
ago)
ago)
RESTARTS
(2d13h ago)
al
(2d13h ago)
AGE
4m9s
4m9s
4m9s
4m9s
4m9s
AGE
3d23h
3d23h
3d23h
3d23h
3d3h
3d23h
3d23h
3d23h
3d23h
[ @naster1 ~]$ kubectl get configmaps
INAME DATA AGE
kube-root-ca.crt 1 3d23h
my-configmap 4 3@h
no-scoring-scheduler-cm 1 19h
@naster1 ~]$ kubectl get configmaps -n kube-system
DAT,
-apiserver-authentication
kube-apiserver-legacy-service-account-token-t racking
kube-proxy
kube-root-ca.crt
kubeadm-config
kubelet-config
```

<!-- Page 54 -->

# Namespaces – cont.

To see which resources are namespaced, and which are cluster-wide, use the following commands:

### Diagram / Terminal Content:
```yaml
Namespaces - cont.
[admin@naster1 ~1$ kubectl api-resources:
NAME ‘SHORTNANES
bindings
confignaps cn
endpoint: ep
events ev
Linitranges Limits
persistentvoluneclaims pve.
pods
podtenplates
replicationcontroLters
controtlerrevisions
daenonsets
deployments
replicasets
statefulsets
Vocalsubjectaccessreviews
horizontalpodautoscalers
cronjobs
jobs
leases
endpointslices
event
ingresses
networkpolicies
poddisruptionbudgets
rolebindings
jaintenplates
csistoragecapacities
=nanespaced=true
APIVERSION
va
va
va
vl
va
apps/vi
apps/vi
apps/vi
apps/vi
apps/v1
authorization. k8s.io/v1
autoscaling/v2
batch/vi
batch,
coordination. k8s. io/v1
discovery. ks. i0/v1.
events. k8s. io/v1
networking. k8s.io/v1
networking. k8s.io/vi
policy/v1
rbac.authorization.
rbac.authorization.
resource. k8s.i0/v1
-k8s_i0/v1
ge. KBs. i0/v1
COMPUTER SOLUTION
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
true
KIND
Binding
ConfigMap
Endpoints
Event
LimitRange
PersistentVoluneClain
Pod.
dTenplate
ReplicationController
ResourceQuota
Secret
ServiceAccount
Service
ControtterRevision
DaemonSet
Deployment
Replicas
Statefulset
LocalsubjectAccessReview]
HorizontalPodAutoscaler
CronJob
Job
Lease
EndpointSlice
Event
Ingress
NetworkPolicy
dDisruptionBudget
RoleBinding
Role
ResourceClain
ResourceClainTenplate
csIStorageCapacity
W@vasterl ~]$ kubectl api-n
persistentvolunes
nutat ingwebhookconf igurations
validat ingadniissionpolicies
alidat ingadmissionpolicybindings
validat ingwebhookconfigurations
customresourcedef initions
apiservices
selfsubjectreviews
tokenreviews
selfsubjectaccessreviews
igningrequests
flowschemas
prioritylevelconfigurations
ingressclasses
ipaddresses
servicecid)
runtineclasses
clusterrolebindings
csidrivers
csinodes
ih
Ources —fanespaced=false
SHORTNAMES VERSION
vi
adnissionregistration. ks.
adnissionregistration.k8s.
inissionregistrat ion. k8s,
inissionregistrat ion. k8
apiextensions.k8s.io/v1
apiregist ration. k8s. io/v1
authent ication. KBs. io/v1
authent icat ion. k8s, io/v1
authorization. kés.io/v1
authorization. kés.io/v1
authorization. kés.io/v1
rtificates.k8s.io/vi
flowcontrol.apiserver.k8s.io/v1
flowcontrol.apiserver. ks. 10/v1
networking. K8S. 10/V1
king. KBs. io/v1
networking. kBs. io/v1
node.k8s. i0/v1
authorization. k8s.io/v1
rac. authorization. KBs. i0/v1
heduLing. k8s.i0/v1
storage: k8s. i0/v1
pohaciles.nge-nigey hy
KIND
Conponentstatus
Node
PersistentVolume
Mutat inghebhookConfiguration
ValidatingAdmissionPolicy
falidat ingAdmissionPolicyBinding
/aLidat ingWebhookConfiguratio
CustonResourceDefinition
APIService
SelfSubjectReview
TokenReview
Sel fSubjectAccessReview
SelfSubjectRulesReview
CertificateSigningRequest
FlowSchema
PriorityLevetConfiguration
IngressClass
TPAddress
ServiceCIOR
RuntimeClass
ClusterRoleBi
ClusterRole
DeviceClass
ResourceSlice
PriorityClass
csIDriver
CSINode
StorageClass
Voluneattachnent
ie tlaakit abteneian
```

<!-- Page 55 -->

# Namespaces – cont.


### Diagram / Terminal Content:
```yaml
a »)
Namespaces - cont.
Cod
apiVersion: v1
kind: Pod
metadata:
name: [admin@master1 examples]$ kubectl create -f pod-
; o-namespace. vaml
namespace: hell Error from server (NotFound): error when creating
"pod-no-namespace.yamL": namespaces "hello" not
spec: found
containers:
— name: nginx
image: nginx: latest
```

<!-- Page 56 -->

# Networking – intro.


### Diagram / Visual Elements:
- Networking — intro.

<!-- Page 57 -->

# Networking in General

At its core, networking is the infrastructure and set of rules (protocols) that allow independent computers to exchange data.
In a DevOps and Kubernetes world, networking is what transforms a pile of isolated servers into a single, cohesive distributed
system.
For two services to interact, they need a reliable way to find each other.
This is achieved using a combination of IP addresses and Ports.
Example:
When your web browser wants a webpage, it sends a request to the server's IP address, specifically aiming for Port 443 (HTTPS).
The server receives the packet, sees it is destined for Port 443, and hands it directly to the web server process listening.
Core Networking Components
IP (IPv4) Address
- A 32-bit logical address assigned to a device on a network (e.g., 192.168.1.50).
- It acts as the routing destination. It is a logical address, meaning it can change depending on which network the device is
connected to.
- Kubernets clusters usually operate on a private subnet range
Port
- A 16-bit logical construct (ranging from 0 to 65535) that identifies a specific process or network service running on a machine.
- It multiplexes network traffic. Because a single server has only one IP address but runs dozens of services (SSH, Web, Database),
ports ensure incoming data is handed to the correct application.
- Port 22 is standard for SSH, 80 for HTTP, and in Kubernetes, the API server typically listens on Port 6443.

### Diagram / Visual Elements:
- Lae)
- GC ) Networking in General
- Cod
- IP (IPv4) Address

<!-- Page 58 -->

# Networking in General – cont.

Subnet Mask
- A 32-bit number that masks an IP address, separating the address into two parts: the Network ID and the Host ID.
- It tells the computer whether the destination IP is on the local network or a remote network. For example, a 255.255.255.0 mask
(or /24 in CIDR notation) means the first three numbers of the IP dictate the network, and the last number dictates the host.
- Kubernetes clusters usually have a CIDR of /16
Gateway (Default Gateway)
- The IP address of the router on your local subnet that provides access to external networks (like the internet or other internal
VPCs).
- It is the "exit door." If a server uses its subnet mask to determine that a destination IP is not on its local network, it forwards the
packet to the Default Gateway, trusting the router to figure out the rest of the path.

### Diagram / Visual Elements:
- la?)
- Cod
- ( <-Fon 292¢—f ——
- =i. | SOCKETS =
- —P ort 23 —» = co =
- CLIENT PC = : =
- 104.45 prec ae
- Running Telnet Client
- SERVER MACHINE
- 1011.1

<!-- Page 59 -->

# Kubernetes Networking Model

In older container systems, there was no automatic connectivity between containers on different hosts, and so it was often
necessary to explicitly create links between containers, or to map container ports to host ports to make them reachable by
containers on other hosts.
This is not needed in Kubernetes; Kubernetes's model is that Pods can be treated much like VMs or physical hosts from the
perspectives of port allocation, naming, service discovery, load balancing, application configuration, and migration.
The Kubernetes network model is built upon several fundamental requirements:
- Each pod in a cluster gets its own unique cluster-wide IP address
- A pod has its own private network which is shared by all of the containers within the pod.
- Processes running in different containers in the same pod can communicate with each other over localhost.
- All pods can communicate with all other pods, whether they are on the same node or on different nodes.
- Pods can communicate with each other directly, without the use of proxies.
- Node processes (such as system daemons, or kubelet) can communicate with all pods on that node.
Kubernetes lets you use Container Network Interface (CNI) plugins for cluster networking.
Different plugins are available (both open/closed) in the Kubernetes ecosystem.
CNIs usually gives Kubernetes a /16 subnet range, which gives us about 65,536 IPv4 addresses to work with.

### Diagram / Visual Elements:
- la?)
- Cod

<!-- Page 60 -->

# Viewing Pod IP Addresses:


### Diagram / Terminal Content:
```yaml
mY)
Ce
[admin@ profile.d]$ kubectl get pods -o wide
NAME READY STATUS RESTARTS AGE
nginx-56c45fd5ff-9tmvw 1/1 Running 1 (114m ago)
nginx-56c45fd5ff-xgdgh 1/1 Running 1 (114m ago)
IP NODE NOMINATED NODE READINESS GATES
3h3m > 10.244.0.13 | minikube <none> <none>
3h3m) 10.244.@.11 | minikube <none> <none>
[admin@ profile.d]$ kubectl describe pod/nginx-56c45fd5ff-9tmvw
Name: nginx-56c45fd5f f-9tmvw
Namespace: default
Priority: @
Service Account: default
Node: minikube/192.168.49.2
Start Time: Sun, 08 Mar 2026 05:34:54 -0400
Labels: app=nginx
pod-template-hash=56c45 fd5ff
Annotations: <none>
Status: Running
TPs 10.244.0.13
IPS:
IP: 10.244.0.13
Controtted By: Rep ticaset/tiginx=56c45fd5t f
Containers:
3°
```

<!-- Page 61 -->

# Services

<!-- Page 62 -->

# Services

Pods are ephemeral. They're created, destroyed, and rescheduled constantly.
When a Deployment scales up, new Pods appear with new IPs. When a Pod crashes and restarts, it likely gets a different IP.
The problem:
Imagine you have a frontend application that needs to talk to a backend API. The backend runs as 3 Pod replicas. How does the
frontend know which IPs to use? Even if you hardcode the current IPs, they'll change as Pods come and go. You need something
stable.
The solution: Services
Services Provide:
Stable IP Address - A Service gets a ClusterIP that never changes for the Service's lifetime. Clients connect to this single IP
regardless of how many Pods exist behind it.
Stable DNS Name - Every Service gets a DNS entry. Instead of using IPs, applications can use names like
backend-api.default.svc.cluster.local.
Load Balancing - When multiple Pods match a Service's selector, traffic is distributed across all healthy Pods. This happens
automatically - no application changes needed.
Health-Aware Routing - Services only send traffic to Pods that pass their readiness probes. If a Pod becomes unhealthy, it's
automatically removed from the Service's endpoints.
Service Discovery - The combination of stable DNS names and automatic endpoint updates means applications can discover and
connect to other applications without hardcoding anything..

### Diagram / Visual Elements:
- a,
- Cd

<!-- Page 63 -->

# How Services Find Pods:

Services use label selectors to identify their target Pods.
A Service with selector "app: backend" routes to all Pods with that label in their metadata.labels field
As Pods with matching labels come and go, Kubernetes automatically updates the Service's endpoint list
Services span across the entire cluster, meaning Pods in NodeA can reach Pods in NodeB.
BUT, Services are not span across namespaces. Service in namespace A can’t point to Pods in namespace B.
To enable cross-namespace Services, we need to set up a Service in each namespace we want.
Pods CAN STILL talk to Services in different namespaces than they are in.
namespace: site-a
namespace: site-b
Service-A
Service-B
Pod-A
Pod-B

### Diagram / Visual Elements:
- Cd

<!-- Page 64 -->

# Service – ClusterIP

ClusterIP is the default Service type and the foundation for all other types.
It provides a stable internal IP address reachable only within the cluster.
How ClusterIP Works:
When you create a ClusterIP Service, Kubernetes allocates an IP from the Service CIDR range (configured during cluster setup,
default is 10.96.0.0/12).
This IP doesn't belong to any physical or virtual interface - it's a virtual IP that exists only in iptables/IPVS rules.
When a Pod sends traffic to the ClusterIP, kube-proxy's rules intercept it and redirect to one of the backend Pod IPs.
The destination Pod sees the traffic coming from the source Pod's real IP (not the Service IP) because kube-proxy does DNAT
(Destination NAT), not full NAT.
deployment
front-end
deployment
back-end
deployment
redis

### Diagram / Visual Elements:
- G2 Service — ClusterlP
- frersprdend
- Jeployment bit]
- abeenidend
- col:
- tPort: 9376
- leployment
- edis redis

<!-- Page 65 -->

# Service – NodePort

If you set the type field to NodePort, the Kubernetes control plane allocates a port from a range 30000-32767.
Each node proxies that port (the same port number on every Node) into your Service.

### Diagram / Terminal Content:
```yaml
Service — NodePort
Ces
apiVersion: v1
kind: Service
metadata:
name: my-se rvice
spec:
- protocol: TCP
targetPort: 8080
nodePort: 30001
```

<!-- Page 66 -->

# Service – LoadBalancer

On cloud providers which support external load balancers, setting the type field to LoadBalancer provisions a load balancer for your
Service.
This only works with a cloud provider.
If you want to use local, on-premise, bare-metal LoadBalancer, you will have to use 3rd party software.

### Diagram / Visual Elements:
- G2 Service — LoadBalancer
- Router
- @ Amazon EKS Cluster
- kOs cluster
- 1P-ADDR1 IP-ADDR2 IP-ADDR3
- C | Amazon Load MetalLB MetalLB MetalLB
- (O eee! LoadBalancer LoadBalancer LoadBalancer
- Users.
- | I I
- Pod
- App2 Deployment Deployment Deployment

<!-- Page 67 -->

# Service – Headless Services

Sometimes you don't want load balancing different IPs, you want to discover all Pod IPs directly.
For this, you use a Headless Service when you want to bypass Kubernetes' built-in load balancing.
You create a Headless Service by explicitly setting ‘clusterIP: None’ in the YAML specification.
Because it lacks an IP address, Kubernetes does not allocate a load balancer or proxy traffic through kube-proxy for this service.
Instead, when you do a DNS lookup for a Headless Service, the DNS server returns the A records (the individual IP addresses) of
every single Pod connected to that service.
Standard services are for stateless applications bthat don’t require a specific Pod to answer.
For stateful applications that need a specific Pod each time, we use Headless Service

### Diagram / Terminal Content:
```yaml
lant »)
Cod
: yanaster1 ~J$ Kubectl get services -o wide
v1 NAME TYPE CLUSTER-IP EXTERNAL-IP — PORT(S) AGE SELECTOR
E . kubernetes ClusterIP 10.96.0.1 <none> 443/TCP 4d9h —<none>
kind : Service my-clusterip-service ClusterIP 10.98.59.233 <none> 80/TCP 2m45s  app=frontend
my-headless-service  ClusterIP None <none> <none> 30s app=database
metadata: my-nodeport-service  NodePort  10.109.255.61 — <none> 80:30001/TCP  2m19s — app=frontend
apiVersion:
name: my-hea
type: ClusterIP
clusterIP: None
selector:
app: database
```

<!-- Page 68 -->

# Cluster Monitoring

<!-- Page 69 -->

# Monitoring

Kubernetes is divided into 2 main parts: The workloads running our applications, and the Kubernetes system itself.
Each part has it’s own logs/errors/system messages, ... etc.
In Kubernetes, logs can be generally viewed in 3 places, each within his own domain; Application specific logs, Resource specific
logs and Cluster-wide specific logs.
Application
Use the command: kubectl logs [-n <namespace>] <pod-name> [<container-name>]
Command will default to the first container in the spec file, otherwise specify a container.

### Diagram / Terminal Content:
```yaml
Ce
japiVersion: v1
kind: Pod
metadata:
name: hello-world-pod
containers:
- name: busybox-hello-world-app
image: busybox@sha256 : b9598f8c98e24d@ad42c1742c¢32516772c3aa2151011ebaf639089bd18c605b8
“/bin/sh"
while true; do echo Hello World $i; i=$((i+1)); sleep 3; done"
[admin@master1 examples]$ kubectl create -f pod-busybox-echo. yaml
pod/hello-world- pod created
[admin@master1 examples]$ kubectl get pods
NAME READY STATUS RESTARTS AGE
hello-world-pod 1/1 Running @ 4s
[admin@master1 examples]$ kubectl logs hello-world-pod
Hello World 1
Hello World 2
Hello World 3
Hello World 4
Hello World 5
Hello World 6
```

<!-- Page 70 -->

# Monitoring – 2+ containers


### Diagram / Terminal Content:
```yaml
bo) Monitoring — 2+ containers
v7)
ivi
: Pod
4 hello-wo rld-pod
= name: busybox-hello-world-app
: busybox@sha256 : b9598f8c98e24d0ad42c1742¢32516772c3aa2151011ebaf639089bd18c605b8
= "/bin/sh"
; while true; do echo Hello World $i; i=$((i+1)); sleep 3; done”
- + busybox-goodbye-world-app
: busybox@sha256 : b9598f8c98e24d0ad42c1742c32516772c3aa2151011ebaf639089bd18c605b8
i "7bin/sh"
while true; do echo Goodbye World $i; i=$((i+1)); sleep 3; done"
fadmin@masterl e les]$ kubectl create -f pod-busybox-echo. yaml
pod/hello-world-pod created
[admin@master1 les]$ kubectl get pods
NAME READY STATUS RESTARTS AGE
hello-world-pod 2/2 Running @ 7s
[admin@master1 1$ kubectl logs hello-world-pod
Defaulted container “busybox—-hello-world-app" out of: busybox—-hello-world-app, busybox—goodbye-world-app
Hello World
Hello World
Hello World
Hello World
Hello World
Hello World
[admin@master1 exam| ]$ kubectl logs hello-world-pod busybox-goodbye-world-app
Goodbye World
Goodbye World
Goodbye World
Goodbye World
Goodbye World
Goodbye World
Goodbye World
Goodbye World
Goodbye World
Goodbye World
Goodbye World
Goodbye World 12
```

<!-- Page 71 -->

# Monitoring – Resource logs

Resource states
This is under ‘status’ in the resource’s spec file
Usage: kubectl get –o <format> <kind> <resource-name>

### Diagram / Terminal Content:
```yaml
Cod
Resource states {admin@naster1 ~]$ kubectl get deployment frontend -o json | jq .status
[admin@master1 ~]$ kubectl get pod hello-world-pod -o json | jq .status|
a2
h,
x WL, [admin@master1 ~]$ kubect1 scale deployment frontend —replicas 1
5 deployment. apps/frontend scaled
Bas [admin@naster1 ~]$ kubect1 get deployment frontend -o json | jq .status
B tly
},
},
:1,
1,
a '
1,
, BBs
: Oh, 11,
: true, 21
: {h, }
```

<!-- Page 72 -->

# Monitoring – Resource logs

Cluster-wide operations logs/state
Usage: kubectl events [-n <namespace>]

### Diagram / Terminal Content:
```yaml
oR)
Cod
[admin@ ~]$ kubectl events
LAST SEEN TYPE REASON OBJECT MESSAGE
3mS@s Normal Scheduled Pod/frontend-84db57f fd f-w4ShS Successfully assigned default/frontend-84db57ffdf-w45h5 to workerl
3n50s Normal Pulling Pod/frontend-84db57ffdf-wa5h5 Pulling image "nginx: latest"
3n50s Normal SuccessfulCreate © ReplicaSet/frontend-84db57ffdf © Created pod: frontend-84db57ffdf—wa5h5
3n5es Normal ScalingReplicaSet  Deployment/frontend Scaled up replica set frontend-84db57ffdf from @ to 1
3n48s Normal Pulled Pod/frontend-84db57ffdf-w45h5 Successfully pulled image “nginx:latest" in 1.8995 (1.95 including waiting). Image size: 1645622
69 bytes.
3m48s Normal Created Pod/frontend-84db57f fdf-w45h5 Created container: nginx
3m48s Normal _ Started Pod/f rontend-84db57f fdf—w45h5 Started container nginx
[adming ~]$ kubectl events
LAST SEEN TYPE REASON OBJECT MESSAGE
3m5@s Normal Scheduled Pod/frontend-84db57f fdf—w45h5 Successfully assigned default/frontend-84db57ffdf-w45h5 to workerl
3m50s Normal Pulling Pod/frontend-84db57ffdf—w45h5 Pulling image "nginx: latest"
3m5@s Normal SuccessfulCreate ReplicaSet/frontend-84db57ffdf Created pod: frontend-84db57ffdf—w45h5
3m50s Normal ScalingReplicaSet  Deployment/frontend Scaled up replica set frontend-B4db57ffdf from @ to 1
3m48s Normal Pulled Pod/f rontend-84db57f fdf—w45h5 Successfully pulled image "nginx: latest" in 1.8995 (1.9s including waiting). Image size:
164562269 bytes.
3m48s Normal Created Pod/f rontend-84db57f fdf—w45h5 Created container: nginx
3m48s Normal Started Pod/frontend-84db57f fdf—w45h5 Started container nginx
[adming ~]$ kubectl set image deployment frontend nginx=nginx@sha256: 810ad1346ec7fd3d0a246c178f2b82e73a43640c691774405adfd38a751lecces —all
deployment.apps/frontend image updated
[adming ~I$ kubectl events
LAST SEEN TYPE REASON OBJECT MESSAGE
8m55s Normal Scheduled Pod/frontend-84db57f fdf—w45h5 Successfully assigned default/frontend-B40b57ffdf-w45h5 to workerl
8m55s Normal ScalingReplicaSet Deployment/frontend Scaled up replica set frontend-84db57ffdf from @ to 1
8m55s Normal SuccessfulCreate  ReplicaSet/frontend-84db57ffdf Created pod: frontend-84db57ffdf—w45h5
8m55s Normal Pulling Pod/frontend-84db57f fdf—w45h5 Pulling image “nginx: latest"
8535 Normal Pulled Pod/frontend-84db57ffdf—-w45h5 © Successfully pulled image "nginx: latest" in 1.8995 (1.95 including waiting). Image size:
164562269 bytes.
8m53s Normal Created Pod/frontend-84db57f fdf—w45h5 Created container: nginx
8m53s Normal Started Pod/f rontend-84db57f fdf—w45h5 Started container nginx
20s Normal SuccessfulCreate ReplicaSet/frontend-5d54fb546b Created pod: frontend-5d54fb546b-q5r2m
20s Normal Scheduled Pod/frontend-5d54fb546b-q5r2m_ Successfully assigned default/frontend-5d54fb546b-q5r2m to workerl
20s Normal Pulling Pod/frontend-5d54fb546b-q5r2m Pulling image “nginx@sha256:810ad1346ec7 fd3d0a246C178T2b82e73a43640C691774405adTd38a75 le
ces"
20s Normal ScalingReplicaSet  Deployment/frontend Scaled up replica set frontend-5d54fb546b from @ to 1
12s Normal Started Pod/frontend-5d54fb546b-q5r2m © Started container nginx
12s Normal Created Pod/frontend-5d54fb546b-q5r2m Created container: nginx
12s Normal Killing Pod/frontend-84db57ffdf—w45h5 © Stopping container nginx
12s Normal Pulled Pod/frontend-5d54fb546b-q5r2m Successfully pulled image "nginx@sha256:810ad1346ec7 fd3d0a246Cc178f2b82e73a43640C69177440:
adfd38a751ecce8" in 7.515s (7.515s including waiting). Image size: 164417856 bytes.
‘Les Normal SuccessfulDelete ReplicaSet/frontend-84db57ffdf Deleted pod: frontend-84db57ffdf—w45h5
12s Normal  ScalingReplicaSet  Deployment/frontend Scaled down replica set frontend-B4db57ffdf from 1 to 0
```

<!-- Page 73 -->

# ANY QUESTIONS?
