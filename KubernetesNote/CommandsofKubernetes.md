# Commands of Kubernetes

Node

\`\`\`

\- kubectl get node - To list down all worker nodes.

\- kubectl delete node \<node_name\> - Delete the given node in cluster.

\- kubectl top node \<node-name\> - Show metrics for a given node.

\- kubectl describe nodes - Describe all the nodes in verbose.

\- kubectl get pods -o wide - List all pods in the current namespace,
with more details.

\- kubectl get node -o wide - List all the nodes with mode details.

\- kubectl describe node - Describe the given node in verbose.

\- kubectl annotate node \<node_name\> - Add an annotation for the given
node.

\- kubectl label node - Add a label to given node

\`\`\`

 

Pod

\`\`\`

\- kubectl get pod - To list the available pods in the default
namespace.

\- kubectl get pods -o wide - To list the available pods with more
details

\- kubectl describe pod \<pod_name\> - To list the detailed description
of pod.

\- kubectl delete pod \<pod_name\> - To delete a pod with the name.

\- kubectl delete -f pod.yaml - If create pod from file we can use this
method

\- kubectl create pod \<pod_name\> - To create a pod with the name.

\- Kubectl get pod -n \<name_space\> - To list all the pods in a
namespace.

\- Kubectl create pod \<pod_name\> -n \<name_space\> - To create a pod
with the name in a namespace.

\- kubectl logs \<podname\> -n namespace - check pod logs in ns

\- kubectl expose deployment nginx --port=80 --type=NodePort - Expose
PODs as services (creates endpoints)

\- kubectl exec -it \<pod\> - Log into pod

\- kubectl get pods --field-selector=status.phase=Running - List all
running pods in a namespace

\- kubectl get pod \<pod-name\> —watch - view pod in watch mode

\- kubectl get pod -A —watch - view all pod in watch mode

\- kubectl get pods -o json - json output

\- kubectl get pods —all-namespaces - List all pods in all namespaces

\- kubectl get pods -A - List all pods in all namespaces

\- kubectl run busybox —image=busybox — sleep 1000 - stop container
after 1000ms

\- kubectl logs \<pod\> -c \<container\> - view container logs in a
pod(if have more than one container)

\- kubectl top pod \<pod\> - show metrics for a given pod

\- kubectl top pod \<pod\> —containers - show metrics for a given pod
and all its containers

\- kubectl explain pod - get the documentation for pod manifests

\`\`\`

 

Deployment

\`\`\`

\- kubectl create deployment myngix --image:nginx - To create a new
deployment.

\- kubectl run mynginx --image=nginx - Create single deployment

\- kubectl apply -f \[yml-file\] - Create deployment

\- kubectl get deployment - To list one or more deployments.

\- kubectl get deployment \[dep-name\] --watch - watch a specific
deployment

\- kubectl get deployment -A - list all deployment

\- kubectl describe deployment \[dep-name\] - To list a detailed state
of one/more deployments.

\- kubectl delete deployment \[dep-name\] - To delete a deployment.

\- kubectl set image deployment/nginx nginx=nginx:1.9.1 - Rolling update
nginx of deployment

\- kubectl scale —replicas=5 deployment/\[dep-name\] - scale up a
deployment

\- kubectl autoscale deployment/\[dep-name\] --min=10 --max=15
--cpu-percent=80

\- kubectl rollout undo deployment/\[dep-name\] - rolling back to
previous one

\- kubectl rollout undo deployment/\[dep-name\] --to-revision=2 -
rolling back to sepecific one

\- kubectl rollout status deployment/\[dep-name\] - check rollout status

\- kubectl rollout history deployment/\[dep-name\] - Check rollout
history

\- kubectl edit deployment/myngix - edit deployment with editor

\- kubectl port-forward deployment/\[pod-name\]
\[localhost-port\]:\[pod-port\] - port forwarding

\`\`\`

 

DeamonSets

\`\`\`

\- kubectl get ds - To list out all the daemon sets.

\- kubectl get ds -all-namespaces - To list out the daemon sets in a
namespace.

\- kubectl describe ds \[daemon_name\]-n \[names_name\] - detailed
information for a daemon set

\`\`\`

 

Configmaps

\`\`\`

\- kubectl create configmap \[map-name\] \[file-directory\] - create
configmap

\- kubectl describe configmaps \[conf-name\] - get more details

\- kubectl get configmap - list configmaps

\- kubectl get configmap \[name\] -o yaml - get configmap in YAML

\`\`\`

 

Services

\`\`\`

\- kubectl get services - To list one or more services.

\- kubcetl get svc -o wide - To list the available service with more
details

\- kubectl describe services \[svc-name\] - To list the detailed display
of services.

\- kubectl delete service \[svc-name\] - To delete a particular service.

\- kubectl explain service - get the documentation for service manifests

\- kubectl port-forward svc/\[pod-name\]
\[localhost-port\]:\[pod-port\] - port forwarding

\- kubectl create service nodeport \[service-name\] - creates a Service
with subtype NodePort

\`\`\`

 

Service Account

\`\`\`

\- kubectl get serviceaccounts \[name\]- To List Service Accounts.

\- kubectl describe serviceaccounts \[name\] - detailed state of
one/more service accounts.

\- kubectl replace serviceaccounts \[name\] - To replace a service
account.

\- kubectl delete serviceaccounts \[name\] - To delete a service
account.

\`\`\`

 

Secrets

\`\`\`

\- kubectl get secrets - To display the secrets

\- kubectl describe secrets/\[secret\] - describe secret

\`\`\`

 

Namespace

\`\`\`

\- kubectl create namespace \[namesp_name\] - To create a namespace by
the given name.

\- kubectl get namespace - To list the current namespace in a cluster.

\- kubectl describe namespace \[namesp_name\] - display detailed state
of one/more namespaces.

\- kubectl delete namespace \[namesp_name\] - To delete a namespace.

\- kubectl edit namespace \<\[namesp_name\] - To edit and update the
definition of a namespace.

\- kubectl create -f namespace.yml - create namespace using yml file

\`\`\`

 

Replicasets

\`\`\`

\- kubectl get replicasets - To List down the ReplicaSets.

\- kubectl describe replicasets \[replica_name\] - list down the
detailed state of one/more rs

\- kubectl scale - replace=\[x\] - To scale a replica set.

\- kubectl port-forward rs/\[pod-name\]
\[localhost-port\]:\[pod-port\] - port forwarding

\`\`\`

 

Volumes

\`\`\`

\- kubectl get pv - Check pv

\- kubectl describe pv \[pv-name\] - Describe pv

\- kubectl get pvc - Check PVC

\- kubectl describe pv \[pv-name\] - Describe pvc

\`\`\`

 

Ingress

\`\`\`

\- kubectl get ingress - Commands to manage Ingress for ClusterIP
service type

\`\`\`

 

Labels

\`\`\`

\- kubectl get nodes - show-labels - List assigned labels on the node

\- kubectl label nodes \[node-name\] \[label\] - Add label to node

\- kubectl label node \[node-name\] \[label\]- - Remove label from a
node, same command but you see minus sign with the label name

\`\`\`

 

Events

\`\`\`

\- kubectl get events -A - list events

\- kubectl get events -o json - get json output

\`\`\`

 

Events

\`\`\`

\- kubectl get events \| grep \[pod-name\] - get events from pod

\- kubectl describe pod \[pod-name\] - this also show events

\`\`\`

 

Annotations

\`\`\`

\- kubectl describe deployment/\[dep-name\] \| grep Annotations - get
annotations

\- kubectl annotate pods \[current-ano-name\]
description='new-description' - update annotations

\`\`\`

 

Taints

\`\`\`

\- kubectl taint \[node_name\] \[taint_name\] - used to update the
taints on one or more nodes.

\`\`\`

 

Common Metrics Command

\`\`\`

\- kubectl top node \[node-name\] - Show metrics for a given node.

\- kubectl top pod \[pod\] - show metrics for a given pod

\- kubectl top pod \[pod\] - -containers - show metrics for a given pod
and all its containers

\`\`\`

 

Common Logs command

\`\`\`

\- kubectl logs \[pod-name\] -n namespace - check pod logs in ns

\- kubectl logs \[pod-name\] \[container-name\] - container logs in a
pod(if more than one cont)

\- kubectl logs \[pod-name\] - all-containers - logs from all containers
in pod

\- kubectl logs - since=1h \[pod-name\] - get logs from last hour

\- kubectl logs - tail-20 \[pod-name\] - To display the most recent 20
lines of logs.

\- kubectl logs \[pod-name\] pod.log - To save the logs into a file
named as pod.log.

\- kubectl logs -l my-label=my-value - all-containers - get logs from
according to the labels

\- kubectl logs deployment/\[dep-name\] - get logs from deployment

\- kubectl logs job/\[job-name\] - get logs from job

\- kubectl logs \[pod-name\] \> my-pod-logs.txt - get logs as a text
file

\`\`\`
