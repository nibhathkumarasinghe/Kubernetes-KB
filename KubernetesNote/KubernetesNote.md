**Kubernetes - Core Concepts**

 

Open Container Initiative(OCI)

Kubernetes Cluster Architecture

 

-ETCD

-Kube-api Server

-Kube Controller manager

-Kubelet

-Kube Proxy

-Container runtime

 

**Kubernetes basics**

 

**-**Pod

-Replication controller

-Replica Set

-Deployment

-Namespace

-Service

-Imperative Commands

 

**Kubernetes- Scheduling**

 

**-**Manual Scheduling: ***spec.nodeName: \<node name\>***

-Labels and selectors

-Annotations: ***metadata.annotations***

-Taints and Toleration: kubectl taint nodes \<node-name\>
key=value:taint-effect

-Node Selectors: ***spec.nodeSelector***

-Resource requirement and Limits:
***spec.conatiners\[\].resources.requests,
spec.conatiners\[\].resources.limits***

-LimitRange

-Resource Quotas

-Daemon Sets

-Static pods

-Priority Classes : ***spec.priorityClassName***

-Multiple Scheduler: ***spec.schedulerName***

\-

-Admission Controllers - '***enable-admission-plugins'***

-Validating & Mutating Admission Controllers

 

 

**Kubernetes- Logging & Monitoring**

 

**-**Monitor cluster component

-Application Logs

 

**Kubernetes- Application Lifecycle Management**

 

**-**Rolling Updates and Rollbacks

-Commands and Arguments

-Environment variables

-ConfigMaps

-Secrets

-Encrypting secret data at rest: argument
***encryption-provider-config***

***-***Multi-container Pods Design Patterns: Co-located, Regular Init,
Sidecar

-InitContainers

-Basics of scaling: horizontal scaling, vertical scaling

 

 

**Kubernetes- Cluster Maintenance**

 

**-**OS Upgrades: drain and uncordon nodes

-Kubernetes Software Versions

-Cluster upgrade process

-Kubeadm upgrade: Upgrading controlplane nodes, Upgrading worker nodes

-Backup and restore methods

-Backup and restore ETCDCTL: data-dir=/var/lib/etcd

 

 

**Kubernetes - Security**

 

**-**Authentication mechanism

-Basics of TLS certificates

-TLS in Kubernetes

-View certificate details

-Certificate workflow & API

-KubeConfig: \$HOME/.kube/config

-API Groups : api/v1 and apis

-Authorization Mechanism and Mode

-Role Based Access Controls - role, rolebindings

-Cluster Roles - clusterrole, clusterrolebindings

-Service accounts

-Image security: spec.imagePullSecrets\[\]

 

 

**Security in Docker**

 

**-**Process isolation

-Users in Security

-Security context in Kubernetes: **spec.securityContext.runAsUser,
spec.containers\[\].securityContext.runAsUser**

-Network policy

-Kubectx

-kubens

-Custom Resource Definitions(CRD)

-Custom Controllers

-Operator Framework: OperatorHub

 

**Kubernetes- Storage**

 

-Docker storage: /var/lib/docker/

-Volumes in Docker

-Container storage interface(CSI)

-Volumes in Kubernetes: spec.volumes\[\].hostPath,
spec.containers\[\].volumeMounts\[\].mountPath.

-Persistent Volumes(PV)

-Persistent Volume Claim(PVC):
volumes\[\].persistentVolumeClaim.claimName

-Storage Class: storageClassName in PVC

 

**Kubernetes-Networking**

 

**-**Linux networking: /proc/sys/net/ipv4/ip_forward

-DNS in Linux: /etc/resolv.conf, /etc/nsswitch.conf, CoreDNS
Installation

-Process namespace

-Network Namespaces in Linux: ip netns add red, virtual interface pair,
virtual switch

-Linux Bridge

-Docker networking: bridge interface docker0, iptables -nvL -t nat

-Container Networking Interface(CNI): CNI plugin

-Cluster networking: netstat -nplt

-Pod networking: /etc/cni/net.d/net-script.conflist,
/opt/cni/bin/net-script.sh\>

-Weaveworks(CNI): bridge - WEAVE, weave peers as daemonsets, IPAM

-Service Networking: service-cluster-ip-range, cluster-cidr

-DNS in Kubernetes

-CoreDNS in Kubernetes

-Ingress : Ingress controller, Ingress resource

-Gateway API: Next generation of Kubernetes Ingress, Load Balancing,
Service Mesh APIs

 

**Kubernetes - Core Concepts**

 

**Open Container Initiative(OCI)**

 

\- imagespec, runtimespec

\- Dockershim

\- CLI : ctr, nerdctl, crictl

 

**Kubernetes Cluster Architecture**

 

\- ETCD: Distributed reliable key-value store for cluster states that
stores information about cluster objects in a key value format

 

\- Kube-api Server : The cluster gateway which gets request,
authenticate, validate and forward request to other processes

\- Kube Controller manager: a process that continuously monitors states
of various components in the system and work towards bringing the
components into desired functioning state.

 

\- kube-scheduler : Scheduler only decides the new pod should go to
which node intelligently, depending on certain criteria such as resource
requirement of pods. Phases of scheduler : Filter nodes, Rank nodes

 

\- Kubelet : Kubelet interface the container runtime and node, register
nodes with cluster. It creates and run pods with containers inside it,
assign resources CPU, RAM from node to container

 

\- Kube Proxy : a process that run on each node in the cluster to make
connectivity across nodes in the cluster. It creates the appropriate
iptable rules on each node to forward traffic to those services to the
backend pod when new service created

 

\- Container runtime : Runtime engine for containers such as Docker,
containerD, rocket, RedHat podman

 

**Kubernetes basics**

 

\- Pod: containers are encapsulated into a pod, Pod creates running
environment or layer on top of the container, has 1-to-1 relationship
with containers, a pod can have multiple containers that share same
network and storage space having same lifecycle of pod, kubectl run
nginx - - image nginx - -port=8080, edit pod, debugging pods

 

\- YAML Configuration File in Kubernetes: apiVersion, kind, metadata,
spec

 

\- Replication controller: It helps to run desired number of single pod
and automatically bringing up a new pod when the existing one fails,
balance the load across multiple pods that span on different nodes in
the cluster, intelligent to identify pod with exact labels matches. The
selector is not mandatory field. If so selector gets for label provided
in template section

 

\- Replica Set: Ensures desired number of replicas are running if pod
crashes and balance the load across multiple pods that span on different
nodes in the cluster, requires Selector definition that helps to
identify what pods are under the Replica Set, manage pods that were not
created as part of the Replica Set creation, kubectl scale replicaset
myapp-replicaset - -replicas=6

 

\- Deployment: defines blueprint for pod in order to replicating pods,
upgrade application on replica pods using rolling updates and rolling
back, scale up and scaling down, Deployment is for stateless
applications, creates new replica set with prefixing name of deployment
on it, kubectl set image deployment nginx nginx=nginx:1.18

 

\- Namespace: The way of grouping kubernetes objects, Namespace has set
of policies that own through which we can define resource limit and who
can do what, create ResourceQuota specifying the resource limit for
resources in a namespace, Pod can access a service inside the same
namespace \<Service Name\>.\<namespace\>.\<type service\>.\<domain\>
db-service.dev.svc.cluster.local , kubectl create namespace dev

 

\- Service: Allow connect application together with other applications
or users, Service doesn't have actively listening process that are
virtual component that only lives Kubernetes memory, Service endpoints
are pods that direct traffic based on the selector specified on the
service and labels on the pods

 

Service-NodePort: To expose application run on worker node to external
users, Mapping port on the node to port on a pod, valid port range
30000- 32767, Port is the mandatory in ports array, use selector section
to link pod with a service based on labels

 

Service-ClusterIP: To deploy multi trier microservice based application
in cluster, ClusterIP service groups multiple pods together in each tier
and provide single interface to access the pods in a group, Default type
of service, use selector section to link pod with a service based on
labels

 

Service-Load balancer: To access multiple pods on multiple nodes single
DNS entry, use native load balancer in cloud platform, kubectl expose
pod redis - -type=NodePort - - port=80 - - name=redis-service -
-dry-run=client -o yaml

 

 

**Imperative vs Declarative:** Imperative approach is that specifying
what to do and how to do more importantly to the system whereas
Declarative approach is that specifying what to do not how to do to the
system.

 

*Imperative*

 

Create Objects: kubectl run nginx --image=nginx, kubectl create
deployment nginx --image=nginx, kubectl expose deployment nginx - -port
80 --name=\<service name\>, kubectl run nginx - -image=nginx - -port
80 - -expose=true, kubectl create -f nginx.yaml

 

Update Objects: kubectl edit deployment nginx, kubectl scale deployment
nginx - -replicas=5, kubectl set image deployment nginx
nginx=nginx:1.18, kubectl replace -f nginx.yaml

 

Delete Objects: kubectl delete -f nginx.yaml, kubectl replace - -force
-f nginx.yaml

 

*Declarative*

 

kubectl apply -f nginx.yaml : apply command is intelligent enough to
create/update/delete an object if it doesn’t already exist. Creates live
object configuration that similar to local of object configuration
within Kubernetes memory, Creates last applied configuration in JSON
format. The last applied configuration is stored in live object
configuration with annotation:
kubectl.kubernetes.io/last-applied-confguration. Kubectl create or
replace command don’t store last-applied-configuration like this.

 

kubectl api-resources ; name of resource, shortname, apiVersion,
namespaced(true/false), kind

kubectl explain pod.spec.containers ; showing you all available fields
for container configuration

kubectl explain pod.spec --recursive ; shows all nested fields
recursively in a single output

 

 

 

**Kubernetes- Scheduling**

 

Manual Scheduling: Every pods has nodeName field which has no value set
by default. Scheduler finds right node and schedule pod on a node by
setting nodeName. You only specify nodeName at creation time. No way to
move running pod to one node to another, deleting existing pod and
recreate a new pod is the option. If there is no scheduler to place a
pod, pod status appears as pending. ***spec.nodeName: \<node name\>***

 

Labels and selectors: properties attached to each object based on the
need, selectors helps you to filter these items. Use to connect
different object together such as pod with ReplicaSet, deployment,
service. kubectl get pods -
-selector=“env=prod,bu=finance,tier=frontend”

 

Annotations: To record other details for informatory purpose. For
example; name, version, build information, contact details, phone
number, email ID. ***metadata.annotations***

 

Taints and Toleration: concept to restrict nodes from accepting certain
pods.

 

Taint: If a taint is assigned to a node and any pod is tolerant with the
taint that has on node can be placed on the node. If pod is not
tolerant, pod will be in pending state. Kubernetes set taints on master
node automatically to prevent any pod schedule on master nodes. kubectl
taint nodes \<node-name\> key=value:taint-effect

 

Taint effect: Taint effect determine what happens to PODs that do not
tolerate the taint. NoSchdule, PreferNoSchdule, NoExecute

 

Tolerations: add tolerations to spec section in pod-definition file as
list ***spec.tolerations***. The values in toleration should or should
not encode with double code. If pod manifest file is modified with a
toleration, the pod will be evicted if new toleration is not tolerant
with the taint that the node has.

 

Node Selectors: To restrict pods from scheduling on appropriate node. It
helps to assign pods that needs large resource demand into large node.
It must have label assign to node prior to create pod with node
selector. Use nodeSelector term in spec section in pod definition file
***spec.nodeSelector***

 

Node Affinity: To ensure that pods are hosted on particular nodes with
advanced capabilities such as OR, NOT. Create match expressions that
label value has any value in the list. Affinity rules needs to be in
spec section of the pod template for deployment and replica set.

 

Node Affinity types: requiredDuringSchdulingIgnoredDuringExecution,
preferredDuringSchedulingIgnoredDuringExecution,
requiredDuringSchedulingRequiredDuringExecution

 

Taint/Toleration and Node affinity: Taint and tolerations doesn’t
guarantee pods that have toleration are placed on a node that match with
the taint if there are more nodes with no taints. There is a chance that
pods with no node affinity rules may be placed on the node that have
labels. So this will solve using combination of Taint/Toleration and
Node affinity rules.

 

Resource requirement and Limits: If nodes having no sufficient CPU,
memory and disks for pods to be scheduled, the scheduler avoids placing
the pods on and pods will be pending state. Resource request and limit
should define for each container list/array. Container cannot consume
CPU beyond the limit as system throttle the CPU. If no limit set, the
container can use excessive memory when available, then pod will be
terminated with OOMKilled (Out Of Memory Killed). Kubernetes uses
cgroups (control groups) which a Linux kernel feature, to limit the
resource usage of processes in container.

 

Resource requests: If container needs more resources than by default
they offer, use spec.conatiners\[\].resources.requests section in pod
definition file to specify the numbers of resources requires.

 

Resource Limits: Container has no limits to consume resources at node.
By default, Kubernetes set 1 vCPU and 512 Mi for containers.

 

Default behaviour: Kubernetes doesn’t by default set resource request or
limit on container. If limit is set, then automatically set the request
same as limits. If request and limit are set, the requested resources,
not used beyond the limit. If only request is set, Defining resource
request on pods prevents from consuming all resources on nodes by any
pod unnecessarily.

 

LimitRange: To set default value for CPU and memory on pod, not at pod
definition file. It is applicable at namespace level. Limit range will
be applied on new pods not to the existing pods.

 

Resource Quotas: To restrict total amount of resources that can be
consumed by application deployed in the Kubernetes cluster. Resource
quotas are created at namespace level. Attributes of a running pods
cannot be edited.

 

Daemon Sets: Daemon set ensures to keep a copy of pods on every node in
the cluster. It adds replica of pods to node when a new node is created
and destroys replica when node is removed from cluster. It is ignored by
kube-scheduler for choosing nodes. Use case is when cluster needs
monitoring solution or log viewer to be deployed on each node in the
cluster such as kube-proxy, network plugin(weave-net, flannel).
Daemonsets uses NodeAffinity rule and default scheduler to schedule pods
on each node. Cannot create daemonset in imperative way, so use create
deployment in imperative way, edit kind on the deployment definition
file as daemonset that gets from dry run output, remove the replicas,
strategy and status fields from the YAML file.

 

Static pods: Kubelet can manage a node independently and create pods
without having instruction from kube-api server. Kubelet stores pod
definition file in the directory /etc/kubernetes/manifests and recreate
pods if any changes on it. If pod crashes, kubelet attempt to restart
the pod. Static pods are ignored by Kube-scheduler. Static pods can be
identified by its name that ends with node name and further kind should
be node name in ownerReference\[\].kind. To find location
kubeconfig.yaml file, ps -aux \| grep kubelet and identify where the
config file of kubelet --config=/var/lib/kubelet/config.yaml. Then check
in the kubelet config file for staticPodPath. We use static pod to
deploy control plane components using kubeadm tool.

 

Priority Classes: To ensure higher priority workloads always get
scheduled without being interrupted by lower priority workloads by
defining priorities for different workloads. The scheduler will try to
terminate a lower priority workload if a higher priority pod cannot be
scheduled. The priority classes are non-namespace objects. Range of
priority numbers: 1,000,000,000\>For Database, Critical App, Job
workloads\>- 2,147,487,648 and 2,000,000,000 \> For Kubernetes
controlplane components\> 1,000,000,000. 1. Create priortiy class:
kubectl create priorityclass high-priority --value=100000
--preemption-policy="PreemptLowerPriority" --global-default=true 2.
Associate the priority class to a pod by spec.priorityClassName . If pod
has no priority class name defined, pod assume to have priority class
value is zero. Set globalDefault: true on a priority class to assign
priority class value to pods that have no priorityClassName set
explicitly. preemptionPolicy to define how schedule higher priority pods
when node resources fill with higher priority and lower priority pods.
Default value of preemptionPolicy is PreemptLowerPriority it would kill
the existing lower priority job and take higher priority job to be
placed. If you do not want it to kill or evict the existing workload and
instead wait for the cluster resources to free up to schedule them in
the scheduling queue, then you must set this preemptionPolicy to never

 

Multiple Scheduler: To schedule specific pods, deployment, ReplicaSet on
nodes after performing additional checks having own scheduling
algorithm. Default scheduler in cluster is named as default-scheduler
and scheduler file is scheduler-config.yaml where we can specify name of
scheduler as property schedulerName. Deploy custom scheduler as a pod
specifying path to kube-scheduler configuration file. Use leader Elect
Option when you have multiple copies of scheduler running. Specify
schedulerName in spec section in pod definition file. You can create
scheduler file locally and pass it as volume mount or mapping ConfigMap
as volume

 

Scheduler Profiles: When pods create, they place to Scheduling Queue and
they are sorted based on priority define on pods using PriorityClass.
The property in the pod is priorityClassName. pod with higher priority
value is sorted to start scheduling queue. In scheduler profiles, we can
enable or disable plugins and specify extension points.

 

Scheduling plugin: Plugins are available for these stages such as
Scheduling queue(PrioritySort),
Filtering(NodeResourceFit,NodeName,NodeUnschedulable),
Scoring(NodeResourceFit, ImageLocality), Binding(DefaultBinder)

 

Extension Points: Stages in scheduling have extension point to which a
plugin can be plugged to. Extension points writes own plugin. Some
plugins span across multiple extension points.

 

Admission Controllers: When you want to validate configuration and
perform additional operations beyond what RBAC can perform. Admission
controllers implement better security and compliance measures by
validating configuration, changes the request itself or perform
additional operations before the pod gets created. Kube-apiserver comes
with number of pre-built admission controllers. kubectl exec -it
kube-apiserver-controlplane -n kube-system -- kube-apiserver -h \| grep
'enable-admission-plugins' , Add the required plugin at the flag
enable-admission-plugins within Kube api server manifest file

 

Validating & Mutating Admission Controllers: Validating admission
controller can validate the configuration, and allow or deny it based on
crtieria. Mutating admission controller can change or mutate the object
itself before pod is created. Generally, mutating admission controllers
are invoked first, followed by validating admission controllers. If we
want our own admission controller with our own mutations and validations
that has our own logic, use MutatingAdmissionWebhook and
ValidatingAdmissionWebhook in sequence after list of built-in admission
controllers. These webhooks to point to an admission webhook server
that's hosted either within the Kubernetes cluster or outside it which
can deploy within the Kubernetes cluster itself as a deployment with
service.

 

 

**Kubernetes- Logging & Monitoring**

 

Monitor cluster component: Node level metric(Number of nodes/pods,
healthy nodes/pods), Performance metrics(CPU, memory, disk and network
utilization on nodes/pods). Kubernetes doesn’t come with full-featured
built in monitoring and analytics solution. Open source solution(metrics
server, Prometheus, Elastic stack) and proprietary solution (Datadog,
Dynatrace). Metrics server retrieves metrics and stores them in memory.
Kubelet has sub component cAdvisor(Container Advisor) which retrieve
performance metrics and expose them to Metrics server. Kubectl top node
\| kubectl top pod ; view CPU and memory consumption on node and pod

 

Application Logs: kubectl logs \<name of pod \>, kubectl logs -f \<name
of pod \> ; show live logs on the pod, kubectl logs -f \<name of pod \>
\<name of container \>

 

 

**Kubernetes- Application Lifecycle Management**

 

Rolling Updates and Rollbacks: A new deployment triggers a rollout with
new deployment revision. This helps to keep track the change made on
application and rollback to previous version. kubectl rollout history
deployment \<deployment name\>. Deployment strategy: Recreate
strategy(It destroys all replicas in existing replicaset at once and
create a new replicaset with required changes with the new deployment
revision. it shows 502 bad gateway error when rollout happens),
RollingUpdate strategy(The default deployment strategy. Take down older
version and bring up newer version on pod one pod at a time. Specify how
many pods in ReplicaSet should be unavailable at a time using
maxUnavailable parameters). Update deployment exactly for updating
application version, updating docker container use, updating labels,
updating the number of replicas. kubectl set image deployment \<name of
deployment \> \<current container name\>=nginx:1.9.1, Rollback: kubectl
rollout undo deployment \<name of deployment \>, kubectl rollout undo
deployment/nginx-deployment --to-revision=2

 

Configure Applications

 

Commands: Unlike virtual machine, containers are meant to run specific
service and application( task or process). A container only lives as
long as the process is alive. If you run a docker container from an
Ubuntu image, it runs an instance and exits immediately as bash command
in Dockerfile is not a process which cannot find the terminal to run
because Docker container doesn't by default attach the terminal to a
container. The option is to append command to docker run command which
overrides the CMD and ENTRYPOINT instruction in Dockerfile docker run
ubuntu \[COMMAND\]. In the Dockerfile: CMD sleep 5 (shell form), CMD
\["sleep","5"\] (JSON array format). ENTRYPOINT is the command run at
startup. CMD is the default parameter pass to the command.

 

Commands and Arguments: With the **args** option of pod definition file,
it overrides **CMD** instruction in Dockerfile. With **command** option
in pod definition file, it overrides **ENTRYPOINT** instruction in the
Dockerfile. command: \["sleep2.0" , "10" \].

 

Environment variables: Use to specify environment variable in container
as example credentials to connect database pod from web/application pod.
env property in pod definition file is an array where every items
consist as key and value pair. ENV value types: Plain key value- list of
dictionaries for key and value, ConfigMap for single key-
valueFrom.configMapKeyRef, Secret for single key- valueFrom.secretKeyRef

 

ConfigMaps: An API object used to store non-confidential data in
key-value pairs. It uses for environment variables, command-line
arguments, or as configuration files in a volume. ConfigMap helps to
centrally manage information required for out from pod definition file
and inject configuration data in to the containers in the pod.
ConfigMaps store data in plaintext format. Configure Config Maps has 2
phases: Create ConfigMaps and Inject ConfigMaps. Create ConfigMaps:
kubectl create configmap \<configmap name\> -
-from-literal=\<key\>=\<value\>, kubectl create configmap \<configmap
name\> - -from-file=\<path-to-file\> ; specify multiple key value pair
with multiple - -from-literal, Inject ConfigMaps: 1. using ConfigMap
definition file 2. using Single ENV 3. using file in a volume

 

Secrets: Secret uses to store sensitive information such as password,
keys in base64 encoded format. The built-in security mechanism is not
enabled by default. Configure secret has 2 phases: Create secret and
Inject secret to pod. Create secret: kubectl create secret generic
\<secret-name\> - -from-literal=\<key\>=\<value\>, kubectl create secret
generic \<secret-name\> - -from-file=\<path-to-file\>. On Linux host,
Encode secret: echo -n ‘password’ \| base64, Decode secret: echo -n
‘encoded password’ \| base64 - -decode, Inject secrets to pod: 1.using
secret definition file with envFrom 2.using Single ENV 3. Secret as
Volume. Configure least-privileges access to Secrets-RBAC. Consider
third party secrets store services such as AWS, Azure, GCP, Vault.
Kubelet stores the secret into a tmpfs so that the secret is not written
to disk storage. kubelet will delete local copy of secret data once the
dependent pod of secret is deleted

 

Encrypting secret data at rest: The secret data is stored in ETCD as
unencrypted format. An argument **--encryption-provider-config** pass to
Kube-apiserver that controls how API data is encrypted in ETCD. ps -aux
\| grep kube-api \| grep “encryption-provider-config” ; to check
encryption at rest is already enabled. In EncryptionConfiguration object
you can specify the resources that need to be encrypted and encryption
algorithm with custom keys and the keys that should use for encryption.
You can specify EncryptionConfiguration in kubeapi server manifest file
that stored in /etc/Kubernetes/manifest using an argument
encryption-provider-config and map the hostpath that have
EncryptionConfiguration definition file as volume and then mount the
volume. Any secret after enabling encryption at rest will be encrypted
but not the secret before enabling. kubectl get secrets - -all-namespace
-o json \| kubectl replace -f - ; update existing secrets with same data
by applying encryption using EncryptionConfiguration object

 

Multi Container Pods: Microservices enables to develop and deploy a set
of independent, small, and reusable code

by decoupling large monolithic application into sub components.
Microservices help to scale up, down, as well as modify each service as
required, as opposed to modifying the entire application. Multi
container pods that share the same life cycle, network space and storage
volumes. Multi container pods avoids volume sharing or services between
the pods to enable communication between them.

 

Multi-container Pods Design Patterns: Co-located Containers - Two
containers run in a pod when two services are dependent on each other.
Both has same pod lifecycle and starts together, Regular Init
Containers - initialization steps to be performed when a pod starts
before the main application itself. Init container does its job and ends
its job starting the main application. Add multiple init containers as
list to be run sequentially, Sidecar Containers - An init container
where the sidecar starts first, does its job, but instead of ending its
run throughout the life cycle of the pod as it has restartPolicy set to
always. Start before the main app starts, end after the main app ends.

If you need to create multi-container pod, first get create pod manifest
file with single container with dry-run option. Then modify the pod
manifest file by adding additional containers.

 

InitContainers: Multi containers that are in pod must be alive in pod's
lifecycle. If any of them fails, the POD restarts. Use initContainer for
initialization steps to be performed before main application starts such
as pulls binary or code from remote repository when the pod is created ,
a process that waits for an external service or external database to be
ready, prepare startup script. The process in the initContainer must run
and complete before the real container starts. If any of InitContainers
fail to complete, Kubernetes restarts the pod repeatedly until the
InitContainers succeeds. Multiple initContainers in the list which run
one at a time in sequential order. Describe command specifies state and
reason of init container. If the process is complete normally, the state
should be Terminated and Reason should be Completed with Exit Code is 0.
kubectl logs \<pod name\> -c \<container name\>

 

Self-Healing Applications: ReplicaSets and Replication Controllers
recreates pod automatically when the application within the POD crashes.
Kubernetes check the health of applications running within PODs and take
necessary actions through Liveness and Readiness Probes.

 

Basics of scaling: Vertical scaling means increase the size of the
server after shutting down servers when load increases and then run out
of existing resources(CPU, memory) on the server. Horizontal scaling
means we can add more servers and share load between them without shut
down servers. Types of scaling - Scaling workloads, Scaling Cluster
Infra. Ways of scaling: manual, automated.

 

Manual approach of horizontal scaling cluster infra: use the kubeadm
join command to add the new nodes to the cluster.

Manual approach of vertical scaling cluster infra: shut down server and
add more resources to the servers and bring it back up.

Manual approach of horizontal scaling workloads: run the kubectl scale
command to scale up or down the number of pods.

Manual approach of vertical scaling workloads: run the kubectl edit
command to change the resource limits and requests associated with pods.

Automated approach of horizontal scaling cluster infra: Kubernetes
Cluster Autoscaler.

Automated approach of horizontal scaling cluster workloads: Horizontal
Pod Autoscaler(HPA).

Automated approach of vertical scaling cluster workloads: Vertical Pod
Autoscaler(VPA)

 

Horizontal Pod Autoscaler(HPA): HPA observe metrics, creates more pods
or removes the extra pods, balances the thresholds. kubectl autoscale
deployment my-app --cpu-percent=50 --min=1 --max=10. HPA relies on
metrics server to make scaling decisions. Custom Metrics Adapter that
can retrieve information from internal workload in cluster. External
adapter refers to external sources such as Datadog or Dynatrace. If the
status of (HPA) target is \<UNKNOWN\>/80, it typically means that the
HPA is unable to retrieve the current metrics for the specified target
where target deployment doesn't have any resource fields defined, it
shows Warning FailedGetResourceMetric. Best for Stateless workloads, web
servers(Nginx, API services), message queues, microservices. HPA
resource definition file: scaleTargetRef, minReplicas, maxReplicas,
metrics

 

In-place Resize of Pod Resources: With Kubernetes version 1.32, if you
change resource requirements of a pod in a deployment, the default
behaviour is to delete the existing pod and then spin up new pods.
Hence, in-place resize of pod resources comes as saviour for stateful
workloads. \$ FEATURE_GATES=InPlacePodVerticalScaling=true. enabling
feature flag, pod definition supports a set of resize policy parameters
which allows to specify a restart policy for each resource. Limitation:
Only CPU and memory resources can be changed, Init containers and
Ephemeral containers cannot be resized, Windows pods cannot be resized

 

Vertical Pod Autoscaling(VPA): Pods will be killed and created new pods
to apply new resource values. VPA observes metrics and then adjust
resources assigned to the pods in a deployment and thus balances
thresholds. VPA does not come built-in with Kubernetes, we must deploy
it with VPA definition file available in the GitHub repo which create
following compoenents as pods. Best for Stateful workloads, CPU/Memory
heavy apps(Databased, AI/ML workloads)HPA resource definition file:
TargetRef, updatePolicy(updateMode), resourcePolicy(containerPolicies)

 

VPA Recommender : continuously monitor resource usage from the
Kubernetes metrics API and then provides recommendations on optimal CPU
and memory values.

VPA Updater: gets the information from the VPA Recommender, compare
collected information with actual pods, detects pods that have
suboptimal resources and evicts them when an update is needed.

VPA Admission Controller: intervenes the pod creation process by
mutating the pod spec to apply CPU and memory values at startup that
recommended by VPA Recommender

 

Update policy modes: Off(Only recommends changes by VPA Recommender but
does not change anything), Initial (Only changes on pod creation, not
later), Recreate(Evicts pods if usage goes beyond range specified),
Auto(Updates existing pods to recommended numbers.)

 

 

 

**Kubernetes- Cluster Maintenance**

 

OS Upgrades: Whenever a node goes down, kube-controller-manager awaits 5
minutes(pod eviction timeout) to evict pods and schedule them to healthy
node. If you are not sure node will come up within 5 minutes in a
maintenance, the safer way to do it is that drain the node that have all
the workloads purposefully to other nodes. Draining a node will
gracefully terminate pods and recreate them on another node. Draining a
node is cordon and marked as unschedulable:true means no pod can be
scheduled until you specifically remove restrictions. Then you can do
maintenance and reboot the node. kubectl drain node-1 ;Node becomes
SchedulingDisabled state. This ignores certain system pods on the node
that cannot be killed. kubectl uncordon node-1 ; here pods that moved to
other nodes will not automatically fall back to node that uncordoned.
kubectl cordon node-2 ; It doesn't terminate existing pods instead new
pods are not scheduled on that node. kubectl drain node-1 -
-ignore-daemonsets ; evicting the pods ignoring daemonsets. Pods that
are not part of ReplicationController, ReplicaSet, Job, Daemonsets or
StatefulSet cannot be terminated by draining a node. To override that,
use option - -force and which pod will be lost forever, kubectl drain
node-1 - -ignore-daemonsets --force

 

PodDistruptionBudget: limits the number of pods of a replicated
application that are down simultaneously from voluntary disruption. It
ensures that the number of replicas running is never brought below the
number needed for a quorum.

 

Kubernetes Software Versions: V1.11.3 depicts major.minor.patch
versions. Minor version releases with features and functionalities every
few months. Patch version releases more often with critical bug fixes.
Before releases new minor version, it comes with alpha
tag(V1.10.0-alpha) which is buggy and features are disabled by default.
After fixing the bug which releases with beta tag(V1.10.0-beta) where
the code is tested and new features are enabled by default. All
Kubernetes cluster components such as kube-apiserver, kube-scheduler,
controller-manger, Kubelet, kubectl, kube-proxy with the executable
bundled for all components except ETCD cluster and CoreDNS come with new
minor version . ETCD cluster and CoreDNS has their own version

 

Cluster upgrade process: None of other components should have higher
version than version of kube-apiserver. If kube-apiserver is v1.10,
controller-manager and kube-scheduler could be v1.10 or v1.9, Kubelet
and kube-proxy could be v1.10 or v1.9 or v1.8. kubectl could be v1.11 or
v1.10 or v1.9. The components can be upgraded up to one higher minor
version at a time as recommended. Upgrading the cluster involves upgrade
the master node(cannot access cluster using kubectl, cannot
create/delete/modify app pods) and upgrade the worker nodes(all nodes at
once/one node at a time/Add new nodes with newer software version and
remove old ones)

 

Kubeadm upgrade: kubeadm upgrade plan provides cluster version, kubeadm
version, latest stable version, current and stable version of cluster
components and kubelet. Kubeadm upgrade apply doesn’t install or upgrade
Kubelet or kubectl. kubeadm tool should be upgraded before upgrade the
cluster.

 

Upgrading control plane nodes: Changing Kubernestes package repository
on controlplane nodes, Determine which version to upgrade to , Upgrade
Kubeadm tool (apt-get install kubeadm=1.31.0-1.1), Upgrade controlplane
components(sudo kubeadm upgrade plan --allow-release-candidate-upgrades,
kubeadm upgrade apply v1.31.0), kubeadm upgrade node(use this command if
cluster has multiple controlplane nodes), Drain the controlplane
node(kubectl drain \<node-to-drain\> - -ignore-daemonsets), Upgrade
Kubelet and kubectl(apt-get install kubelet=1.31.0-1.1), Restart the
kubelet(sudo systemctl daemon-reload, sudo systemctl restart kubelet),
Uncordon the controlplane node(kubectl uncordon \<node-to-drain\>)

 

Upgrading worker nodes: Drain the worker node from controlplane (kubectl
drain node1 - -ignore-daemonsets), Changing the package repository on
worker nodes, Determine which version to upgrade to, Upgrade Kubeadm
tool (apt-get install kubeadm=1.31.0-1.1), Upgrade the local kubelet
configuration on worker node(kubeadm upgrade node), Upgrade Kubelet and
kubectl(apt-get install kubelet=1.31.0-1.1), Restart the kubelet(sudo
systemctl daemon-reload, sudo systemctl restart kubelet), Uncordon the
worker node

 

Backup and restore methods: Backup resource configuration- query
kube-apiserver using kubectl or by accessing kube-apiserver directly and
save all resource configuration for objects as copy(kubectl get all -
-all-namespaces -o yaml \> all-deploy-services.yaml), Backup ETCD -
Instead backing up each resources as before, periodically backing up the
ETCD cluster data is important. ETCD comes with built-in snapshot
solution. Backup ETCD- etcdctl snapshot save snapshot.db, snapshot
status snapshot.db and restore ETCD - etcdutl snapshot restore
snapshot.db - -data-dir /var/lib/etcd-from-backup, Specify new data
directory in volume\[\].hostPath.path in
/etc/kubernetes/manifests/etcd.yaml. With this change, /var/lib/etcd on
the container points to /var/lib/etcd-from-backup on the controlplane.
If the etcd pod is not getting Ready 1/1 or pending state, then restart
it by kubectl delete pod -n kube-system etcd-controlplane and wait 1
minute

 

Working with ETCDCTL: set the ETCDCTL_API to 3 for tasks as backup and
restore. etcdctl snapshot save -h , etcdctl snapshot restore -h

 

Backup and restore multiple kubernetes clusters: Explore multiple
clusters and its conetxts (kubectl config, kubectl config view, kubectl
config use-context cluster1, kubectl config get-clusters, kubectl config
get-contexts)

 

Determine how ETCD configured: Stacked ETCD(Running as a pod that
managed by kubeadm, If it has localhost IP address on etcd server in
Kube-apiserver with describe command), External ETCD(Check the static
pod manifest files in the path, Inspect ETCD process to find IP address
of ETCD server, Inspect IP address on etcd server in Kube-apiserver with
describe command)

 

More about ETCD: Connect external ETCD and inspect process(ps aux \|
grep -i etcd), Check members in the external ETCD cluster(Use etcdctl
with options to check member list), Restore a backup to external
ETCD(restore backup to new directory on etcd server itself, update new
data directory in etcd service, grant permissions on the new directory,
restart the etcd service)

 

 

**Kubernetes - Security**

 

Security primitives: Secure Hosts with controls, Authentication(Admins,
developers and service accounts for admin purpose, end users for use),
Accounts(Kubernetes relies on an external source such as files with user
details, certificates, or third party identity service(LDAP), Kubernetes
can create and manage service account)

 

Authentication mechanism: Static password
files(--basic-auth-file=user-details.csv as option in kube-apiserver
service or update in spec.containers.command section in static pod
definition file, curl -v -k <https://master-node-ip:6443/api/v1/pods> -u
“user1:password 123”), Static token files(-
-token-auth-file=user-details.csv, this file includes token, username,
user ID and group, curl -v -k
<https://master-node-ip:6443/api/v1/pods> - -header “Authorization:
Bearer Khsoheynekbst”), Certificates, External identity services(LDAP,
Kerberos, NTLM authentication), Service accounts

 

Basics of TLS certificates : Certificate makes guarantee trust between
two parties when data transfers between parties is encrypted

 

Symmetric encryption uses single key to encrypt and decrypt the data
where the key in plaintext exchange between sender and receiver may be
exposed to hacker.

Asymmetric encryption uses pair of keys such private key and public
key(public lock) where user/browser encrypts data with public key and
transfer data to server and server decrypts the data transferred using
the private key

With asymmetric encryption, user successfully transfer the symmetric key
to the server. With symmetric encryption, thus we secure all future
communication between them.

Thus hackers impersonate with a website looks exactly like original
website to steal the symmetric key from user by generating private and
public key pairs and configures them on his web server

To avoid hackers intervention in this way, the user's browser should
verify the public key received is legitimate key before encrypting the
symmetric key so that certificate created by server is signed by CA
using their private key. CA's public key(root CA certificate) are built
in browsers that uses for certificate validation mechanism.

The CA signed certificate includes serial number, signature algorithm,
issuer, validity, subject name, subject alternative name and public key
info

If a hacker tries to get sign a certificate with a request in similar
way from CA, CA rejects signing the certificate by verifying the
identity of originals. The CA uses different techniques to make sure the
actual owner of the domain.

 

1\. The server uses pair of keys to secure HTTPS traffic. For this,
generate certificate signing request(CSR) using key generated and the
domain name of your website and send it to CA for signing

2\. CA has private and public key pair which is called trusted root CA
certificate. CA uses its CA private key to sign the CSR which has server
certificate.

3\. All users/browsers have a copy of CAs public keys built in with

4\. The certificate signed by CA is sent back to the server. Now the
organization has a certificate signed by CA that the all browsers trust.

5\. The server configures web application with signed certificate

6\. When user access web application, the server first sends the
certificate with its public key

7\. The users/browsers reads the certificate and uses CA's public key
that comes built in the browsers to validate and retrieve the server’s
public key

8\. Then user/browser generates a symmetric key for all communication
going forward and the symmetric key is encrypted using server’s public
key and send back to the server

9\. The server uses private key to decrypt the message and retrieve the
symmetric key

10\. The symmetric key is used to communicate going forward securely.

 

The certificates with public key - \*.crt or \*.pem extension, The
private key - \*.key or XXX- key.pem

 

TLS in Kubernetes: Server certificate for servers exposes HTTPS service
to communicate with its clients. Client certificate for clients who
access through kubectl or REST API. Kubernetes cluster needs one CA for
all components except ETCD server and separate CA for ETCD server.

 

Server certificate for servers: kube-apiserver(apiserver.crt and
apiserver.key), ETCD server(etcdserver.crt and etcdserver.key), Kubelet
server(kubelet.crt and kubelet.key)

 

Client certificate for clients: Admin-kubectl(admin.crt and admin.key),
Kube-scheduler(scheduler.crt and scheduler.key), Kube-controller-manager
(controller-manger.crt and controller-manager.key),
Kube-proxy(kube-proxy.crt and kube-proxy.key), Kube-apiserver as client
talks to ETCD server(apiserver-etcd-client.crt and
apiserver-etcd-client.key), Kube-apiserver as client talks to Kubelet
server(apiserver-kubelet-client.crt and apiserver-kubelet-client.key),
Kubelet as client talks to Kube-apiserver(kubelet-client.crt and
kubelet-client.key)

 

TLS in Kubernetes - Certificate creation: Tools to create certificates
such as EasyRSA, OpenSSL, CFSSL.

 

Generating CA certificate:

generate private key- openssl genrsa -out ca.key 2048

generate CSR specifying private key and CN -openssl req -new -key ca.key
-subj “/CN=KUBERNETES-CA” -out ca.csr

sign the certificate specifying private key - openssl x509 -req -in
ca.csr -signkey ca.key -out ca.crt

 

Generating client certificate:

generate private key for admin user - openssl genrsa -out admin.key 2048

generate CSR specifying private key- openssl req -new -key admin.key
-subj “/CN=Kube-admin/O=system:masters ” -out admin.csr

sign the certificate by specifying CA key pair - openssl x509 -req -in
admin.csr -CA ca.crt -CAKey ca.key -out admin.crt

 

Ways to specify certificate and key to make API call: curl command with
options --key --cert --cacert, Moving parameters into configuration
file- kubeconfig

 

Generating server certificate: ETCD servers deploy as cluster with
multiple servers having peer certificate. Kube-api server: generate
private key, generate CSR specifying private key, CN, OpenSSL config
file that have all subject alternate names, sign the certificate by
specifying CA key pair. Kubelet: certificate and keys are named with
name of each worker nodes, Kubelet client certificate that talks to
kube-apiserver generate with system:node group having admin privilege,
server and client certificate must specify in kubelet-config.yaml file

 

View certificate details: Based on Kubernestes setup from the scratch -
certificates create manually, Based on Kubernetes setup by kubeadm tool-
certificates create automatically, details in certificates : Type(
client or server), Certificate path, CN Names, ALT Names, Organization,
Issuer, Expiration. To decode the certificate intext form: openssl x509
-in /etc/kubernetes/pki/apiserver.crt -text -noout. Inspect service logs
when cluster setup from the scratch: journalctl -u etcd.service -l.
Inspect the logs etcd pod : kubectl logs etcd-master. docker ps -a \|
grep kube-apiserver, docker logs \<container ID\>. All the server and
client certificates for kube-apiserver except ETCD are placed under
/etc/kubernetes/pki/ , ETCD has its own CA and the CA root certificate
is placed in /etc/kubernetes/pki/etcd/

 

Certificate workflow & API: If new user needs pair of certificate(client
certificate and root CA certificate) and key, he needs to generate CSR
and get sign it from CA server through admin. Kubernetes has a built in
certificate API that automates signing certificate and rotating when it
expires. User sends CSR directly to Kubernetes through an API call. When
administrator receives a CSR, instead administrator log in to master
node and signing certificate by himself, he creates Kubernetes API
object which is called CertificateSigningRequest. cat jane.csr \| base64
-w 0 ; encode the CSR to base64 with single line, spec section has
groups, signerName, usages and request. kubectl certificate approve
\<name of CSR\>, kubectl get csr \<name of CSR\> -o yaml ; view
CertificateSigningRequest status output after CSR approved, echo
"\<encoded certificate\>" \| base64 --decode ; decode the signed
certificate into plaintext format. All certificates related operations
are carried out by the controller manager which has controllers as
CSR-APPROVING and CSR-SIGNING. In controller manager service
configuration file has two options where path of CA key file and CA root
certificate file are specified in order to sign CSR, it needs CA
server's root certificate and private key. --cluster-signing-cert-file,
--cluster-signing-key-file

 

KubeConfig: We need to query cluster through Kubernetes REST API(curl)
or kubectl by specifying CA certificate, client certificate, client key
and server. To avoid typing these commands, move these information
KubeConfig file in the path \$HOME/.kube/config. KubeConfig file
contains clusters(name, server, certificate-authority), contexts(name,
namespace, cluster, user), users(name, client-key, client-certificate).
kubectl config -h, kubectl config view, kubectl config use-context
prod-user@production(change the current context in default kubeconfig
file), kubectl config use-context prod-user@production -
-kubeconfig=my-custom-config(change the current context in custom
kubeconfig file). Move the custom kubeconfig file to path of default
kubeconfig file to set custom file as default file. Use full path to
specify for certificate and key in Kubeconfig file :
certificate-authority-data: \<base64 encoded data\>

 

API Groups: All resources of Kubernetes API are grouped into multiple
API groups based on their purpose(/version,/metrics and
/healthz,/logs,/api,/apis), /api/v1- core group is where all core
functionality exists(namespaces, pods, replication controllers, nodes,
services, configmaps, secret, PV, PVC, endpoints, events), /apis - named
group are more organized. All newer features are made available in this
group(apps, extensions, networking, storage, authentication,
certificates, authorization), These API groups have resource group.
/apps/v1 - /deployments, /replicasets, /statefulsets. Each resource
group has set of actions(list, get, create, delete, update, watch). curl
<http://localhost:6443> -k. curl curl <http://localhost:6443> -k - -key
admin.key - -cert admin.crt - -cacert ca.crt - User will not be allowed
to access without certificates. Kubectl proxy lunches a proxy service
locally on port 8001 that will use the credentials and certificates in
kubeconfig file to forward the request to kube-APIserver. (kubectl
proxy, curl <http://localhost:8001> -k)

 

Authorization: This defines the level of access and operation can be
performed by administrators, developers, testers, applications

 

Authorization Mechanism: Node authorization(Kubelet access
Kube-APIserver to read information about objects and reports information
about nodes. node authorizer handles these request and identify node
prefixed with system:node:\<node name\> to grant access), ABAC
authorization(create a policy definition file for each user or group
with set of permissions in JSON format and passing it to API server.
This needs to restart the kube-apiserver after modifying policy
definition file), RBAC authorization(Define roles with associating set
of permissions instead directly associated to user or group. If we
change on role, it reflects to all users associated to the role),
Webhook(to manage authorization externally. Kubernetes makes API call to
the Open Policy Agent which decides whether user should be or not
permitted based on access requirement)

 

Authorization Mode: AlwaysAllow(allow all requests), AlwaysDeny(deny all
requests), Node, ABAC, RBAC, Webhook. These mode are set using
--authorization-mode option on the kube-api server. -
-authorization-mode=Node,RBAC,Webhook(multiple modes evaluate based on
the order of chain. Every time a mode denies the request, it goes to the
next one in the chain and as soon as a module approves the request, no
more checks are done and the user is granted permission)

 

Role Based Access Controls: 1. Create a role with set of permissions and
resources(kubectl create role \<role name\> - -verb=list, create,
delete - -resource=pods --resource-name=\<pod name in namesapce\>), add
multiple rules 2. Create role binding to associate role to user(kubectl
create rolebinding \<name of RoleBinding\> - -role=developer -
-user=dev-user). subjects- add multiple user as list/array, roleRef- add
the role, Role and RoleBinding fall under the scope of namespaces(add
namespace in metadata section). kubectl auth can-i create deployments.
kubectl auth can-i create pods - -as dev-user - -namespace test

 

Cluster Roles: All resources are categorized either namespaced or
cluster-scoped. Namespaced resources(kubectl api-resources -
-namespaced=true) ,Cluster-scoped resources(kubectl api-resources -
-namespaced=false) 1. Create a clusterrole with set of permissions and
resources(kubectl create clusterrole \<role name\> --verb=list, create,
delete --resource=nodes), add multiple rules 2. Create cluster role
binding to associate role to user(kubectl create clusterrolebinding
\<name of RoleBinding\> --clusterrole=\<cluster role\> --user=dev-user).
You can create ClusterRole for namespaced resources where users will
have access to the resources across all namespaces in the cluster. Kind
of the subject can be User, Group or ServiceAccount

 

Service accounts: Service account is used by an application to interact
with a Kubernetes cluster.

 

Before Kubernetes 1.22: Creates service account that is no time bound,
then it automatically creates a token for service account, store the
token in secret object and link secret object with service account.
kubectl create serviceaccount \<name of service account\>. Whenever a
pod is created, the default service account for namespace is used and
its token is mounted as secret as volume. kubectl exec -it \<pod name\>
cat /var/run/secrets/kubernetes.io/serviceaccount/token(to view the
exact token). spec.serviceAccountName: \<service account name\> in spec
section to specify custom service account. automountServiceAccountToken:
false(not to mount service account automatically from namespace default
service account). kubectl set serviceaccount deploy/web-dashboard
dashboard-sa(Set service account created onto existing deployment). To
edit the service account in the existing pod, delete and recreate the
pod

 

Since Kubernetes 1.22: TokenRequestAPI is introduced for provisioning
Kubernetes service account token that is audience bound, time bound,
object bound. This is more secure and scalable via an API. The secret
token is mounted as projected volume onto the pod

 

Since Kubernetes 1.24: It doesn’t automatically create token or secret
when you create a service account. Create a token manually followed by
the name of service account. kubectl create serviceaccount \<name of
service account\>, kubectl create token \<name of service account\>(You
can specify time limit if not it is one hour by default). jq -R
'split(".") \| select(length \> 0) \| .\[0\],.\[1\] \| @base64d \|
fromjson' \<\<\< bfhkfnekfkjebk or jwt.io site (decode the token). You
should only create service account token secrets if you can't use the
TokenRequestAPI to obtain a token

 

 

Image security: Image format image:
\<Registry\>/\<User,Account\>/\<Image Repository\> (image:
docker.io/library/nginx), Private repository must be accessed using a
set of credentials(docker login private-registry.io). Docker runtime in
worker node needs credentials to pull from private repository. So create
secret object with the credentials of private repository and pass it
with spec.imagePullSecrets\[\] in spec section in pod definition file to
pull the image.(kubectl create secret docker-registry private-reg-cred
--docker-username=dock_user --docker-password=dock_password
--docker-email=dock_user@myprivateregistry.com
--docker-server=myprivateregistry.com:5000)

 

**Security in Docker**

 

Process isolation: Containers and the host share the same Kernal. All
the processes run by the containers are in fact run on the host itself
but they are in their own namespace. Processes in the child namespaces
are visible as just another process in the host but with different
process ID. That's how Docker isolates containers within a system.

 

Users in Security: By default, docker run processes within containers as
the root user. Processes are run as root user both within the container
and outside the container on the host. docker run - -user=1000 ubuntu
sleep 3600(specify user ID to run processes in the container). Define
the user Dockerfile instruction in Docker image itself at building of
docker image using Dockerfile. To mitigate the risk of root user running
processes in container and host same way, by default docker limits the
capabilities of root user within the container against root user within
the host. Thus root user within container has no privilages to disrupt
the host or other containers on the same host. docker run --cap-add
MAC_ADMIN ubuntu, docker run --cap-drop KILL ubuntu (add or remove
privileges to an user more than default privileges) docker run
--privileged ubuntu (run a container with all privileges)

Security context in Kubernetes: Configure security settings that user ID
to run processes, Linux capabilities that can be added or removed from
container. You can configure security settings at a container level or
pod level. Capabilities are only supported at the container level not at
the pod level. kubectl exec \<pod name\> -- whoami (To check the user
who is running the pod). If no user defined in securityContext in pod
level, by default process will be run as root user

 

Network policy: Kubernetes is configured by default with an "All Allow"
rule that allows traffic from any pod to any. Network policy is used to
restrict the traffic from one pod or to another pod. Network policy must
be linked to one or more pods with defined ingress or egress rule.
Network policy spec section containes podSelector(link pod with Network
policy like replicaset, service), policyTypes(Ingress,Egress),
ingress(from)/egress(to). Once you add specific rule to network policy,
all other unmatched traffic of the rule is blocked. Only traffic type
you specified in policyTypes is isolated and all other traffic is
allowed. Only need to concern about the direction of the request
originates from. Hence, we don't need seperate rule for reply traffic to
be allowed. To restrict from or to pod in specific namespace, use
namespaceSelector property with podSelector. If resource for from or to
is not a pod, use ipBlock.cidr to set IP address. Add multiple elements
into a rule as list/array.

 

Kubectx- Command line utility to switch between contexts. Kubectx
installation. kubectx(To list all contexts). kubectx \<context_name\>(To
switch to a new context). kubectx - (switch back to previous context).
kubectx -c (see current context)

 

Kubens- Command line utility to switch between namespaces. Kubens
installation. kubens \<new_namespace\>(To switch to a new namespace),
kubens - (switch back to previous namespace)

 

Custom Resource Definitions(CRD): Resources in the cluster have
controllers for watching status of objects and making the necessary
changes. Kubernetes can have custom resources and custom controller. To
create custom resource, first configure custom resource in the
Kubernetes API so that we need custom resource definition(CRD).

 

CRD contains scope(namespaced or cluster-scoped), group(API group),
names: (kind, singular, plural, shortnames), versions(Multiple version
are served), schema(only one storage version are marked from many).

 

CRD allows to create resource and store data about resource in ETCD.
Custom controller allows to perform action with the resource created
based on the resource specifications

 

Custom Controllers: Custom Controller is any process or code that runs
in a loop which monitor the status of the resource objects in ETCD and
perform actions such as making API call. To build a custom
controller(Install Go programming language, Clone GitHub Repo named
sample-controller into local directory, Customize the controller.go with
your custom logic, Build the code, Run by specifying the kubeconfig
file). You can package the custom controller in a docker image and
choose to run it as a pod or deployment.

 

Operator Framework: Operator framework is to package creating the CRD
and the resources using the CRD, then deploying the controller as a pod
or a deployment. Most popular operator is the ETCD operator(ETCDCluster
CRD, custom controller that watches ETCDCluster). All operators are
available at the OperatorHub. Deploy application with operator
framework(Install Operator Lifecycle Manager(OLM), Install the operator,
Watch your operator come up)

 

 

**Kubernetes- Storage**

 

Docker storage: Docker by default stores all its data in
directory(/var/lib/docker/). Each layer creates for line of instruction
in the Dockerfile is built with changes from previous layer and they are
cached. This method is faster and save disk usage. Image layers are read
only. Files in image layers can be modified in container layer which is
writable. Once container delete, all the data stored in container layer
are removed. To resolve this, add a persistent volume to the container.

 

Volumes in Docker: Volume mounting is that mount the volume from volume
directory to data path in container (docker run -v
data_volume:var/lib/mysql mysql). Bind mounting is that mount a full
directory path of any location on docker host to path data in
container(docker run -v /data/mysql:var/lib/mysql mysql) docker run \\
--mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql.
Storage drivers are to maintain the layered architecture, creating a
writeable layer, moving files across layers, to manage storage on images
and containers. They have different performance and
stability(AUFS,ZFS,BTRFS,Device Mapper,Overlay). Volume driver plugins
to handle volume we create. The default volume driver plugin is Local
which create volumes in /var/lib/docker/volumes. Third party volume
driver plugin(Convoy,Azure File
Storage,GlusterFS,Flocker,Portworx,Rexray)

 

Container storage interface(CSI): CRI standardized how new container
runtime communicates with Kubernetes which is independent on Kubernetes
source code. New container runtime must follow the CRI standards.
Kubernetes similarly communicates with different networking solutions
such as weaveworks, flannel, cilium using Container Network
Interface(CNI) plugin. Kubernetes similarly communicates with CSI
drivers such as Portworx, Amazon EBS, Azure Disk, DellEMC Isilon,
GlusterFS using Container Storage Interface(CSI) standards. CSI allows
any container orchestration tool Kubernetes, CloudFoundry, Mesos. CSI
defines set of RPC(Remote Procedure Call) that will be called by
container orchestrators to implement storages by storage
drivers(CreateVolume,DeleteVolume,ControllerPublishVolume)

 

Volumes in Kubernetes: Attach volumes to containers to persist data in
volume when it is created. 1.Create a volume on host
spec.volumes\[\].hostPath 2.Mount the volume inside the container
spec.containers\[\].volumeMounts\[\].mountPath. Data in container
replicates to hostPath. External replicated cluster storage is a
solution for multi node cluster(AWS EBS, Azure Disk or File, Google
Persistent Disk)

 

Persistent volumes: It is not scalable to create volumes for each pods
every time for large environment that have multiple pods. Persistent
volumes is a cluster wide pool of storage volumes that manage centrally
by administrator. Users can select a storage from the pool by using
Persistent Volume Claim(PVC). PV definition file includes
specification(accessModes, capacity, persistentVolumeReclaimPolicy,
hostPath/awsElasticBlockStore/gcePersistentDisk). accessModes
(ReadOnlyMany, ReadWriteOnce, ReadWriteMany, ReadWriteOncePod ). Once PV
is created, its status is Available until PV binds with PVC. Then status
of PV is Bound. kubectl get persistentvolume \| pv

 

Persistent Volume Claim(PVC): Persistent Volume Claim is to make the
persistent volume to be available to node. Kubernetes binds the PV to
PVC based on the requests and properties set on the PVC(Sufficient
Capacity, Access Modes, Volume Modes, StorageClass, Selector). Use
labels and selectors(selector.matchLabels) to bind PVC strictly to
particular PV regardless multiple matches. A PV and PVC has one-to-one
relationship. If no PV available that matches with PVC, the PVC remains
in pending state. Once find the best match in PV, status changes as
Bound. specification(accessModes\[\], resources.requests.storage
,storageClassName). Use PVC in
pods(volumes\[\].persistentVolumeClaim.claimName)

 

Delete PVC: persistentVolumeReclaimPolicy\[Retain(PV will remain until
administrator delete the volume manually and it is not available to
reuse), Delete(PV will be deleted automatically when the PVC
deletes),Recycle(The data in the data volume will be scrubbed before
making it available for other PVC)\]. If a PVC is associated with a pod,
deleting PVC will be stuck in terminating state. The PVC will be deleted
once the associated pod is deleted. Then the PV will be released

 

Storage Class: Manually provisioning a disk in a provisioner such as
GCP, AWS and then attach it on PV definition file is static provisioning
volumes. Automatically provision storage through provisioner in storage
class object and attach that storage to pod using PVC is dynamic
provisioning volumes. PV and any associated storage is going to be
created automatically when pod that links PVC specified starts to run.
Define storage class name in PVC definition file with storageClassName.
Then it provision a new disk with the required size by provisioner and
create PV and then bind it to PVC. Pass additional parameters such as
type of disks to provision, replication type in storage class object. If
the storage class use of VolumeBindingMode set to WaitForFirstConsumer,
This will delay the provisioning and binding of a PersistentVolume until
a Pod that uses the PersistentVolumeClaim is created.
StorageClass(provisioner,parameters\[type,replication-type\],VolumeBindingMode\[WaitForFirstConsumer\])

 

 

**Kubernetes-Networking**

 

Linux networking: ip link(list interfaces on host), ip address(list ip
addresses on host), ip address show type bridge, ip address add
192.168.1.10/24 dev eth0(set ip address), ip route(display Kernal IP
routing table), ip route add 192.168.2.0/24 via 192.168.1.1(edit
/etc/network/interface file to persist the change),ip route add default
via 192.168.1.1, echo 1 \> /proc/sys/net/ipv4/ip_forward(set the value
to 1 to forward packets between interfaces on Linux host. To persist the
changes after reboots, edit the net.ipv4.ip_forward value to 1 in
/etc/sysctl.conf)

 

DNS in Linux: Add dns entry to host file(/etc/hosts) for DNS resolution.
Point nameservers for DNS lookup to centrally resolve on
host(/etc/resolv.conf).The order of DNS query forwarding to defines in
/etc/nsswitch.conf(hosts: files dns). Forward DNS queries that DNS
server cannot resolve(Forward All to 8.8.8.8). Add search entry with top
level domain name into /etc/resolv.conf to append the top level domain
when you ping from the hostname. 'nslookup' only query from DNS servers
not from host file. 'dig' query from Linux host

 

CoreDNS Installation: 1.Download CoreDNS binaries from Github releases
2.Extract the binary file 3.Run executable to start DNS
server(./coredns) 3.Add DNS entries to /etc/hosts file 4.Configure
Corefile to use /etc/hosts file

 

Process namespace: Processes running inside container are isolated with
namespace. So the container doesn't see any other processes running on
the host, or any other containers. The underlying host has visibility
into all of the processes in containers and host itself. The same
process running in container has different process ID on Linux host.

 

Network Namespaces in Linux: Linux host has its own interfaces, routing
table and ARP table. When a container is created, Docker host creates a
network namespace for it to isolate network information for container
against host and other containers. Network namespace has its own virtual
interfaces, routing, and ARP tables. We can connect two network
namespaces using virtual ethernet pair or a virtual cable(veth) or pipe.

 

ip netns add red(create a new network namespace), ip netns(list ns in
host), ip -n red link(list the network interface in the red ns), ip
netns exec red arp(list arp table in the red ns), ip netns exec red
route(list routes in the red ns)

 

1.Create a virtual interface pair(ip link add veth-red type veth peer
name veth-blue) 2.Attach virtual interfaces in the pair to each network
namespace(ip link set veth-red netns red) 3.Assign ip address to virtual
interfaces(ip -n red addr add 192.168.15.1 dev veth-red) 4.Bring the
virtual interfaces up(ip -n red link set veth-red up) 5.Check
connectivity between namespace(ip netns exec red ping 192.168.15.2, ip
netns exec red arp)

 

When increase number of network namespaces on the host, we create
virtual switch to communicate network namespaces such as Linux bridge
and Open vSwitch.

 

Linux Bridge: To create internal bridge network, this adds an interface
on the host which is the virtual switch for network namespaces. 1.Create
an interface the type set as bridge(ip link add v-net-0 type bridge)
2.Bring up the interface(ip link set dev v-net-0 up) 3.Create new
virtual cable to connect red namespace with bridge network(ip link add
veth-red type veth peer name veth-red-br) 4.Attach veth-red to red
network namespace(ip link set veth-red netns red) 5.Attach veth-red-br
to virtual switch/bridge network(ip link set veth-red-br master v-net-0)
6.Assign ip address to veth veth-red(ip -n red addr add 192.168.15.1/24
dev veth-red) 6.Bring the veth-red up(ip -n red link set veth-red up)
7.Assign an IP address from internal bridge network to virtual switch
interface(ip addr add 192.168.15.5/24 dev v-net-0). Thus all namespace
that connect to virtual switch in the host have connectivity within
bridge network.

 

Further, the host have interfaces for bridge network and LAN network(the
network that connects the all nodes in the cluster). If a container in a
network namespace need to connect another host in LAN network, add a
route to the namespace(ip netns exec blue ip route add 192.168.1.0/24
via 192.168.15.5). Add a new entry into NAT table in the IP tables in
postrouting chain to masquerade in order to reach to external network
from bridge network(iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j
MASQUERADE). If any network namespace from outside network needs to
reach network namespace to inside bridge network, add an ip route entry
onto outside host to reach the inside host or add port forwarding rule
using iptable in the host(iptables -t nat -A PREROUTING - -dport 80 -
-to-destination 192.168.15.2:80 -j DNAT)

 

Docker networking: 1.None network(Containers that don’t attach to any
network)-docker run - -network none nginx 2.Host network(Containers are
attached to host network. No network isolation from host)- docker run -
-network host nginx 3.Bridge network(Docker creates internal private
network called bridge)- docker network ls. Interface name for bridge
network on the host is docker0. docker inspect \<container ID\>(shows
the network namespace has the name starts with random numbers).
Containers inside the docker host can access with port through port
mapping(docker run -p 8080:80 nginx). The port mapping creates
underlying NAT entry in NAT table.(iptables -t nat -A DOCKER --dport
8080 --to-destination 172.17.0.3:80 -j DNAT), list entry in NAT
table(iptables -nvL -t nat)

 

Container Networking Interface(CNI): Set of standards that define how
program(plugin) should be developed for all container solutions to
perform all the networking tasks to attach containers to bridge network
by using single program code/script and how container runtime should
invoke plugins. Network tasks(1.Create network namespace for container,
2.Create Bridge network/interface, 3.Create VETH Pairs 4.Attach vEth to
Namespace 5.Attach Other vEth to Bridge 6.Assign IP addresses to vEth
interfaces 7.Bring the interfaces up 7.Add routes on network namespace
for external network 8.Enable NAT - IP Masquerade). CNI plugins such as
BRIDGE, VLAN, IPVLAN, MACVLAN, WINDOWS as well as IPAM plugin such as
DHCP, host-local. Third party plugins supports these network solutions
such as weaveworks, flannel, cilium, VMware NSX, calico, infoblox.
Docker does not implement with CNI instead it uses Container Network
Model(CNM)

 

Cluster networking: Master and worker nodes should have IP address,
unique hostname and MAC address. Ports that should open in master and
worker nodes(Kube-API server: 6443,Kubelet on master and worker nodes:
10250,Kube-scheduler: 10259,Kube-controller-manager: 10257,ETCD server:
2379,Worker nodes expose NodePort Services for external access:
30000-32767, ETCD peer-to-peer connectivity for multiple master
nodes:2380 ). netstat -nplt(all active Internet connections state, local
address with port, PID/program name). netstat - -help(all commands
available on netstat). netstat -npa (search client connections
established)

 

Pod networking: A networking model has these requirements for pod
networking(1.Every pod have an IP address 2.Every pod should communicate
with every other pod in same node 3.Every pod should communicate with
every other pod in other node using the same IP address without NAT).
Network solution such as weaveworks, flannel, cilium, VMware NSX, calico
creates bridge network and choose unique subnet for each node. Container
runtime looks CNI configuration in /etc/cni/net.d/and runs net-script.sh
script on nodes that includes tasks to add container to bridge network
and connect with other containers in the cluster(Container runtime\>
/etc/cni/net.d/net-script.conflist \> /opt/cni/bin/net-script.sh\>
./net-script.sh add \<container name\> \<namespace ID\>). CNI tells
Kubernetes how add container to the network, delete container interface
and free up IP address.

 

CNI in Kubernetes: Container runtime invokes CNI plugin after the
container is created. CNI plugin is configured in Kubelet service on
each node. All network plugins are installed in /opt/cni/bin. Multiple
plugin configuration files to configure plugin have in /etc/cni/net.d.
Plugin configuration files specify
cniVersion,name,type,isGateway,ipMasq,ipam\[subnet, type -
\[host-local/dhcp\], routes\]

 

Weaveworks(CNI): To avoid configuring routing manually to make for pods
connectivity in the cluster, weaveworks CNI plugins deploys an agent or
service on each node. The agent on each nodes communicate to exchange
information regarding nodes, networks and pods through topology of
entire setup. Weave creates its own bridge on the nodes as WEAVE and
assign IP addresses bridge network. A pod can attach to multiple bridge
such weave bridge, docker bridge. Weave make sure that pods gets the
correct route to reach the agent. Then agent routes the packet to
destination pod through internal network for weave agents. Weave and
weave peers can be deployed as services or daemons on each node or pods
in the namespace kube-system(kubectl apply -f
“<https://cloud.weave.works/K8s/net?k8s-version=$(kubectl> version \|
base64 \| tr -d ‘\n’)”). Deploy weave as daemonsets, it creates for
serviceaccount, cluster role, cluster role binding, role and role
binding. If you deploy the cluster with kubeadm tool and weave plugin,
you can see the weave peers as pods deployed on each node. Weave
manifest file is located in /root/weave. Subnet value for bridge network
specify as environment variable(IPALLOC_RANGE) in daemonset. To check
range of IP addresses configured for PODs in the cluster(cat
/etc/kubernetes/manifests/kube-controller-manager.yaml \| grep
cluster-cidr or kubectl logs weave-net-wmgpt weave -n kube-system)

 

IP address management(IPAM)- Weave: CNI plugin manages IP subnets for
pods on each node without conflicting by storing list of IPs in a
file(ip-list.txt). Code in the script defines command line arguments to
retrieve free IP from that file\[ip = get_free_ip_from_file()\]. CNI
comes with two built-in plugin(DHCP and host-local). host-local manage
IP address locally on each host\[ip = get_free_ip_from_host_local()\].
IPAM section in CNI plugin configuration file-
/etc/cni/net.d/net-script.conf specify type, subnet and routes.
Weaveworks by default allocates IP address from the range
10.32.0.0/12(10.32.0.1 \> 10.47.255.254).

 

Service Networking: Service Networking uses when a pod need access to
another pod on different nodes in the cluster. A service for the pod is
hosted across the cluster and it is not bound to a node, the type is
ClusterIP. If a pod need to access from outside the cluster, the type is
NodePort. Service is a virtual object that run across the cluster that
have no processes, namespaces or interfaces. When service is created, it
is assigned ip address from predefined range(service-cluster-ip-range),
then kube-proxy creates forwarding rule on each node to forward traffic
comes to IP address of service to ip address of the pod. Kube-proxy
creates these rules using userspace, ipvs rules or iptables(kube-proxy
--proxy-mode \[userspace\|iptables\|ipvs\]). IP range assign for
services is specified in kube-apiserver with the option called
service-cluster-ip-range\[ps -aux \| grep kube-api-server\]. Type of
proxy that kube-proxy configured to use(kubectl logs -n kube-system
\<kube-proxy-pod-name\>). Check all rules for services are created by
kube-proxy in iptables as dnat rule(iptables -L -t nat \| grep
db-service). To check range of IP addresses configured for services(cat
/etc/kubernetes/manifests/kube-controller-manager.yaml \| grep
service-cluster-ip-range or cat
/etc/kubernetes/manifests/kube-apiserver.yaml \| grep
service-cluster-ip-range)

 

DNS in Kubernetes: Kubernetes uses DNS server CoreDNS. When a service is
created, the Kubernetes DNS service(CoreDNS) creates a DNS record. Pod
uses DNS record created for the service to access another pod. To access
other pods within namespace(curl <http://web-service>). To access other
pods in another namespace(curl <http://web-service.apps> or curl
<http://web-service.apps.svc>). FQDN for
service(hostname.namespace.type.root=web-service.apps.svc.cluster.local).DNS
records for the name of pods are not created by default. FQDN for
pod(hostname.namespace.type.root=10-243-2-5.apps.pod.cluster.local)

 

CoreDNS in Kubernetes: Prior to Kubernestes 1.12,the DNS server by
Kubernetes is kube-dns. Later to Kubernestes 1.12,the DNS server by
Kubernetes is CoreDNS. CoreDNS implements as pod in kube-system
namespace which have two pods for redundancy as part of ReplicaSet
within deployment. Number of plugins configured in configration file -
Corefile(/etc/coredns/Corefile). Plugins are configured for handling
errors, reporting health, monitoring metrics, cache,
proxy(./etc/resolv.conf) and kubernetes(root domain cluster.local,
creating record for pods converting IPs to dash format). This Corefile
is passed into the CoreDNS pod by using configmap object(coredns). DNS
record for pod or service is added into the CoreDNS database. When we
deploy CoreDNS solution, it creates a service(kube-dns) for CoreDNS
pods. IP address of this CoreDNS service is configured as the nameserver
on pods(/etc/resolv.conf) in the cluster. kubelet configuration
file(/var/lib/kubelet/config.yaml) which includes clusterDNS and
clusterDomain. /etc/resolv.conf has nameserver and search entry. curl
and host are utilities that are similar to nslookup.

 

Ingress: To make a deployment accessible from outside the cluster, point
public DNS to IP address of NodePort service for pods. To avoid end
users to remember port number along with DNS, deploy reverse proxy
server between DNS server and the Kubernetes cluster to redirect web
traffic to NodePort service for on-premise K8s cluster. Deploy a network
load balancer as reverse proxy server to route traffic to LoadBalancer
service ports on all the nodes for cloud based K8 cluster.

 

If you deploy another application in the same cluster and access it from
different URL path(https:my-online-store.com/video), you will have to
implement LoadBalancer service for the deployment with network load
balancer in cloud which will affect cloud bill and have extra burden for
load balancer configuration. Enable SSL for the application at either
application ,load balancer, proxy server level and configuring firewall
rules for new service are also required. Hence Ingress helps to route
traffic to different services within the cluster based on the URL path
and implement SSL security handle within the cluster itself using
another Kubernetes definition file. Even with ingress, you still need to
expose deployments outside the cluster by publishing Ingress as a
NodePort service or cloud native LoadBalancer service. **Ingress
controller** act as layer 7 load balancer that configures SSL
certificates, defining URL based routes(GCP HTTP(S) Load Balancer(GCE),
NGINX, Contour, HAPROXY, traefik and Istio). **Ingress resources**
configure set of rules that is created using definition files such as
pod, deployment and services.

 

Ingress controller: Kubernetes cluster has no ingress controller by
default. It has intelligence built in to monitor Kubernetes cluster for
new ingress resources and configure the underlying NGINX server.

 

Create a deployment with the specific image(NGINX), one replica,
selector and pod definition template for ingress controller. Pod
template has these specifications in spec section.

 

args: The nginx program is stored at location /nginx ingress controller
within the image and pass it as command to start the nginx controller
service, pass ConfigMap as argument for set of configuration such as
path to store the logs, keepalive threshold, SSL settings, session
timeout

 

env: pass the environment variables that carries the pod name and
namespace that is deployed to

 

ports: Specify the ports use by ingress controller

 

Create a service to expose ingress controller to public network where by
using NodePort service or LoadBalancer service and link it to the
deployment using selector.

Create service account with right set of permissions which associate to
Role, RoleBinding, ClusterRole, ClusterRoleBinding

 

Ingress resources: Ingress resource is a set of rules and configurations
applied on the ingress controller(kubectl create -f Ingress-wear.yaml).
Use case 1- Forward all incoming traffic to a single
application(spec.backend.\<serviceName,servicePort\>)

 

Ingress resource -Rules: Use rules when you want route traffic based on
different conditions. Use case 2- Route traffic based on URL
path(spec.rules\[\].http.paths\[\].path: /wear,
backend.\<serviceName,servicePort\>). User directs to default
backend.(default-http-backend) if it doesn't match any rules(service for
display this 404 not found error message). Use Case 3 - Route traffic
based on domain name/host name. Create rules for each
domain/hostname(spec.rules\[\].host: wear.my-online-store.com,
http.paths\[\]. backend.\<serviceName,servicePort\>). You can still have
multiple paths specifications in each of these host to handle different
URL paths. Create ingress resource in the namespace that have services
and deployments. Imperative method: kubectl create ingress INGRESS_NAME
--rule="host/path=service:port\[,tls\[=secret\]\]" \[options\], kubectl
create ingress ingress-pay -n critical-space
--rule="/pay=pay-service:8282"

Ingress - Annotations and rewrite-target: Ingress controller forwards
users to the appropriate application in the backend with rewrite-target
option\[http://\<ingress-service\>:\<ingress-port\>/watch --\> \].
Without the rewrite-target option, this will
happen\[http://\<ingress-service\>:\<ingress-port\>/watch --\> \]. So
applications don't expect /watch or /wear in the URLs, so the request
would fail and throw a 404 not found error. To fix this, specify the
rewrite-target option as replace("/path","/")
\[metadata.annotations.nginx.ingress.kubernetes.io/rewrite-target: /\]

 

Gateway API: Ingress has these limitation no multi-tenancy, no namespace
isolation, no RBAC for features, no resource isolation, only supports
HTTP based rules such as host matching or path matching. Ingress passes
configuration such as SSL redirect, CORS to controllers using complex
annotations specified in Ingress. These configurations are very specific
to the underlying controllers we use such as NGINX or traefik for the
same use case and Kubernetes itself can't validate these settings.
Gateway API comes to resolve those limitation and additionally support
for TCP/UDP routing, Traffic splitting/weighting, Header manipulation,
Authentication, Rate limiting, Redirects, Rewriting, Middleware,
WebSocket support, Custom error pages, Session affinity, CORS. Gateway
API focus on L4 and l7 routing and it represents next generation of
Kubernetes Ingress, Load Balancing, Service Mesh APIs. Gateway APIs
introduces three separate objects.

 

1.GatewayClass - Deploy controller for gateway which defines underlying
network infrastructure (spec.controllerName). 2.Gateway - Deploy
instance of the GatewayClass(spec.gatewayClassName, spec.listeners\[\])
3. HTTPRoute/TCPRoute/GRPCRoute - Deploy route
rule(spec.parentRefs\[\],spec.hostnames\[\],spec.rules\[\].matches\[\].path,spec.rules.matches.backendRefs)

 

Ingress vs Gateway API: Ingress configures SSL redirect(HTTP to HTTPS)
controller specific annotation whereas Gateway API has listeners section
specifies tls configuration without annotations(spec.listeners\[\].tls).
Ingress configures canary deployment to route 20% traffic to secondary
Ingress while route remaining traffic to primary ingress by specific
annotation whereas Gateway API configure HTTPRoute rule to split traffic
between services with percentage in backendRefs
section(spec.rules.backendRefs.weight). Complex controller-specific
annotations needed for CORS settings whereas no annotations needed with
Gateway API(responseHeaderModifier). Gateway controllers are Amazon EKS,
Azure Application Gateway for Containers, Contour,NGINX Gateway Fabric,
Traefik Proxy

 

Kubernetes Gateway API: We use the NGINX Gateway Controller, which
supports all standard Gateway API resources

 

1.Installing Gateway API with NGINX 2. GatewayClass Definition(Define
Gateways regardless the underlying controller, Supports multiple gateway
in single cluster) 3. Configuring HTTP Gateway and Listener(Defines how
traffic enters your cluster specifying protocols, ports, and routing
rules for incoming traffic) 4. HTTP Routing(HTTPRoute defines how HTTP
traffic based on path that enter through listener is forwarded to
Kubernetes services) 5. HTTP Redirects and Rewrites(HTTP to HTTPS
Redirects are used to force traffic to a different
scheme\[spec.rules.filters\], Rewrites modify the request path before
forwarding it to the backend) 6. HTTP Header Modification(Modify HTTP
headers in requests or responses to add, set, or remove specific
headers) 7. HTTP Traffic Splitting(Distribute traffic between multiple
backend services) 8. HTTP Request Mirroring(Send a copy of incoming
requests to a secondary service for testing or analysis, without
affecting the primary service) 9. TLS Configuration(Configure a Gateway
to terminate TLS traffic) 10. TCP, UDP, and Other Protocols(Gateway API
supports TCP\[connection-oriented protocol often used for applications
like databases\], UDP\[connectionless protocol often used for DNS or
streaming applications\], and even gRPC\[gRPC is a high-performance RPC
(Remote Procedure Call) framework often used in microservices\])

 

 

**Kubernetes - Helm and Kustomize**

 

Helm: To avoid complexity at micromanaging and troubleshooting complex
infrastructure that need collection of
objects(deployment,PV,PVC,service,secret) with separate YAML file. Helm
is built to ground up to manage all Kubernetes objects as a package
manager and also as release manager to help upgrade or rollback
applications. Install and mange WordPress app package by decalring
desired values at installed time in single location such as size of PV,
name of app, admin password of database in a file like values.yaml
(1.helm install wordpress - install entire app 2.helm upgrade
wordpress - upgrade the application where Helm knows what objects need
to change 3.helm rollback wordpress - roll back to previous revision as
Helm tracks the all the changes made to the app files 4.helm uninstall
wordpress - uninstall the apps)

 

Helm Installation and configuration: Prerequisites- A functional
Kubernetes cluster, kubectl installed, login details set up the
kubeconfig file. Helm can be installed on Linux, Windows or MacOS
systems. For Linux(sudo snap install helm --classic). Add key and
sources list before installing Helm for APT based system such as Debian
and Ubuntu. Helm now has an installer script that will automatically
grab the latest version of Helm and install it locally.

 

Helm 2: Helm client communicated with Tiller when it performs Helm
specific operation and to support features RBAC, CRD. Tiller has full
privilages on kubernetes cluster which is security concern. Every
changes make on the cluster referred as revision which is snapshot of
exact stage of Kubernetes package. Control the revision to place on
required stage on application. Manual changes to Helm chart do not track
with revision

 

Helm 3: Removed Tiller placed between Helm client and Kubernestes
cluster with having features RBAC, CRD. Has same RBAC permission
whatever tool uses such as kubectl, or with Helm. Manual changes to Helm
chart track with live state of Kubernetes objects in YAML form whcih is
called Supports 3-Way Strategic Merge Patch. Can perform upgrade or
rollback of Helm operation with this feature

 

Helm Components: Helm CLI utility - to perform Helm actions such as
installing a chart, upgrade, rollback. Helm Charts- A collection of
files that contain all the instructions for Helm to create collection of
objects for app in Kubernetes cluster. Release - A single installation
of an application applied to clsuter using a Helm chart. Revision - a
snapshot of the application when changes make to application.

 

Helm charts: Helm is automation tool that will go through all the
required steps to install application. Download publicly available Helm
chat from public repository. Helm saves metadata such as the releases
that it installed, the charts used, revision states directly in the
Kubernetes cluster as Kubernetes secrets. The configurable parameters
such as image name and number of replicas are specified in a file
values.yaml which is called templating. Chart.yaml contains information
about chart itself such as API version(Helm2:v1.Helm3:v2.chart that does
not have apiVersion value set, the chart is built for Helm2), app
version(Version of the application), version(its own version of chart),
name(name of the chart), description, type(application:for deploying
applications,library:provides utilities that help in building
charts),dependencies(If the application has dependencies with database
or other),keywords(keywords associated with this
project),maintainers(information about chat repositories,its name and
email). Helm Chart Structure has template directory which have
values.yaml(configurable values),Chart.yaml(Chart
information),LICENSE(chart license),README.md(chart in a human readable
form),charts(Dependency charts)

 

Helm Releases: helm install \[release-name\] \[chart-name\] e.g. helm
install my-site bitnami/wordpress , It allows install multiple releases
based on the same chart. A releases can be tracked and changed
independently as different entities. This is useful to maintain
production site and development site with releases.

 

Helm Repositories: hosts thousand of charts in repositories such as
Appscode, Community Operators, TrueCharts, Bitnami. All of these
repositories have listed their charts in Helm Hub or Artifact
Hub(artifacthub.io)

 

Working with Helm: To invoke the Helm CLI(helm --help). Find charts that
has the official or a verified publisher badge. Find charts from in
artifacthub.io or specific repository(helm search hub\|repo wordpress).
Add chart repository to local Helm setup so that helm install commands
finds where the charts must be installed from(helm repo add bitnami
<https://charts.bitnami.com/bitnami>). List repos added to local Helm
setup(helm repo list). Install the chart from local Helm setup(helm
install my-release bitnami/wordpress). List all existing releases with
current revision number, chart and app version(helm list). Remove all
Kubernetes object added by release(helm uninstall my-release). Pull
updated repositories from online repository to local computer(helm repo
update). List revision numbers of the release(helm history \<release
name\>)

 

Customizing chart parameters: Customize chart configurable parameters
with --set option to values.yaml file by overriding the existing values
in the file. Use multiple times to pass multiple parameters(helm install
--set wordpressBlogName="Helm Tutorials" my-release bitnami/wordpress).
If there are too many of values, move these values to custom
custom-values.yaml file and specify the file with --values option(helm
install --values custom-values.yaml my-release bitnami/wordpress). helm
install command performs both pull the charts from online repository and
apply the charts. To modify the built-in values.yaml file itself instead
of using the command line option --set or the custom values file with
--values(1.pull the chart in an archived:helm pull bitnami/wordpress
2.unarchive all the files parts of the chart: helm pull --untar
bitnami/wordpress 3.edit default values in values.yaml file in text
editor 4.run hem install command specifying the path of local:helm
install my-release ./wordpress)

 

Lifecycle management with Helm: Manage multiple releases of same chart
in single kubernetes cluster independently. Specify the chart version to
use specific app version on helm install command(helm install
nginx-release bitnami/nginx --version 7.1.0). Upgrade the release of a
helm chart with helm upgrade command by updating revision(helm upgrade
nginx-release bitnami/nginx). Rollback the release to the state that was
in specified revision number(helm rollback nginx-release 1). All the
rollbacks are very similar to a Backup Restore feature. It doesn't cover
file or directory data that may be created by our applications. Instead,
Helm backs up and restores the declarations or manifest files of
Kubernetes objects. Using chart hooks take consistent backups of
databases before upgrading charts.

 

Summary:

 

\$ helm search hub wordpress

\$ helm repo add bitnami <https://charts.bitnami.com/bitnami>

\$ helm repo list

\$ helm install nginx-release bitnami/nginx --version 7.1.0

\$ helm list

\$ helm upgrade nginx-release bitnami/nginx --version 13

\$ helm history nginx-release

\$ helm rollback nginx-release \<revision number\>

\$ helm uninstall nginx-release

 

**Kustomize Basics**

 

Kustomize Problem Statement & ideology: Kustomize was created to modify
Kubernetes configs on per environments basis without copying them into
different directories of environment. Base config represents default
value across all environments that can override later. Overlays allows
to customize the parameters/properties on a per environment basis and
add new resources exclusively for that specific environment. Folder
structure: base/ and overlays/dev, overlays/stg. Kustomize comes
built-in with kubectl but you still want to install the kustomize CLI to
get the latest version. Kustomize uses standard YAML file whereas helm
charts uses complex go templating.

 

Kustomize vs Helm: Helm is a package manager for apps more than
customizing configurations on a per environment basis.Helm maintains
sperate values.yaml file on a per environment basis under the
environments directory and all of Kubernetes manifests under templates
directory where specify values for each environment using go templating
syntax {{ .Values.replicaCount}}. Kustomize uses standard YAML file
which is simple to read

 

Installation/Setup: Prerequisites- A functional Kubernetes cluster,
kubectl installed, login details set up the kubeconfig file. Can be
installed on a Linux, Windows or a Mac. Kustomize team provides nice
script that will automatically detect your operating system and install
the appropriate version of Kustomize(\$ curl -s
"<https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh>"
\| bash )

 

kustomization.yaml File: Kustomize always looks for specific file
kustomization.yaml file in root directory and import the list of all
Kubernetes resources files specified in kustomization.yaml file, the
apply the transformations in dry-run mode e.g transformers, patches.
Point kustomize to the K8s directory using kustomize build
command(kustomize build k8s/) where command output throws how final
configs would look like in dry-run mode after applying transformations.

 

Kustomize Output: The kustomize build command does not apply/deploy the
Kubernetes resources to the cluster so that output of kustomize build
command needs to be redirected to the kubectl apply command(kustomize
build k8s/ \| kubectl apply -f -). Apply the configs to the cluster
natively with the kubectl tool(kubectl apply -k k8s/ ). Build all the
configs and delete them from the cluster(kustomize build k8s/ \| kubectl
delete -f -). Delete the configs to the cluster natively with the
kubectl tool(kubectl delete -k k8s/)

 

Kustomize Api Version & Kind: Set the API
version(kustomize.config.k8s.io/v1beta1) and kind(Kustomization)
properties in a Kustomization.yaml. They are technically optional. But
it is recommended that you hard code these values to be compatible in
the future.

 

Managing Directories: Kustomize allows organizing Kubernetes manifest
files that spread across multiple sub directories based on application
or environment. Then manage create/delete resources navigating sub
directories using kubectl apply/delete command(kubectl apply -f k8s/api
-f k8s/db). Kustomize helps avoid going each one of sub directories and
run kubectl apply by creating kustomization.yaml file in root directory
with all mainfest files in sub directories in resources section. To neat
resources in kustomization.yaml when sub directories grow, list all of
the manifest files within specific kustomization.yaml file resides in
the sub directory and import them into kustomization.yaml file in root
directory. Thus we can apply transformation through kustomization.yaml
file at root or sub directory level.

 

Common Transformers: To modify or transform Kubernetes configs for all
Kubernetes resources file at one time at root or sub directory level.
Kustomize imports resources specified in kustomization.yaml file in root
or sub directory and apply transformers. Kustomize has built in
transformers and supports custom transformers. Adds labels to all
resources(commonLabels.org: KodeKloud). Adds a common
namespace(namespace: lab).Add common name prefix and name
suffix(namePrefix: KodeKloud-, nameSuffix: -dev). Add common
annotation(commonAnnotations.branch: master)

 

Image Transformer: To modify an image name or tag or both together in
specific deployment or
container(images\[\].name,newName),(images\[\].name,newTag),(images\[\].name,newName,newTag).
Specify image tage as string within ""

 

Patches Introduction: Kustomize patches provide surgical approach to
target one or more specific sections in a Kubernetes resource and modify
Kubernetes configs. Ways to define a patch are JSON 6902 and Strategic
Merge Patch.

 

JSON 6902 patch has these parameters must be provided(1.target: match
resource that patch should be applied on with different properties such
as Kind,Version/Group,Name,Namespace,labelSelector,AnnotationSelector
2.Operation type(op): add/remove/replace 3.path: path to specific
property to replace/update in YAML tree 4.value: the value that will
either be replaced or added with).

 

Strategic Merge Patch has standard Kubernetes config with original
manifest file by deleting all the config properties that don't want to
change. The new config is merged to old config. Strategic Merge Patch
has two things must be provided.(1.What Kubernetes object that you want
to update 2.The specific properties that you want to change)

 

Types of patches: 1.Inline(patches: \|-): define the patch within the
kustomization.yaml file itself 2. Seperate file: define the patch by
using separate file where we provide target and a path to a YAML
file(replica-patch.yaml). For seperate file; JSON 6902:
patches\[\].path: replica-patch.yaml and Strategic Merge Patch:
patches\[\]replica-patch.yaml

 

Patches Dictionary: JSON 6902(replace: specify the existing key in path
and the value need to change in value section, add: specify the new key
in path and the value need to add in value section, remove: specify the
existing key in path that need to remove and no need specify value)
Strategic merge patch(replace: Only add the property that you want to
change into sperate YAML file, add: Only add the property that you want
to add into sperate YAML file, remove: Specify the value of key is null
to delete the key in sperate YAML file)

 

Patches list: JSON 6902(replace: specify the index number that you want
to update and the value need to change in value section, add: specify -
symbol to append to end of list or index which place in that list you
want to add value, remove: specify index number in path which item in
that list you want to delete) Strategic merge patch(replace: Only add
value for existing property that you want to replace in to sperate YAML
file, add: Only add value with property that you want to add in to
sperate YAML file, remove: Specify \$patch and then delete to delete the
item with property in the list in sperate YAML file)

 

Overlays: Kustomize was created to allow us to import base Kubernetes
configs and customize it on a per-environment basis using overlays. A
Kustomized project can break into folder structure, base and overlays.
base - contains all of default or share Kubernetes configuration across
all environments. overlays - Specify all of the environment specific
configs so that we can import the base config and modify for the
specific environment. In overlays directory, you specify bases property
with relative path(../../base) to go up to base directory in
Kustomization.yaml. Additionally, you can reside new configs under the
resources section in kustomization.yaml file that doesn't exist in the
base directory

 

Components: Components provides the ability to define a reusable piece
of configuration logic(resource + patches) that can be imported to
subset of overlays not all overlays. Instead creating and updating
configs in required overlays, components allow to centrally manage
configs. Create extra directory called Components like base and store
all of components in that directory. 1. Create kustomization.yaml for
required feature in sub directory in Components directory 2. Create
kustomization.yaml for required environment in overlays directory and
specify bases components with relative path(../../components/db) to go
up to components directory in Kustomization.yaml to import Kubernetes
config

 

 

 

**Kubernetes - Troubleshooting**

 

Application Failure: Check Accessibility(curl
[http://web-service-ip:node-port](http://web-service-ip:node-port)),
Check service status( describe service command output must show
endpoint. Labels of pod should match with selector of service, match
TargetPort of service with port on container), For two tier application,
web pod connects to credentials of database pod with environment
variables where check the environment variable is correct. Check
pod(kubectl get \| describe \| logs - running state and the number of
restarts, kubectl logs web -f, kubectl logs web -f --previous - to view
the logs of a previous pod if pod restarts), Check dependent service and
applications(kubectl describe service db-service, kubectl get \|
describe \| logs ), kubectl config set-context --current
--namespace=\<namespace name\>

 

Control Plane Failure: Check Node Status, Check Controlplane
Pods(kubectl get pods -n kube-system), Check Controlplane Services- If
the control plane components are deployed as services(service
kube-apiserver \|kube-controller-manager\|kube-scheduler status), Check
the status of services such as kubelet and kube-proxy(service
kubelet\|kube-proxy status), Check Service Logs (kubectl logs
kube-apiserver-master -n kube-system), use journalctl utility to view
service logs(sudo journalctl -u kube-apiserver)

 

Worker Node Failure: Check Node Status- kubectl describe node worker-1:
Conditions(OutOfDisk, MemoryPressure, DiskPressure, PIDPressure, Ready),
Check for possible CPU, memory and disk space(top, df -h), Check Kubelet
Status(service kubelet status), Check the kubelet logs(sudo journalctl
-u kubelet, sudo journalctl -u kubelet -f), Check kubelet Certificates
and ensure they are not expired and part of right group and the
certificates are issued by right CA(openssl x509 -in
/var/lib/kubelet/worker-1.crt -text)

 

Network Troubleshooting: CoreDNS's memory usage is affected by number of
Pods and Services in the cluster, size of the filled DNS answer cache,
rate of queries received(QPS). coredns deployment has Corefile plugin
that consists of important configuration. proxy . /etc/resolv.conf
forward out of cluster domains directly to right authoritative DNS
server on host.

 

Troubleshooting issues related to coreDNS(1. If you find CoreDNS pods in
pending state first check network plugin is installed 2. coredns pods
have CrashLoopBackOff or Error state if you have nodes that are running
SELinux with an older version of Docker. To solve that upgrade to a
newer version of Docker, disable SELinux, modify the coredns deployment
to set allowPrivilegeEscalation to true or as quick fix edit your
Corefile, replacing forward . /etc/resolv.conf with the IP address of
your upstream DNS 3.If CoreDNS pods and the kube-dns service is working
fine, check the kube-dns service has valid endpoints- kubectl -n
kube-system get ep kube-dns).

 

kubeproxy is responsible for watching services and endpoint associated
with each service which is responsible for sending traffic to actual
pods. Kubeproxy fetches the configuration from a configuration file ie,
/var/lib/kube-proxy/config.conf where we define the clusterCIDR,
kubeproxy mode, ipvs, iptables, bindaddress, kube-config.
Troubleshooting issues related to kube-proxy(1.Check kube-proxy pod in
the kube-system namespace is running 2.Check kube-proxy logs 3.Check
configmap is correctly defined and the config file for running
kube-proxy binary is correct 4.kube-config is defined in the config map
5.Check kube-proxy is running inside the container-netstat -plan \| grep
kube-proxy)

 

 

**YAML**

 

Human friendly data serialization standard for all programming language.
Use to represent the configuration data. Syntax : strict indentation.
Consists with key value pair. An array(-) lists elements under it. A
dictionary is a set of properties of single object. Properties are
defined in key-value format. Dictionary In Dictionary. List of
dictionaries: Expand each item in the list with dictionaries. Dictionary
is unordered collection means changing order of properties is equal,
arrays are ordered collection means changing order of items is not
equal. Any line begin with Hash \# is considered as comment.

 

**JSON PATH**:

JSON uses braces or curly brackets to organize data into lists and
dictionaries. JSON uses square brackets to define a list and each item
within the list is separated by comma. Convert data in YAML to JSON or
JSON to YAML using online converter(https://www.json2yaml.com). JSON
path is query language that can help you parse data represented in a
JSON or YAML like SQL.

 

Query with JSON PATH:

 

Dictionaries: Extract properties of dictionaries and dictionaries of
dictionaries with dot notation. JSON has root dictionary which has no
name is known as the root element of a JSON document. The root element
is denoted by a dollar \$ (\$.vehicles.car). All results of JSON path
query are encapsulated within an array means within square bracket.

 

Lists: Use square bracket in your query with root element \$ mark and
specify the position of the item. The indexes start at zero (\$\[3\],
\$\[0,3\]). get all names from the first to the fourth element, use
"colon": \$\[0:4\] . Use the step option by adding another colon and
specifying the number of hubs to take after each item: \$\[0:8:2\] . Get
the last element: \$\[-1:0\] or \$\[-1:\] . Get the last 3 elements:
\$\[-3:\] or \$\[-8:-2\]

 

Dictionary & Lists: Use dot notation and square bracket in your query
with root element \$ mark (\$.car.wheels\[1\].model)

 

Criteria: Apply criteria or conditions to query. Check if -\> ? () ,
each item in the list -\> @ . Operators: equal to 40 each item: @ == 40,
not equal to 40 each item: @ != 40, numbers either 40,43,45 from each
item: @ in \[40,43,45\], numbers are not either 40,43,45 from each item:
@ nin \[40,43,45\] example: \$.car.wheels\[?(@.location ==
"rear-light")\].model , cat q1.json \| jpath '\$.property1'

 

Wildcard: use star \* for wildcard meaning all or any to query values of
specific property. Dictionary: \$.\*.price , List of Dictionaries:
\$\[\*\].model , List & Dictionaries: \$.\*.wheel\[\*\].model example:
'\$.\*\[\*\].laureates\[\*\].firstname'
'\$.\*\[?(@.year=="2014")\].laureates\[\*\].firstname'

 

JSON PATH Use case - Kubernetes: Focus how to use JSON path queries with
the kubectl utility. Mainly view specific fields of all resources, query
data about resources based on different criteria. Kube-apiserver returns
the requested information in JSON format, kubectl converts it into a
human-readable format and prints it out to our screen. Use -o wide
option with kubectl get command for additional details. Use kubectl
describe command to get more details such as resource capacity available
on nodes, taint set on nodes, conditions on node, the hardware
architecture, container image available on nodes. None of the built-in
commands bring those output seperately.

 

How to JSON PATH in KubeCtl: 1.Identify the kubectl command- required
information in the raw format 2.Familiarize with JSON output- output in
JSON format( -o json or -o=jsonpath=) 3. Form the JSON PATH query- build
JSON PATH query retrieve the required information where ignore \$ sign
4. Use the JSON PATH query with kubectl - encapsulate the JSON PATH
query within pair of single quotes and curly braces example: kubectl get
pods -o=jsonpath= '{.items\[0\].spec.containers\[0\].image}' . JSON PATH
query evaluator like jsonpath.com. Use {"\n"} for New line, {"\t"} for
Tab to arrange the output. example: kubectl get nodes
-o=jsonpath='{.items\[\*\].metadata.name} {"\n"}
{.items\[\*\].status.capacity.cpu}'

 

Loops: Use loops to iterate through items in a list and print properties
of each item. Use the range keyword to specify the for each statement in
loops and end the loop end keyword example: kubectl get nodes
-o=jsonpath='{range .items\[\*\]} {.metadata.name} {"\t"}
{.status.capacity.cpu} {"\n"} {end}'

 

JSON PATH for Custom Columns: This is an easier approach than using the
loop method and it allows to add custom columns into output. kubectl get
nodes -o=custom-columns=\<COLUMN NAME\>:\<JSON PATH\> kubectl get nodes
-o=custom-columns=NODE:.metadata.name ,CPU:.status.capacity.cpu

Note that you must exclude the item section of the query as the
custom-columns assumes the query is for each item in the list. Similarly
you can add additional column and JSON PATH pairs separated by a comma.

 

JSON PATH for Sort: sort objects by specifying the --sort-by option
where sort the output based on value of a properties example: kubectl
get nodes --sort-by= .metadata.name

 

 
