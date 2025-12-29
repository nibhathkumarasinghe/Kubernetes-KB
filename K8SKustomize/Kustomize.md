# Kustomize Basics

 

**Kustomize Problem Statement & ideology**

 

***Why Kustomize***

 

**-**In this case, we have nginx deployment .yml file where we're going
to deploy one single nginx pod to multiple environments for development,
staging and production. Assume that we want to customize the deployments
so that it behaves a little bit differently in each one of environments
such as modifying number of replicas on per environment basis.

-One of the simplest solutions to this problem is to create three
separate directories for each environment within the respective folder.

-Duplicate the configs across all of the three different environments by
modifying attributes on per environment basis

 

<img src="./images/media/image1.jpeg"
style="width:6.26806in;height:2.95556in"
alt="dev/nginx.yml stg/nginx.yml prod/nginx.yml apiVersion: apps/v1 apiVersion: apps/v1 apiVersion: apps/v1 kind: Deployment kind: Deployment kind: Deployment metadata: metadata: metadata: name: nginx-deployment name: nginx-deployment name: nginx-deployment spec : spec : spec : replicas: 1 replicas: 2 replicas: 5 ...... selector: selector : selector : matchLabels: matchLabels: matchLabels: component: nginx component: nginx component: nginx template: template: template: metadata: metadata: metadata: labels: labels: labels: component: nginx component: nginx component: nginx spec: spec : spec : containers: containers: containers: - name: nginx - name: nginx - name: nginx image: nginx image: nginx image: nginx " />

 

-Then store these configs specifying the specific environment directory

 

\$ kubectl apply -f dev/

\$ kubectl apply -f stg/

 

-Whenever we want to add .yml file for number of resources to each
directories and maintain those file, that is too tedious task and harder
to maintain. This is one of reasons why kustomize was created so that we
can modify Kubernetes configs on per environments basis without copying
them into different directories of environment.

 

***Base***

 

-Base config represents config that's going to be identical across all
of environments.

-Base config represents default value across all environments and can
override later

 

***Overlays***

 

-Overlays allow us to customize the behaviour on a per environment basis

-Can create overlays for each environments

-Specify all of the parameters or properties that we want to override or
change from the base config

 

<img src="./images/media/image2.jpeg"
style="width:6.26806in;height:3.37847in"
alt="Overlays base overlays/dev apiVersion: apps/v1 kind: Deployment spec : metadata: name: nginx-deployment replicas: 1 spec: replicas: 1 overlays/prod selector: matchLabels: spec : component: nginx template: replicas: 5 metadata: overlays/stg labels: component: nginx spec: spec: containers: ¡replicas: 2 - name: nginx image: nginx " />

 

***Folder Structure***

 

base/

 

\- base directory contains all of your base configurations to share
across all environments such as deployments, services

 

overlays/

 

-It has a different folder for each environment

-Each folder has the values that you want to override and change from
the base config

-It has any new resources that should only be added exclusively for that
specific environment.

 

***Kustomize***

 

<img src="./images/media/image3.jpeg"
style="width:6.26806in;height:3.59861in"
alt="Folder Structure K8s/ Share or default k8s/ configs across all base/ environments kustomization.yaml nginx-depl.yaml service.yaml redis-depl.yaml overlays/ dev/ [ kustomization.yaml config-map.yaml stg/ kustomization.yaml Environment specific config-map.yaml configurations that add or prod/ modify the base configs 1 kustomization.yaml config-map.yaml " />

 

<img src="./images/media/image4.jpeg"
style="width:6.26806in;height:2.36875in"
alt="Kustomize + Base Overlay Final Manifests " />

 

-You have base config and overlays, then Kustomize refers both of them
and create the final Kubernetes manifests.

-Kustomize comes built-in with kubectl so no other packages need to be
installed

-You may still want to install the kustomize CLI to get the latest
version because Kubectl doesn't always come with the latest version

-With Kustomize, it does not require learning how to use any templating
systems like Helm. Instead use to define base configs and then specify
overlays for different environments.

-It uses standard YAML file which is readable whereas some of complex
helm charts is difficult to read because of the templating systems

-Every artifact that Kustomize uses is plain YAML and can be validated
and processed as it doesn't have special syntax and templating language.
It is so simple.

 

 

**Kustomize vs Helm**

 

-Helm makes use of go templating to allow assigning variables to
properties

-It specifies with two curly braces {{ .Values.replicaCount}}

 

<img src="./images/media/image5.jpeg"
style="width:6.26806in;height:3.74514in"
alt="Kustomize vs Helm Deployment.yaml values.yaml apiVersion: apps/v1 kind: Deployment replicaCount: 1 metadata: name: {{.Values.name }} image: tag: &quot;2.4.4&quot; spec : replicas: |{{.Values.replicaCount }} selector: matchLabels: app: {{.Values.name }} template: metadata: labels: app: {{ .Values.name }} spec : containers: - name: {{ .Chart.Name image: [&quot;nginx:{{.Values.image.tag }}&quot; " />

 

In traditional Helm project structure, it contains following.

 

\- It maintains sperate values.yaml file on a per environment basis
under the environments directory

\- all of Kubernetes manifests under templates directory where we have
to insert variables required for each environment using go templating
syntax

 

<img src="./images/media/image6.jpeg" style="width:5in;height:6.09167in"
alt="Deployment. yaml k8s/ environments/ E values.dev.yaml values.stg.yaml values.prod.yaml templates/ nginx-deployment.yaml nginx-service.yaml db-deployment.yaml db-service.yaml " />

 

 

-Helm is more than just a tool to customize configurations on a per
environment basis. Helm is also a package manager for app. It is like
either YUM and APT for a Linux system

-Helm provides extra features that Kustomize doesn't have like
conditionals, loops, functions and hooks

-Helm templates are not valid YAML as they use go templating syntax

-Complex templates become hard to read

 

One of the perks about Kustomize is that Kustomize is very simple. It's
very easy to read, it's just regular YAML. The regular base
configurations is just regular Kubernetes configs, and then the
overlays, where we go to modify it on a per environment basis, that's
all valid YAML as well whereas Helm is bit more complex but it has more
features. There are trade-offs that each one bring to the table.

 

 

**Installation/Setup**

 

Before installing Kustomize,

 

-It must have Kubernetes cluster up and running

-Installed kubectl on the local machine and configured to connect
Kubernetes cluster

 

-can be installed on a Linux, Windows or a Mac machine

-Kustomize team make it easy to install Kustomize by providing a nice
script that will automatically detect your operating system and install
the appropriate version of Kustomize.

 

\$ curl -s
"<https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh>"
\| bash ;download and run the script to install Kustomize on the local
machine

 

\$ kustomize version --short ; verify that Kustomize installed correctly

 

<img src="./images/media/image7.jpeg" style="width:5in;height:1.75in"
alt="&gt;_ $ kustomize version -- short {kustomize/v4.4.1 11/11/2021 23:36:27 " />

 

-If you don't see an output similar to this that means most likely there
was an issue with an installation or maybe the environment variables
weren't updated in the current terminal session. In that case, it is
recommended to close the current terminal and open new terminal, then
rerun the installation script again.

 

 

**kustomization.yaml File**

 

-Create a directory - **k8s** that contain Kubernetes config YAML files
so that we are going to point Kustomize to this folder

-But Kustomize won't look these files. Instead Kustomize looks for
specific file kustomization.yaml file

-kustomization.yaml file contain a list of all Kubernetes **resources**
that should be managed by Kustomize and all of the customization that we
want to apply for change e.g. **commonLabels**

 

 

-------------kustomization.yaml----------------------

 

\#kubernetes resources to be managed by kustomize

resources:

\- nginx-deployement.yaml

\- nginx-service.yaml

\#Customizations that need to be made

commonLabels:

company: KodeKloud

 

<img src="./images/media/image8.jpeg"
style="width:6.26806in;height:3.46806in"
alt="Kustomization.yaml File k8s Kustomization.yaml nginx-depl.yml # kubernetes resources to be managed by kustomize nginx-service.yml resources : - nginx-deployment.yaml nginx-service.yaml kustomization.yaml #Customizations that need to be made commonLabels: company: Kodekloud " />

 

 

 

\$ kustomize build k8s/ ;point kustomize build command to the K8s
directory to where Kustomize looks the kustomization.yaml file

 

-Kustomize looks for kustomization.yaml file and is going to point two
resources we defined. Then import them to apply all of the
transformations we have defined

-Then Kustomize build command output throws how final configs would look
like in dry-run mode after customizing the commonLabels

 

 

<img src="./images/media/image9.jpeg"
style="width:6.26806in;height:3.37639in"
alt="Kustomize Build kustomize build k&amp;s/ apiVersion: v1 kind: Service metadata: $ kustomize build k8s/ labels: company: Kodekloud name:&quot;nginx-loadbalancer-service service spec: ports : - port: 80 protocol: TCP targetPort: 3000 selector: company: Kodekloud component: nginx type: LoadBalancer .... apiVersion: apps/v1 kind: Deployment metadata: labels: company: Kodekloud name: nginx-deployment spec : replicas: 1 selector: matchLabels: company: Kodekloud component: nginx nginx template: metadata: labels: company: Kodekloud component: nginx spec : containers: - image: nginx name: nginx ........... " />

 

Kustomize looks for a kustomization file which contains

- List of all the Kubernetes manifests kustomize should manage

- All of the customizations that should be applied

 

The *kustomize build* command combines all the manifests and applies the
defined transformations

 

The *kustomize build* command does not apply/deploy the Kubernetes
resources to the cluster

- The output needs to be redirected to the ***kubectl apply*** command

 

 

**Kustomize Output**

 

***Apply Kustomize Configs***

 

After running *kustomize build* command, then we need to deploy it to
the Kubernetes cluster.

 

\$ kustomize build k8s/ \| kubectl apply -f - ; build all the configs
and apply them to the cluster using Linux pipe utility

 

It redirects the output of first command into the input of the second
command, which is the command to the right of the Linux pipe utility.

 

\$ kubectl apply -k k8s/ ; apply the configs natively with the kubectl
tool

 

<img src="./images/media/image10.jpeg"
style="width:6.26806in;height:3.30139in"
alt="Apply Kustomize Configs $ kustomize build k8s/ kubernetes Doesn&#39;t apply/deploy config $ kustomize build k8s/ | kubectl apply -f - service/nginx-loadbalancer-service created deployment. apps/nginx-deployment created k Redirect NGINX Service $ kubectl apply -k k8s/ service/nginx-loadbalancer-service created deployment.apps/nginx-deployment created " />

 

***Delete with Kustomize***

 

\$ kustomize build k8s/ \| kubectl delete -f - ; build all the configs
and delete them from the cluster using Linux pipe utility

 

\$ kubectl delete -k k8s/ ; delete the configs natively with the kubectl
tool

 

<img src="./images/media/image11.jpeg"
style="width:6.26806in;height:3.07569in"
alt="Delete with Kustomize $ kustomize build k8s/ | kubectl delete -f - kubernetes service &quot;nginx-loadbalancer-service&quot; deleted deployment. apps &quot;nginx-deployment&quot; deleted $ kubectl delete -k k8s/ service &quot;nginx-loadbalancer-service&quot; deleted deployment.apps &quot;nginx-deployment&quot; deleted " />

 

 

**Kustomize Api Version & Kind**

 

-Can set the API version and kind properties in a Kustomization.yaml
file

-They are technically optional as Kustomize pick up default values. But
it is recommended that you hard code these values in case there is any
kind of breaking changes in the future

 

-------------kustomization.yaml----------------------

 

apiVersion: kustomize.config.k8s.io/v1beta1

kind: Kustomization

 

\#kubernetes resources to be managed by kustomize

resources:

nginx-deployement.yaml

nginx-service.yaml

\#Customizations that need to be made

commonLabels:

company: KodeKloud

 

**Managing Directories**

 

***Multiple Directories***

 

-Kustomize allows managing Kubernetes manifests that spread across
multiple directories

-As long as all of Kubernetes manifest files may contain in a single
directory, we only just do kubectl apply command

-When the number of YAML files grow, we can organize YAML files by
moving them into sub directories based on application or environment,
then go onto the directories and do the kubectl apply.

-It is bit of a pain to go each one of sub directories and run kubectl
apply whenever every time we make changes on config files. We also have
to configure CI/CD pipeline to do the same. This is where Kustomize come
into play.

 

\$ kubectl apply -f k8s/api

 

\$ kubectl apply -f k8s/db

 

\$ kubectl apply -f k8s/api -f k8s/db ; Create resources in multiple
directories from single command

 

\$ kubectl delete -f k8s/api -f k8s/db ; Delete resources in multiple
directories from single command

 

<img src="./images/media/image12.jpeg"
style="width:6.26806in;height:3.24653in"
alt="Multiple Directories k8s $ kubectl apply -f k8s/api/ api $ kubectl apply -f k8s/db/ api-depl.yaml api-service. yaml db db-depl.yaml db-service. yaml API API DB Depl DB Service Depl Service " />

 

 

 

-can create a kustomization.yaml file in the root directory of our k8s

-list out all of the resources with relative path to import inside
kustomization.yaml file

 

<img src="./images/media/image13.jpeg"
style="width:6.26806in;height:3.36042in"
alt="Multiple Directories k8s Kustomization.yml kustomization.yaml apiVersion: kustomize.config.k8s.io/v1beta1 kind: Kustomization api api-depl.yaml + # kubernetes resources to be managed by kustomize resources : api-service.yaml - api/api-depl.yaml - api/api-service.yaml db - db/db-depl.yaml - db/db-service.yaml db-depl.yaml db-service.yaml " />

 

-We can deploy all resources from just one command without going onto
each one of sub directories

 

\$ kustomize build k8s/ \| kubectl apply -f - Or

 

\$ kubectl apply -k k8s/

 

-Whenever the number of sub directories that use for each one of
application grow, the number of resources we define in
kustomization.yaml seems messy.

 

<img src="./images/media/image14.jpeg"
style="width:6.26806in;height:3.50139in"
alt="Multiple Directories k8s Kustomization.yml kustomization.yaml apiVersion: kustomize.config.k8s.io/v1beta1 kind: Kustomization api # kubernetes resources to be managed by db kustomize resources : cache - api/api-depl.yaml - api/api-service.yaml - db/db-depl.yaml kafka - db/db-service.yaml - cache/redis-depl.yaml - cache/redis-service.yaml - cache/redis-config.yaml - kafka/kafka-depl.yaml - kafka/kafka-service.yaml - kafka/kafka-config.yaml " />

 

-We list all of the manifest files within that directory and import
those files inside that specific kustomization.yaml file that reside in
the sub directory.

-The better way of handling this using Kustomize is we can add in a
separate kustomization.yaml file within each one of the sub directories
where it is going to import the manifest files in the sub directories to
kustomization.yaml file in root directory

-In the root Kustomization.yaml file, we'll provide a path to all of the
sub directories to import manifests files through Kustomization.yaml
file in sub directories.

 

<img src="./images/media/image15.jpeg"
style="width:6.26806in;height:3.65278in"
alt="Multiple Directories k8s k8s/kustomization.yaml kustomization.yaml · resources : - api/ api - db/ - cache/ kustomization.yaml - kafka/ db k8s/db/kustomization.yaml kustomization.yaml resources : cache db-depl.yaml kustomization.yaml - db-service.yaml kafka kustomization.yaml " />

 

***Apply Kustomize Configs***

 

-We can deploy all resources from just a one command without going onto
each one of sub directories

 

\$ kustomize build k8s/ \| kubectl apply -f - Or

 

\$ kubectl apply -k k8s/

 

 

**Common Transformers**

 

-This is to modify or transform Kubernetes configs for all Kubernetes
resources

-Kustomize has several built in transformers such as common transformer

-can even create custom transformers

 

Assume that we want to specifically add a label or specific prefix or
suffix to the name in Kubernetes manifest file. In production
environment, you're going to have significantly more than just two YAML
files. Doing this by hand is not a scalable solution, it's
time-consuming and it's going to lead to a lot of errors.

 

commonLabels- adds a label to all Kubernetes resources

 

namePrefix/Suffix -add a common prefix-suffix to all resource names

 

Namespace - adds a common namespace to all resources

 

commonAnnotations - adds an annotation to all resources

 

***CommonLabel Transformer***

 

-add label to all of Kubernetes resources that are imported by
kustomization.yaml file

 

------- kustomization.yaml file -------------

 

commonLabels:

org: KodeKloud

 

<img src="./images/media/image16.jpeg"
style="width:6.26806in;height:3.62292in"
alt="CommonLabel Transformer db-service.yaml Kustomization.yaml apiVersion: v1 commonLabels: kind: Service org: Kodekloud metadata: labels: org: Kodekloud name: api-service spec : ports: - port: 80 protocol: TCP targetPort: 3000 selector: component: api jorg: Kodekloud type: LoadBalancer " />

 

***Namespace Transformer***

 

***-***apply common namespace to all Kubernetes resources

 

------- kustomization.yaml file -------------

 

namespace: lab

 

 

***Name Prefix/suffix Transformer***

 

***-***apply common name prefix and name suffix to all Kubernetes
resources

 

------- kustomization.yaml file -------------

 

namePrefix: KodeKloud-

 

nameSuffix: -dev

 

<img src="./images/media/image17.jpeg"
style="width:6.26806in;height:3.03889in"
alt="Name Prefix/suffix Transformer db-service.yaml Kustomization.yaml apiVersion: v1 namePrefix: Kodekloud- kind: Service metadata: nameSuffix: - dev name :; Kodekloud-api-service-dev; + spec: ports: - port: 80 protocol: TCP targetPort: 3000 selector: component: api type: LoadBalancer " />

 

 

***CommonAnnotations Transformer***

 

***-***apply common annotation to all Kubernetes resources

 

------- kustomization.yaml file -------------

 

commonAnnotations:

branch: master

 

\* When you apply any transformers to a kustomization.yaml file within a
subdirectory, it's going to only apply transformers to the resources
that import from kustomization.yaml file in the sub directory

\* When you apply any transformers to a kustomization.yaml file within
root directory, , it's going to apply transformers to the resources that
import from kustomization.yaml file in the root directory

 

 

**Image Transformer**

 

-This is to modify an image that a specific deployment or container is
going to use through Kustomize

 

------- kustomization.yaml file -------------

 

images:

\- name: nginx

newName: haproxy

 

name - the name of the specific image that we want to replace in all
Kubernetes configs

newName - the new name of the specific image we replace in all
Kubernetes configs

 

------- kustomization.yaml file -------------

 

<img src="./images/media/image18.jpeg"
style="width:6.26806in;height:3.53194in"
alt="Image Transformer Web-depl.yaml Web-depl.yaml apiVersion: apps/v1 apiVersion: apps/v1 kind: Deployment kind: Deployment metadata: metadata: name: web-deployment name: web-deployment spec : kustomization.yaml spec : replicas: 1 replicas: 1 selector: + images : selector: matchLabels: (-oname: nginx matchLabels: component: web newName: haproxy component: web template: template: metadata: metadata: labels: labels: component: web component: web spec : spec : containers: containers: - name: web - name: web image: &#39;nginx! image: [haproxy " />

 

images:

\- name: nginx

newTag: 2.4

 

newTag - the new tag of the specific image we replace in all Kubernetes
configs

 

Image tag may want to apply as a string with "". As example "2.4"

 

<img src="./images/media/image19.png"
style="width:6.26806in;height:0.69722in" />

 

 

We can combine image name and tag as follows.

 

------- kustomization.yaml file -------------

 

images:

\- name: nginx

newName: haproxy

newTag: 2.4

 

<img src="./images/media/image20.jpeg"
style="width:6.26806in;height:3.42292in"
alt="Image Transformer Web-depl.yaml Web-depl.yaml apiVersion: apps/v1 apiVersion: apps/v1 kind: Deployment kind: Deployment metadata: metadata: name: web-deployment name: web-deployment spec : kustomization.yaml spec : replicas: 1 replicas: 1 selector: selector: images: matchLabels: matchLabels: ( -. name: nginx component: web component: web newName: haproxy template: template: newTag: 2.4 metadata: labels: metadata: component: web labels: spec : component: web containers: spec: - name: web containers: image: haproxy: 2.4 - name: web image: {nginx! " />

 

 

 

**Patches Intro**

 

-Kustomize patches provide another method to modify Kubernetes configs

-Unlike common transformers, patches provide a more "surgical" approach
to targeting one or more specific sections in a Kubernetes resource

-However, kustomize patch is to match a specific object and change the
value on one specific object or a couple of objects

-To create a patch, 3 parameters must be provided:

 

- Operation type: add/remove/replace Ex: Add another container to list
  of containers in a deployment, change a replica count in a deployment
  using replace

- Target: What resource should this patch be applied on. We can mix and
  match on a couple of different properties from below.

 

Kind

Version/Group

Name

Namespace

labelSelector

AnnotationSelector

 

- Value: What is the value that will either be replaced or added
  with(only needed for add/replace operations)

 

------- kustomization.yaml file -------------

 

patches:

\- target:

kind: Deployment

name: api-deployment

 

patch: \|-

\- op: replace

path: /metadata/name

value: web-deployment

\|- this is for what is referred to as an ***inline patch.***

 

Path property to tell Kustomize what is the specific property we want to
replace or update, and how do we get to it in the YAML tree.

 

<img src="./images/media/image21.jpeg"
style="width:6.26806in;height:3.40556in"
alt="Patches api-depl.yaml api-depl.yaml apiVersion: apps/v1 apiVersion: apps/v1 kind: Deployment kind: Deployment metadata: name: api-deployment kustomization.yaml metadata: name: web-deployment spec: spec : replicas: 1 patches : replicas: 1 selector: - target: selector: matchLabels: kind: Deployment matchLabels: component: api name: api-deployment component: api template: template: metadata: patch: | - metadata: labels: op: replace labels: component: api path: /metadata/name value: web-deployment component: api spec : spec : containers: containers : - name: nginx - name: nginx image: nginx image: nginx " />

 

<img src="./images/media/image22.jpeg"
style="width:6.26806in;height:3.58819in"
alt="Patches api-depl.yaml Kustomization api-depl.yaml apiVersion: apps/v1 patches : apiVersion: apps/v1 kind: Deployment - target: kind: Deployment metadata: kind: Deployment metadata: name: api-deployment name: api-deployment name: api-deployment spec: patch: | - spec : į replicas: 1 - op: replace replicas: 5 selector: path: /spec/replicas Selector: matchLabels: value: 5 matchLabels: component: api component: api template: template: metadata: A metadata: labels: labels: component: api component: api spec : spec : containers: containers: - name: nginx - name: nginx image: nginx image: nginx " />

 

 

***JSON 6902 vs Strategic Merge Patch***

 

In kustomize, there's actually two different ways to define a patch.

 

***JSON 6902 Patch*** has two things that you have to provide.

target - The Kubernetes object that we want to patch

patch - What operation, what property we want to update and what is the
new value

<https://datatracker.ietf.org/doc/html/rfc6902> shows step by step
through all the different features and functionalities it has.

 

------- kustomization.yaml file -------------

 

patches:

\- target:

kind: Deployment

name: api-deployment

 

patch: \|-

\- op: replace

path: /spec/replicas

value: 5

 

 

***Strategic Merge Patch*** has standard Kubernetes config where we copy
the original deployment file and just paste it, then delete all the
config properties that don't want to change.

 

One of the perks of using a strategic merge patch is that it's using
regular Kubernetes configs. Then it's going to take this new config and
merge it with the old config, it's going to figure out exactly what's
changed and it's going to update those properties.

 

*Strategic Merge Patch* has two things that you have to provide.

 

1.  What Kubernetes object that you want to update i.e name of the
    deployment

2.  The specific properties that you want to change

 

 

------- kustomization.yaml file -------------

 

patches:

\- patch: \|-

apiVersion: apps/v1

kind: Deployment

metadata:

name: api-deployment

spec:

replicas: 5

 

<img src="./images/media/image23.jpeg"
style="width:6.26806in;height:3.08333in"
alt="A screenshot of a computer AI-generated content may be incorrect." />

 

 

**Different types of patches**

 

There is two different ways you can define a patch.

 

1.  Inline - define the patch within the kustomization.yaml file itself

 

2.  Sperate file - define the patch by using separate file where we
    provide target and a path to a YAML file that contain all patches

 

If you have a lot of patches, it may start to clutter kustomization.yaml
file up, and then you can move it to a separate file and then use the
separate file method.

 

 

***JSON 6902 patch inline vs Separate File***

 

------- kustomization.yaml file -------------

 

patches:

\- path: replica-patch.yaml

target:

kind: Deployment

name: api-deployment

 

----------replica-patch.yaml-------------

 

\- op: replace

path: /spec/replicas

value: 5

 

 

<img src="./images/media/image24.jpeg"
style="width:6.26806in;height:3.15764in"
alt="Json 6902 patch Inline vs Separate File Inline Separate File Kustomization Kustomization replica-patch.yaml patches : - target: kind: Deployment patches : - op: replace name: api-deployment path: replica-patch. yaml path: /spec/replicas patch: | - target: kind: Deployment value: 5 - op: replace path: /spec/replicas name: nginx-deployment value: 5 " />

 

 

***Strategic merge patch inline vs Separate File***

 

------- kustomization.yaml file -------------

 

patches:

\- replica-patch.yaml

 

 

----------replica-patch.yaml-------------

 

apiVersion: apps/v1

kind: Deployment

metadata:

name: api-deployment

spec:

replicas: 5

 

<img src="./images/media/image25.jpeg"
style="width:6.26806in;height:3.08819in"
alt="Strategic merge patch Inline vs Separate File Inline Separate File Kustomization Kustomization replica-patch.yaml patches : - patch: | - apiVersion: apps/v1 patches : - replica-patch.yaml apiVersion: apps/v1 kind: Deployment kind: Deployment metadata: metadata: name: api-deployment name: api-deployment spec : spec: replicas: 5 replicas: 5 " />

 

 

<img src="./images/media/image26.png"
style="width:6.26806in;height:5.60903in"
alt="A screenshot of a computer program AI-generated content may be incorrect." />

 

<img src="./images/media/image27.png"
style="width:5.95in;height:2.50833in"
alt="A screen shot of a computer AI-generated content may be incorrect." />

 

**Patches Dictionary**

 

***Replace Dictionary JSON 6902***

 

Changing a value of key in dictionary where specify the key in path
section using JSON 6902.

 

------- kustomization.yaml file -------------

 

patches:

\- target:

kind: Deployment

name: api-deployment

 

patch: \|-

\- op: replace

path: /spec/template/metadata/labels/**component \<--- key**

value: web

 

 

<img src="./images/media/image28.jpeg"
style="width:6.26806in;height:3.61458in"
alt="Replace Dictionary Json6902 api-depl.yaml kustomization api-depl.yaml apiVersion: apps/v1 patches: kind: Deployment - target: apiVersion: apps/v1 metadata: kind: Deployment kind: Deployment metadata: name: api-deployment name: api- name: api-deployment spec : deployment replicas: 1 patch: | - spec : selector: - op: replace replicas: 1 selector: matchLabels: path : / spec/template/metadata/ matchLabels: component: api template: labels/component component: api value: web template: metadata: metadata: labels: labels: component: api component: web spec : spec: containers: containers: - name: nginx - image: nginx image: nginx name: nginx " />

 

***Replace Dictionary Strategic merge patch***

 

Copy and paste the deployment configuration, delete all the properties
that will not be changing. Only add the property that you want to change
in to sperate YAML file. Then Kustomize takes this config and merge it
with original deployment config.

 

<img src="./images/media/image29.jpeg"
style="width:6.26806in;height:3.46181in"
alt="Replace Dictionary Strategic Merge Patch api-depl.yaml kustomization label-patch.yaml apiVersion: apps/v1 patches : apiVersion: apps/v1 kind: Deployment - label-patch.yaml kind: Deployment metadata: metadata: name: api-deployment name: api-deployment spec : spec : replicas: 1 template: selector: metadata: matchLabels: labels: component: api component: web template: metadata: labels: component: api spec : containers: - name: nginx image: nginx " />

 

 

***Add Dictionary JSON 6902***

 

Adding a key in dictionary using JSON 6902.

 

------- kustomization.yaml file -------------

 

patches:

\- target:

kind: Deployment

name: api-deployment

 

patch: \|-

\- op: add

path: /spec/template/metadata/labels/**org \<--- key**

value: KodeKloud

 

 

<img src="./images/media/image30.jpeg"
style="width:6.26806in;height:3.43542in"
alt="Add Dictionary Json6902 api-depl.yaml kustomization api-depl.yaml apiVersion: apps/v1 patches : kind: Deployment - target: apiVersion: apps/v1 metadata: kind: Deployment kind: Deployment name: api-deployment name: api- metadata: spec : deployment name: api-deployment replicas: 1 patch: | - spec: selector: - op: add replicas: 1 matchLabels: path: selector: component: api / spec/template/metadata/ matchLabels: template: labels/org component: api metadata: value: Kodekloud template: labels: metadata: component: api labels: spec : component: api containers: org: Kodekloud - name: nginx speč: image: nginx containers: - image: nginx name: nginx " />

 

 

***Add Dictionary Strategic merge patch***

 

<img src="./images/media/image31.jpeg"
style="width:6.26806in;height:3.43125in"
alt="Add Dictionary Strategic Merge Patch api-depl.yaml kustomization label-patch.yaml apiVersion: apps/v1 kind: Deployment patches : - label-patch.yaml apiVersion: apps/v1 kind: Deployment metadata: metadata: name: api-deployment name: api-deployment spec : replicas: 1 spec : selector: template: matchLabels: metadata: component: api labels: template: org: kodekloud metadata: labels: component: api spec: containers: - name: nginx image: nginx " />

 

 

***Remove Dictionary JSON 6902***

 

Only specify the key that you need to remove from the path in
kustomization.yaml file where no value specify as the value is removed.

 

------- kustomization.yaml file -------------

 

patches:

\- target:

kind: Deployment

name: api-deployment

 

patch: \|-

\- op: remove

path: /spec/template/metadata/labels/**org \<--- key**

 

 

<img src="./images/media/image32.jpeg"
style="width:6.26806in;height:3.51319in"
alt="Remove Dictionary Json6902 api-depl.yaml kustomization api-depl.yaml apiVersion: apps/v1 patches: kind: Deployment - target: apiVersion: apps/v1 metadata: kind: Deployment kind: Deployment metadata: name: api-deployment name: api- deployment name: api-deployment spec : spec: replicas: 1 patch: replicas: 1 selector: - op : ¡ remove selector: matchLabels: path: component: api / spec/template/metadata/1 matchLabels: component: api template: abels/org i template: metadata: metadata: labels: labels: org: Kodekloud component: api component: api spec : spēč: containers: containers : - image: nginx - name: nginx name: nginx image: nginx " />

 

***Remove Dictionary Strategic merge patch***

 

Specify the value of key is null to delete the key in sperate YAML file.

 

<img src="./images/media/image33.jpeg"
style="width:6.26806in;height:3.35139in"
alt="Remove Dictionary Strategic Merge Patch api-depl.yaml kustomization label-patch.yaml apiVersion: apps/v1 patches : kind: Deployment apiVersion: apps/v1 - label-patch.yaml kind: Deployment metadata: metadata: name: api-deployment name: api-deployment spec: replicas: 1 spec: template: selector: matchLabels: metadata: ( labels: component : api template: org: null metadata: labels: i org: Kodekloud component: api spec : containers: - name: nginx image: nginx " />

 

 

**Patches list**

 

Let's take a look at how we can perform remove, replace, and add keys
operations on a list.

 

***Replace List JSON 6902***

 

In the path section in kustomization file, we specify the index is going
to represent which item in that list you want to update. As example, we
take containers section in spec to update.

 

0 index number is 1st item in the list.

 

------- kustomization.yaml file -------------

 

patches:

\- target:

kind: Deployment

name: api-deployment

 

patch: \|-

\- op: replace

path: /spec/template/spec/containers/**0 \<--- Index number**

value:

name: haproxy

image: haproxy

 

OR

 

patches:

  - target:

      kind: Deployment

      name: api-deployment

    patch: \|-

      - op: replace

        path: /spec/template/spec/containers/0/image

        value: caddy

 

 

 

<img src="./images/media/image34.jpeg"
style="width:6.26806in;height:3.50764in"
alt="Replace List Json6902 api-depl.yaml kustomization api-depl.yaml apiVersion: apps/v1 patches: kind: Deployment - target: apiVersion: apps/v1 kind: Deployment kind: Deployment metadata: metadata: name: api-deployment name: api- deployment name: api-deployment spec : patch: | - spec : replicas: 1 replicas: 1 selector: - op: replace selector: matchLabels: path: component: api / spec/template/spec/cont matchLabels: component: api template: ainers/0 template: metadata: List (value: labels: name: haproxy metadata: image: haproxy labels: component api component: api spec : spec : containers: (containers: · name: nginx - image: haproxy image: nginx name: haproxy " />

 

 

***Replace List Strategic merge patch***

 

Only add the property that you want to change in to sperate YAML file.
Then Kustomize takes this config and merge it with original deployment
config.

<img src="./images/media/image35.jpeg"
style="width:6.26806in;height:3.46528in"
alt="Replace List Strategic Merge Patch api-depl.yaml kustomization label-patch.yaml apiVersion: apps/v1 kind: Deployment patches : apiVersion: apps/v1 - label-patch.yam1 kind: Deployment metadata: name: api-deployment metadata: name: api-deployment spec : replicas: 1 spec: template: selector: matchLabels: spec : component: api containers: template: - name: nginx image: haproxy metadata: labels: component: api spec : containers: - name: nginx image: nginx " />

 

 

***Add List JSON 6902***

 

In the path section in kustomization file, we specify - mark to append
to end of list or index which item in that list you want to update.

 

------- kustomization.yaml file -------------

 

patches:

\- target:

kind: Deployment

name: api-deployment

 

patch: \|-

\- op: replace

path: /spec/template/spec/containers/**- \<--- Append end of list**

value:

name: haproxy

image: haproxy

 

 

<img src="./images/media/image36.jpeg"
style="width:6.26806in;height:3.68056in"
alt="Add List Json6902 api-depl.yaml kustomization api-depl.yaml apiVersion: apps/v1 patches : apiVersion: apps/v1 kind: Deployment - target: kind: Deployment metadata: kind: Deployment metadata: name: api-deployment name: api- name: api-deployment spec : deployment pec: replicas: 1 patch :_ 1- Append to replicas: 1 selector: - lop: add end of list selector: matchLabels: path: matchLabels: component: api / spec/templ-+ spec/cont component: api template: ainers/- template: metadata: Value: metadata: labels: name: haproxy labels: component: api image: haproxy component: api spec: spec : containers: containers: - name: nginx - image: nginx image: nginx name :_ nginx - image: haproxy name: haproxy " />

 

 

***Add List Strategic Merge Patch***

 

<img src="./images/media/image37.jpeg"
style="width:6.26806in;height:3.64167in"
alt="ROUE Add List Strategic Merge Patch api-depl.yaml kustomization label-patch.yaml apiVersion: apps/v1 patches : apiVersion: apps/v1 kind: Deployment - label-patch.yaml kind: Deployment metadata: metadata: name: api-deployment name: api-deployment spec: replicas: 1 spec: template: selector: matchLabels: spec : component: api containers: template: - name: haproxy metadata: image: haproxy labels: component: api spec: containers: - name: web image: nginx " />

 

 

***Delete List JSON 6902***

 

In the path section in kustomization file, we specify index number which
item in that list you want to delete where we don't need to provide
value as this is remove operation.

 

patches:

\- target:

kind: Deployment

name: api-deployment

 

patch: \|-

\- op: remove

path: /spec/template/spec/containers/**1 \<--- Index of container to
delete**

 

<img src="./images/media/image38.jpeg"
style="width:6.26806in;height:3.52847in"
alt="Delete List Json6902 api-depl.yaml kustomization api-depl.yaml apiVersion: apps/v1 kind: Deployment patches: apiVersion: apps/v1 metadata: - target: kind: Deployment name: api-deployment kind: Deployment metadata: spec : name: api- name: api-deployment replicas: 1 deployment Index of spec: selector: patch: 1- container replicas: 1 matchLabels: - top: remove to delete selector: component: api path: matchLabels: template: / spec/template/ pec/cont component: api metadata: ainers/1 template: labels: metadata: component: api labels: spec : component: api containers: spec : - name: web containers: _image :_ nginx - image: nginx - name: database name: web image: mongo " />

 

 

***Delete List Strategic Merge Patch***

 

Specify \$ symbol patch, and then delete to delete the item in the list
in sperate YAML file where we use the delete directive. We're listing
out all of the Kubernetes configs that we want to change.

 

----------replica-patch.yaml-------------

 

apiVersion: apps/v1

kind: Deployment

metadata:

name: api-deployment

spec:

template:

spec:

containers:

\- \$patch: delete

name: database

 

 

<img src="./images/media/image39.jpeg"
style="width:6.26806in;height:3.56875in"
alt="Delete List Strategic Merge Patch api-depl.yaml kustomization label-patch.yaml apiVersion: apps/v1 patches : apiVersion: apps/v1 kind: Deployment - label-patch.yaml kind: Deployment metadata: metadata: name: api-deployment name: api-deployment spec : spec : replicas: 1 template: selector: Delete Directive spec: matchLabels: component: api specifies which containers: container to delete $patch: delete template: name: database metadata: labels: component: api spec : containers: - name: web image: nginx - name: database ! image: mongo " />

 

 

**Overlays**

 

Kustomize was created to allow us to import base Kubernetes config and
customize it on a per-environment basis. We may want to tweak certain
properties in development, staging and production environments. To
accomplish this, we use Kustomize with overlays.

 

A Kustomized project can break into following folder structure.

 

base - This contains all of default or share Kubernetes configuration
across all environments

 

overlays - Specify all of the environment specific configs so that we
can import the base config and modify for the specific environment as
desired

 

<img src="./images/media/image40.jpeg"
style="width:6.26806in;height:3.325in"
alt="Overlays K8s/ Share or default configs across all k8s/ environments - base/ Env kustomization.yaml nginx-depl.yaml service.yaml redis-depl.yaml overlays/ dev/ dev stg prod kustomization.yaml config-map.yaml stg/ kustomization.yaml config-map.yaml Environment specific prod/ configurations that add E kustomization.yaml or modify base configs config-map.yaml " />

 

In same way, you can accomplish by providing patches for each
environments.

 

Example:

 

We get same folder structure where default Kubernetes config files are
imported to Kustimization.yaml file in base folder.

 

In overlays directory, you can see ***bases*** property in
Kustomization.yaml file in each environment folder. We provide relative
path to go up to base directory and import from Kustimization.yaml file
in each environment folder.

 

Once this is defined, Kustomize will look for the kustomization.yaml
file in the base directory to know all the resources it should be
importing, then provide a patch.

 

------------dev/kustomization---------------------

 

bases:

\- ../../base

 

patch: \|-

\- op: replace

path: /spec/replicas

value: 2

 

<img src="./images/media/image41.jpeg"
style="width:6.26806in;height:3.07569in"
alt="Overlays Relative path to kustomization. yaml file in base folder K8s/ Base/kustomization dev/kustomization k8s / resources : bases: base/ - nginx-depl.yaml - .. / .. /base kustomization.yaml - service yaml nginx-depl. yaml - redis-depl.yaml patch: | - service.yaml - op: replace redis-depl.yaml path: /spec/replicas overlays/ Base/nginx-depl.yaml value: 2 dev/ kustomization.yaml + config-map.yaml apiVersion: apps/v1 kind: Deployment stg/ prod/kustomization kustomization.yaml metadata: LI name: nginx-deployment bases: prod/ config-map.yaml spec: .. / .. /base E replicas: 1 kustomization.yaml patch: | - config-map.yaml - op: replace path: /spec/replicas value: 3 " />

 

 

In one of environment folder in overlay directory, you can reside new
configs that doesn't exist in the base directory.

 

In kustomization.yaml file is, under the resources section, we import it
like a regular file. Thus you can add new resources inside your overlay
folder.

 

<img src="./images/media/image42.jpeg"
style="width:6.26806in;height:4.19653in"
alt="Overlays K8s/ k8s/ base/ kustomization.yaml prod/kustomization nginx-depl.yaml service.yaml bases : redis-depl.yaml - .. / .. /base overlays/ dev/ i resources : kustomization.yaml - grafana-depl.yaml config-map.yaml volume.yaml stg/ patch: kustomization.yaml - op: replace config-map.yam1 path: /spec/replicas prod/ value: 2 kustomization.yaml _config-map. yam1 grafana-depl.yaml " />

 

The base folder can be broken out into extra sub directories based on
features.

 

<img src="./images/media/image43.jpeg"
style="width:4.99167in;height:5.85in"
alt="k8s/ base/ kustomization.yaml db/ E db-depl.yaml db-service.yaml kustomization.vaml api/ E api-depl.yaml api-service.yaml kustomization.yaml overlays/ - dev/ kustomization.yaml db/ db-patch.yaml kustomization. yaml api/ api-patch.yaml kustomization.yaml prod/ kustomization.yaml db/ db-patch.yaml kustomization.yaml api/ 」 api-patch.yaml kustomization.yaml " />

 

 

**Components**

 

Components provides the ability to define a reusable piece of
configuration logic(resource + patches) that can be imported to sub set
of overlays.

 

Components are useful in situations where applications support multiple
optional features that need to be enabled only in a subset of overlays
not for all overlays. We could do that pretty easily by defining them in
each one of the overlays. But then, now we must be copying and pasting
all of the configs in required subset of overlays. If you make changes
in one of the folders, you'd have to remember to make changes in the
other one. And let's say we over time decide to add a few more
variations, you would have to remember to copy it into all of those
overlays. However, that's not a scalable solution.

 

It's all of the resources for a specific feature, all of the patches,
all of the config maps, secrets, and any other Kubernetes related
configs associated with the feature.

 

Let's take scenario that we need caching feature(enable Redis database)
enabled in Premium and Self-hosted overlays except dev, further need
external DB service enabled(Postgres database) on only in dev and
Premium overlays.

 

<img src="./images/media/image44.jpeg"
style="width:6.26806in;height:2.57431in"
alt="Components Caching · Premium · Self Hosted base External DB · Dev Premium Self · Premium dev hosted caching caching " />

 

We create extra folder called **Components** and store all of components
in that folder. We have all of Kubernetes configs that associated with
caching and external database in each subfolder in components directory
like caching and DB. That way we can import them into subset of overlays
as we want.

 

<img src="./images/media/image45.jpeg"
style="width:6.26806in;height:2.74028in"
alt="Components 1 base Components dev Premium Self hosted .-.- caching db " />

 

 

1.  Create kustomization.yaml for required feature in Components
    directory

 

----------Components/db/kustomization.yaml-----------------

 

apiVersion: kustomize.config.k8s.io/v1alpha1 \<\< apiVersion for
Component

kind: Compenent

 

resources: \<\< import Kubernetes config

\- postgres-depl.yaml

 

secretGenerator:

\- name: postgres-credential

literals:

\- password=postgres123

patches:

\- deployemnt-patch.yaml

 

<img src="./images/media/image46.jpeg"
style="width:6.26806in;height:3.22778in"
alt="Components k8s/ Components/db/kustomization.yaml base/ kustomization.yaml apiVersion: api-depl.yaml kustomize.config.k8s.io/v1alpha1 components/ kind: Component caching/ E - kustomization.yaml deployment-patch. yaml resources : redis-depl.yaml - postgres-depl.yam1 db/ kustomization.yaml secretGenerator: deployment-patch. yaml - name: postgres-cred postgres-depl.yaml literals: overlays/ - password=postgres123 dev/ __ kustomization.yaml - premium! patches : - deployment-patch.yaml L kustomization.yaml standalone/ __ kustomization.yaml " />

 

<img src="./images/media/image47.jpeg"
style="width:6.26806in;height:3.02778in"
alt="k8s/ base/ kustomization.yaml Deployment-patch api-depl.yaml apiVersion: apps/v1 components/ kind: Deployment caching/ - customization. yam1 metadata: deployment-patch.yaml name: api-deployment redis-depl.yaml spec : db/ template: kustomization.yaml spec : deployment-patch.yaml containers: postgres-depl.yaml - name: api overlays/ env: dev/ - name: DB_PASSWORD - kustomization.yaml - premium/ valueFrom: secretKeyRef: L kustomization.yaml name: postgres-cred standalone/ key: password L kustomization.yaml " />

 

2.  Create kustomization.yaml for required environment in overlays
    directory to import Kubernetes config

 

----------Overlay/premium/kustomization.yaml-----------------

 

bases:

\- ../../base

components:

\- ../../components/db

 

 

apiVersion: kustomize.config.k8s.io/v1alpha1

kind: Component

resources:

  - keycloak-depl.yaml

  - keycloak-service.yaml

patches:

  - path: api-patch.yaml

 

 

 

<img src="./images/media/image48.jpeg"
style="width:6.26806in;height:3.21736in"
alt="Components k8s/ Overlay/premium/kustomization base/ kustomization.yaml bases: api-depl.yaml components/ - .. / .. /base caching/ kustomization.yaml components : deployment-patch. yaml - .. / .. / components/db redis-depl.yaml db/+ ~ kustomization. yaml - deployment-patch.yam1 postgres-depl.yaml overlays/ dev/ kustomization.yaml premium/ L kustomization.yaml standalone/ L kustomization.yaml " />

 

 

🚀 End-to-End DevSecOps Pipeline with Trivy, DefectDojo, GitLab CI/CD,
Helm & Argo CD

🔐 Security isn’t just a phase — it’s a culture.

I recently implemented a DevSecOps pipeline that:

Scans with Trivy

Tracks findings in DefectDojo

Deploys via GitLab CI/CD, Helm, and Argo CD

Here’s how you can build it too 👇

⚡ Why This Setup Works

Trivy → Fast, accurate vulnerability scanner for containers,
filesystems, and code.

DefectDojo → Centralized vulnerability management & reporting.

GitLab CI/CD → Automates scanning, validation, and release steps.

Helm → Simplifies Kubernetes deployments.

Argo CD → GitOps-based continuous delivery for Kubernetes.

Benefits:

✅ Security gates before deployment

✅ Centralized vulnerability tracking

✅ Automated GitOps deployments to Kubernetes

 

🔍 The Flow

1️⃣ Developer commits code → triggers GitLab CI/CD.

2️⃣ Trivy scans the container image for vulnerabilities.

3️⃣ Results are sent to DefectDojo via API for tracking.

4️⃣ If scan passes policy, Helm charts are updated in the GitOps repo.

5️⃣ Argo CD detects changes and syncs the application in Kubernetes.

 

🔗 Key Integration Points

🛡 Trivy + GitLab CI/CD

.gitlab-ci.yml

 

trivy-scan:

image: aquasec/trivy:latest

script:

\- trivy image --exit-code 0 --format json -o trivy-report.json
\$CI_REGISTRY_IMAGE

\- curl -X POST "\$DOJO_URL/api/v2/import-scan/" \\

-H "Authorization: Token \$DOJO_API_KEY" \\

-F "scan_type=Trivy Scan" \\

-F "file=@trivy-report.json" \\

-F "product_id=\$DOJO_PRODUCT_ID" \\

-F "engagement_id=\$DOJO_ENGAGEMENT_ID" \\

-F "active=true" \\

-F "verified=true"

Notes:

DOJO_PRODUCT_ID & DOJO_ENGAGEMENT_ID come from DefectDojo.

Trivy JSON output matches DefectDojo’s Trivy Scan type.

Store secrets in GitLab’s CI/CD variables.

⚓ Helm + Argo CD

Helm chart defines your app.

Argo CD automatically syncs from the GitOps repo to Kubernetes.

 

🏆 Benefits in Production

✅ No manual security checks — scans run on every commit.

✅ Vulnerabilities tracked in DefectDojo for audit and follow-up.

✅ GitOps guarantees what’s in Git is what’s running in Kubernetes.

 

💡 Takeaway: Security should be baked into your delivery process, not
bolted on at the end.

With Trivy + DefectDojo + GitLab + Helm + Argo CD, you can:

Shift security left

Keep a single source of truth for deployments

Ship faster, safer

 

💬 Have you integrated vulnerability scanning into your GitOps pipeline?
I’d love to hear your approach!

\#DevSecOps \#Trivy \#DefectDojo \#GitLabCI \#Helm \#ArgoCD \#Kubernetes
\#CloudSecurity \#CICD \#ContainerSecurity \#OpenSourceSecurity

 

<img src="./images/media/image49.jpeg" style="width:5in;height:3.3in"
alt="From Code to Cluster: Integrating Trivy, DefectDojo, GitLab, Helm, and Argo CD for Continuous Security HELM &lt;/&gt; GitLab CI/CD DefectDojo eveloper Helm Argo CD Run Upload Deploy Sync Trivy Scan Results via Helm Changes " />
