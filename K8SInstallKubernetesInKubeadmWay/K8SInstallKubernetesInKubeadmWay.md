**Summary : Install Kubernetes in Kubeadm way**

 

1.  Provision VMs for controlplane and worker nodes

 

2.  Setup forwarding rules
    <https://kubernetes.io/docs/setup/production-environment/container-runtimes/>

 

 

1.  Enable IPv4 packet forwarding on all nodes

 

\# sysctl params required by setup, params persist across reboots

cat \<\<EOF \| sudo tee /etc/sysctl.d/k8s.conf

net.ipv4.ip_forward = 1

EOF

 

\# Apply sysctl params without reboot

sudo sysctl --system

 

Verify that net.ipv4.ip_forward is set to 1 with:

 

sysctl net.ipv4.ip_forward

 

2.  Load br_netfilter kernel module on all nodes

 

This module is required for Flannel's VXLAN backend to work properly, as
it enables iptables to see bridged traffic. Without this module, network
connectivity between pods might not work correctly.

 

Here's how to fix this:

 

Load the br_netfilter module:

Run this command on your controlplane node:

 

bash

sudo modprobe br_netfilter

 

Make it persistent across reboots:

 

bash

echo 'br_netfilter' \| sudo tee /etc/modules-load.d/k8s.conf

 

Verify the module is loaded:

 

bash

lsmod \| grep br_netfilter

Check the sysctl settings:

 

These should be set to 1:

 

bash

cat /proc/sys/net/bridge/bridge-nf-call-iptables

cat /proc/sys/net/bridge/bridge-nf-call-ip6tables

 

If they're not set to 1, you can set them with:

 

bash

sudo sysctl -w net.bridge.bridge-nf-call-iptables=1

sudo sysctl -w net.bridge.bridge-nf-call-ip6tables=1

 

Make the sysctl settings persistent:

Add these lines to /etc/sysctl.conf or create a new file in
/etc/sysctl.d/:

 

text

net.bridge.bridge-nf-call-iptables = 1

net.bridge.bridge-nf-call-ip6tables = 1

 

 

Use the following to add configs

 

cat \<\<EOF \| sudo tee /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-ip6tables = 1

net.bridge.bridge-nf-call-iptables = 1

EOF

 

 

 

3.  Installing kubeadm, kubelet and kubectl on all nodes
    <https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/>

 

controlplane ~ ✖ cat /etc/\*-release ; check the Linux distribution to
install kubeadm, kubelet and kubectl

 

1.  Update the apt package index and install packages needed to use the
    Kubernetes apt repository:

 

sudo apt-get update

 

2.  Download the public signing key for the Kubernetes package
    repositories. The same signing key is used for all repositories so
    you can disregard the version in the URL:

 

curl -fsSL <https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key> \|
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

 

3.  Add the appropriate Kubernetes apt repository.

 

echo 'deb \[signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg\]
<https://pkgs.k8s.io/core:/stable:/v1.33/deb/> /' \| sudo tee
/etc/apt/sources.list.d/kubernetes.list

 

4.  Update the apt package index, install kubelet, kubeadm and kubectl,
    and pin their version:

 

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

 

 

If you want to install specific version of kubelet and kubeadm

 

sudo apt-get install -y kubelet=1.33.0-00 kubeadm=1.33.0-00
kubectl=1.33.0-00

 

 

4.  Creating a cluster with kubeadm
    <https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/>

 

1.  Install a container runtime on all nodes
    <https://kubernetes.io/docs/setup/production-environment/container-runtimes/>,
    <https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd>

sudo apt update

sudo apt install -y containerd

2.  Configuring cgroup driver: On Linux, control groups are used to
    constrain resources that are allocated to processes in containers.
    There are two cgroup drivers available such as **cgroupfs**,
    **systemd**. Both the kubelet and container runtime must be
    configured to use same cgroup driver.

 

 

(i). To set the cgroup driver on Kubelet, create a kubelet configuration
file. In version 1.22 or later, if user does not set the cgroup driver
field under kubelet configuration, kubeadm will default to systemd
<https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/>

 

(ii). Make sure that container runtime is set to utilize the cgroup
driver so that create config file for containerd which store in
/etc/containerd/config.toml. By default, it is not going to be using
systemd.
<https://kubernetes.io/docs/setup/production-environment/container-runtimes/>,
<https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cgroupfs-cgroup-driver>

\>sudo mkdir -p /etc/containerd ; create directory /etc/containerd on
all nodes

Now the containerd process allows to generate default configuration. If
you don't specify config file, it uses default configurations. Take a
look the configuration by running 'containerd config default'. With the
following command, set SystemdCgroup to true and send the config to
/etc/containerd/config.toml

\> containerd config default ; check containerd default configuration

\> containerd config default \| sed 's/SystemdCgroup =
false/SystemdCgroup = true/' \| sudo tee /etc/containerd/config.toml ;
set SystemdCgroup to true and send the config to
/etc/containerd/config.toml

\> cat /etc/containerd/config.toml \| grep -i SystemdCgroup -B 50 ;
printing out whole bunch of lines before the grep keyword to make sure
it is right section
\[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options\]

\> sudo systemctl restart containerd ; restart containerd

 

 

 

3.  Initializing control-plane node
    <https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/>

 

\> sudo kubeadm init --apiserver-advertise-address 192.168.56.11
--pod-network-cidr "10.244.0.0/16" --upload-certs

 

While --apiserver-advertise-address can be used to set the advertised
address for this particular control-plane node's API server,
--control-plane-endpoint can be used to set the shared endpoint for all
control-plane nodes.

 

--control-plane-endpoint allows both IP addresses and DNS names that can
map to IP addresses. Please contact your network administrator to
evaluate possible solutions with respect to such mapping.

 

--upload-certs this make sure that we upload all of the certificates to
a secret so all of the other nodes will have access to the certificates.

 

(i).To start using your cluster, copy kubeconfig file to the default
home directory. Then you need to run the following as a regular user:

 

mkdir -p \$HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf \$HOME/.kube/config

sudo chown \$(id -u):\$(id -g) \$HOME/.kube/config

 

Alternatively, if you are the root user, you can run:

 

export KUBECONFIG=/etc/kubernetes/admin.conf

 

Check node status. Note that node status is currently in a not ready
state as we haven't set up the network plugin up and running. The node
will be moved to ready status

vagrant@controlplane:~\$ kubectl get nodes

NAME STATUS ROLES AGE VERSION

controlplane NotReady control-plane 56m v1.33.1

 

(ii). Copy the kubeadm join command that show when initialize
controlplane node

 

kubeadm join 192.168.56.11:6443 --token 5xf60f.9cs2ig2lm2snvx7c \\

--discovery-token-ca-cert-hash
sha256:5d071e1fc348a1cc9ccdef728fae2b387e99e2649de3ac100fb6fac0ee7cc9de

 

 

(iii). Deploy a pod network for your cluster which is netowrk add on or
CNI plugin
<https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy>

We choose flannel as CNI plugin
<https://github.com/flannel-io/flannel#deploying-flannel-manually>

\> wget
<https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml>

 

vagrant@controlplane:~\$ kubectl apply -f kube-flannel.yml

namespace/kube-flannel created

serviceaccount/flannel created

clusterrole.rbac.authorization.k8s.io/flannel created

clusterrolebinding.rbac.authorization.k8s.io/flannel created

configmap/kube-flannel-cfg created

daemonset.apps/kube-flannel-ds created

 

5.  Join any worker nodes to the cluster by running following command on
    worker nodes

sudo kubeadm join 192.168.56.11:6443 --token o4cfas.3534vrh4aiup3qo5 \\

--discovery-token-ca-cert-hash
sha256:02a2f75b4f2cdfcd8cdf85a46f1ec6188de8943c9dc287b525b6d0f05372825

 

 

 

controlplane ~ ➜ kubectl get nodes

NAME STATUS ROLES AGE VERSION

controlplane Ready control-plane 12m v1.32.0

node01 Ready \<none\> 10m v1.32.0

 

 

 

 

PS
C:\Users\nibha\certified-kubernetes-administrator-course\kubeadm-clusters\virtualbox\>
vagrant status

Current machine states:

 

controlplane not created (virtualbox)

node01 not created (virtualbox)

node02 not created (virtualbox)

 

This environment represents multiple VMs. The VMs are all listed

above with their current state. For more information about a specific

VM, run \`vagrant status NAME\`.

 

PS
C:\Users\nibha\certified-kubernetes-administrator-course\kubeadm-clusters\virtualbox\>
vagrant up

Bringing machine 'controlplane' up with 'virtualbox' provider...

Bringing machine 'node01' up with 'virtualbox' provider...

Bringing machine 'node02' up with 'virtualbox' provider...

==\> controlplane: Box 'ubuntu/jammy64' could not be found. Attempting
to find and install...

controlplane: Box Provider: virtualbox

controlplane: Box Version: \>= 0

==\> controlplane: Loading metadata for box 'ubuntu/jammy64'

controlplane: URL:
<https://vagrantcloud.com/api/v2/vagrant/ubuntu/jammy64>

==\> controlplane: Adding box 'ubuntu/jammy64' (v20241002.0.0) for
provider: virtualbox

controlplane: Downloading:
<https://vagrantcloud.com/ubuntu/boxes/jammy64/versions/20241002.0.0/providers/virtualbox/unknown/vagrant.box>

controlplane:

==\> controlplane: Successfully added box 'ubuntu/jammy64'
(v20241002.0.0) for 'virtualbox'!

==\> controlplane: Importing base box 'ubuntu/jammy64'...

==\> controlplane: Matching MAC address for NAT networking...

==\> controlplane: Setting the name of the VM: controlplane

Vagrant is currently configured to create VirtualBox synced folders with

the \`SharedFoldersEnableSymlinksCreate\` option enabled. If the Vagrant

guest is not trusted, you may want to disable this option. For more

information on this option, please refer to the VirtualBox manual:

 

<https://www.virtualbox.org/manual/ch04.html#sharedfolders>

 

This option can be disabled globally with an environment variable:

 

VAGRANT_DISABLE_VBOXSYMLINKCREATE=1

 

or on a per folder basis within the Vagrantfile:

 

config.vm.synced_folder '/host/path', '/guest/path',
SharedFoldersEnableSymlinksCreate: false

==\> controlplane: Clearing any previously set network interfaces...

==\> controlplane: Preparing network interfaces based on
configuration...

controlplane: Adapter 1: nat

controlplane: Adapter 2: bridged

==\> controlplane: Forwarding ports...

controlplane: 22 (guest) =\> 2222 (host) (adapter 1)

==\> controlplane: Running 'pre-boot' VM customizations...

==\> controlplane: Booting VM...

==\> controlplane: Waiting for machine to boot. This may take a few
minutes...

controlplane: SSH address: 127.0.0.1:2222

controlplane: SSH username: vagrant

controlplane: SSH auth method: private key

controlplane: Warning: Connection reset. Retrying...

controlplane: Warning: Connection aborted. Retrying...

controlplane: Warning: Remote connection disconnect. Retrying...

controlplane:

controlplane: Vagrant insecure key detected. Vagrant will automatically
replace

controlplane: this with a newly generated keypair for better security.

controlplane:

controlplane: Inserting generated public key within guest...

controlplane: Removing insecure key from the guest if it's present...

controlplane: Key inserted! Disconnecting and reconnecting using new SSH
key...

==\> controlplane: Machine booted and ready!

==\> controlplane: Checking for guest additions in VM...

controlplane: The guest additions on this VM do not match the installed
version of

controlplane: VirtualBox! In most cases this is fine, but in rare cases
it can

controlplane: prevent things such as shared folders from working
properly. If you see

controlplane: shared folder errors, please make sure the guest additions
within the

controlplane: virtual machine match the version of VirtualBox you have
installed on

controlplane: your host and reload your VM.

controlplane:

controlplane: Guest Additions Version: 6.0.0 r127566

controlplane: VirtualBox Version: 7.1

==\> controlplane: Setting hostname...

==\> controlplane: Configuring and enabling network interfaces...

==\> controlplane: Mounting shared folders...

controlplane:
C:/Users/nibha/certified-kubernetes-administrator-course/kubeadm-clusters/virtualbox
=\> /vagrant

==\> controlplane: Running provisioner: setup-hosts (shell)...

controlplane: Running:
C:/Users/nibha/AppData/Local/Temp/vagrant-shell20250612-15784-pnuj0k.sh

==\> controlplane: Running provisioner: setup-dns (shell)...

controlplane: Running:
C:/Users/nibha/AppData/Local/Temp/vagrant-shell20250612-15784-thcu9d.sh

==\> controlplane: Running provisioner: setup-ssh (shell)...

controlplane: Running:
C:/Users/nibha/AppData/Local/Temp/vagrant-shell20250612-15784-fhg9s7.sh

==\> controlplane: Running provisioner: file...

controlplane: ./ubuntu/tmux.conf =\> \$HOME/.tmux.conf

==\> controlplane: Running provisioner: file...

controlplane: ./ubuntu/vimrc =\> \$HOME/.vimrc

==\> controlplane: Running action triggers after up ...

==\> controlplane: Running trigger: Post provisioner...

Nothing to do here

==\> node01: Box 'ubuntu/jammy64' could not be found. Attempting to find
and install...

node01: Box Provider: virtualbox

node01: Box Version: \>= 0

==\> node01: Loading metadata for box 'ubuntu/jammy64'

node01: URL: <https://vagrantcloud.com/api/v2/vagrant/ubuntu/jammy64>

==\> node01: Adding box 'ubuntu/jammy64' (v20241002.0.0) for provider:
virtualbox

==\> node01: Importing base box 'ubuntu/jammy64'...

==\> node01: Matching MAC address for NAT networking...

==\> node01: Setting the name of the VM: node01

==\> node01: Fixed port collision for 22 =\> 2222. Now on port 2200.

==\> node01: Clearing any previously set network interfaces...

==\> node01: Preparing network interfaces based on configuration...

node01: Adapter 1: nat

node01: Adapter 2: bridged

==\> node01: Forwarding ports...

node01: 22 (guest) =\> 2200 (host) (adapter 1)

==\> node01: Running 'pre-boot' VM customizations...

==\> node01: Booting VM...

==\> node01: Waiting for machine to boot. This may take a few minutes...

node01: SSH address: 127.0.0.1:2200

node01: SSH username: vagrant

node01: SSH auth method: private key

node01:

node01: Vagrant insecure key detected. Vagrant will automatically
replace

node01: this with a newly generated keypair for better security.

node01:

node01: Inserting generated public key within guest...

node01: Removing insecure key from the guest if it's present...

node01: Key inserted! Disconnecting and reconnecting using new SSH
key...

==\> node01: Machine booted and ready!

==\> node01: Checking for guest additions in VM...

node01: The guest additions on this VM do not match the installed
version of

node01: VirtualBox! In most cases this is fine, but in rare cases it can

node01: prevent things such as shared folders from working properly. If
you see

node01: shared folder errors, please make sure the guest additions
within the

node01: virtual machine match the version of VirtualBox you have
installed on

node01: your host and reload your VM.

node01:

node01: Guest Additions Version: 6.0.0 r127566

node01: VirtualBox Version: 7.1

==\> node01: Setting hostname...

==\> node01: Configuring and enabling network interfaces...

==\> node01: Mounting shared folders...

node01:
C:/Users/nibha/certified-kubernetes-administrator-course/kubeadm-clusters/virtualbox
=\> /vagrant

==\> node01: Running provisioner: setup-hosts (shell)...

node01: Running:
C:/Users/nibha/AppData/Local/Temp/vagrant-shell20250612-15784-76rgof.sh

==\> node01: Running provisioner: setup-dns (shell)...

node01: Running:
C:/Users/nibha/AppData/Local/Temp/vagrant-shell20250612-15784-7qw0er.sh

==\> node01: Running provisioner: setup-ssh (shell)...

node01: Running:
C:/Users/nibha/AppData/Local/Temp/vagrant-shell20250612-15784-hyzvvq.sh

==\> node01: Running action triggers after up ...

==\> node01: Running trigger: Post provisioner...

Nothing to do here

==\> node02: Box 'ubuntu/jammy64' could not be found. Attempting to find
and install...

node02: Box Provider: virtualbox

node02: Box Version: \>= 0

==\> node02: Loading metadata for box 'ubuntu/jammy64'

node02: URL: <https://vagrantcloud.com/api/v2/vagrant/ubuntu/jammy64>

==\> node02: Adding box 'ubuntu/jammy64' (v20241002.0.0) for provider:
virtualbox

==\> node02: Importing base box 'ubuntu/jammy64'...

==\> node02: Matching MAC address for NAT networking...

==\> node02: Setting the name of the VM: node02

==\> node02: Fixed port collision for 22 =\> 2222. Now on port 2201.

==\> node02: Clearing any previously set network interfaces...

==\> node02: Preparing network interfaces based on configuration...

node02: Adapter 1: nat

node02: Adapter 2: bridged

==\> node02: Forwarding ports...

node02: 22 (guest) =\> 2201 (host) (adapter 1)

==\> node02: Running 'pre-boot' VM customizations...

==\> node02: Booting VM...

==\> node02: Waiting for machine to boot. This may take a few minutes...

node02: SSH address: 127.0.0.1:2201

node02: SSH username: vagrant

node02: SSH auth method: private key

node02:

node02: Vagrant insecure key detected. Vagrant will automatically
replace

node02: this with a newly generated keypair for better security.

node02:

node02: Inserting generated public key within guest...

node02: Removing insecure key from the guest if it's present...

node02: Key inserted! Disconnecting and reconnecting using new SSH
key...

==\> node02: Machine booted and ready!

==\> node02: Checking for guest additions in VM...

node02: The guest additions on this VM do not match the installed
version of

node02: VirtualBox! In most cases this is fine, but in rare cases it can

node02: prevent things such as shared folders from working properly. If
you see

node02: shared folder errors, please make sure the guest additions
within the

node02: virtual machine match the version of VirtualBox you have
installed on

node02: your host and reload your VM.

node02:

node02: Guest Additions Version: 6.0.0 r127566

node02: VirtualBox Version: 7.1

==\> node02: Setting hostname...

==\> node02: Configuring and enabling network interfaces...

==\> node02: Mounting shared folders...

node02:
C:/Users/nibha/certified-kubernetes-administrator-course/kubeadm-clusters/virtualbox
=\> /vagrant

==\> node02: Running provisioner: setup-hosts (shell)...

node02: Running:
C:/Users/nibha/AppData/Local/Temp/vagrant-shell20250612-15784-hts8kr.sh

==\> node02: Running provisioner: setup-dns (shell)...

node02: Running:
C:/Users/nibha/AppData/Local/Temp/vagrant-shell20250612-15784-jd8mte.sh

==\> node02: Running provisioner: setup-ssh (shell)...

node02: Running:
C:/Users/nibha/AppData/Local/Temp/vagrant-shell20250612-15784-664zuo.sh

==\> node02: Running action triggers after up ...

==\> node02: Running trigger: Post provisioner...

Gathering IP addresses of nodes...

Setting /etc/hosts on nodes...

Uploading hosts.tmp to /tmp/hosts.tmp

Upload has completed successfully!

 

Source: hosts.tmp

Destination: /tmp/hosts.tmp

controlplane

node01

node02

Uploading hosts.tmp to /tmp/hosts.tmp

Upload has completed successfully!

 

Source: hosts.tmp

Destination: /tmp/hosts.tmp

controlplane

node01

node02

Uploading hosts.tmp to /tmp/hosts.tmp

Upload has completed successfully!

 

Source: hosts.tmp

Destination: /tmp/hosts.tmp

controlplane

node01

node02

 

VM build complete!

 

Use either of the following to access any NodePort services you create
from your browser

replacing "port_number" with the number of your NodePort.

 

[http://:port_number](NULL)

[http://:port_number](NULL)

 

PS
C:\Users\nibha\certified-kubernetes-administrator-course\kubeadm-clusters\virtualbox\>
vagrant status

Current machine states:

 

controlplane running (virtualbox)

node01 running (virtualbox)

node02 running (virtualbox)

 

This environment represents multiple VMs. The VMs are all listed

above with their current state. For more information about a specific

VM, run \`vagrant status NAME\`.

 

 

1\. Setup forwarding rules
<https://kubernetes.io/docs/setup/production-environment/container-runtimes/>

 

 

To manually enable IPv4 packet forwarding:

 

\# sysctl params required by setup, params persist across reboots

cat \<\<EOF \| sudo tee /etc/sysctl.d/k8s.conf

net.ipv4.ip_forward = 1

EOF

 

\# Apply sysctl params without reboot

sudo sysctl --system

 

Verify that net.ipv4.ip_forward is set to 1 with:

 

sysctl net.ipv4.ip_forward

 

 

 

set net.bridge.bridge-nf-call-iptables to 1:

 

cat \<\<EOF \| sudo tee /etc/modules-load.d/k8s.conf

br_netfilter

EOF

 

cat \<\<EOF \| sudo tee /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-ip6tables = 1

net.bridge.bridge-nf-call-iptables = 1

EOF

 

sudo sysctl --system

 

controlplane ~ ✖ cat /etc/\*-release ; check the Linux distrbution to
install kubeadm, kubelet and kubectl

 

 

2\. Installing kubeadm, kubelet and kubectl on all nodes
<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/>

 

\- Update the apt package index and install packages needed to use the
Kubernetes apt repository:

 

sudo apt-get update

 

\- Download the public signing key for the Kubernetes package
repositories. The same signing key is used for all repositories so you
can disregard the version in the URL:

 

curl -fsSL <https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key> \|
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

 

\- Add the appropriate Kubernetes apt repository.

 

echo 'deb \[signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg\]
<https://pkgs.k8s.io/core:/stable:/v1.33/deb/> /' \| sudo tee
/etc/apt/sources.list.d/kubernetes.list

 

\- Update the apt package index, install kubelet, kubeadm and kubectl,
and pin their version:

 

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

 

 

If you want to install specific version of kubelet and kubeadm

 

sudo apt-get install -y kubelet=1.33.0-00 kubeadm=1.33.0-00
kubectl=1.33.0-00

 

 

vagrant@controlplane:~\$ sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

Hit:1 <http://archive.ubuntu.com/ubuntu> jammy InRelease

Hit:2 <http://archive.ubuntu.com/ubuntu> jammy-updates InRelease

Hit:3 <http://archive.ubuntu.com/ubuntu> jammy-backports InRelease

Hit:5 <http://security.ubuntu.com/ubuntu> jammy-security InRelease

Get:4
<https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb>
InRelease \[1186 B\]

Get:6
<https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb>
Packages \[5202 B\]

Fetched 6388 B in 12s (533 B/s)

Reading package lists... Done

Reading package lists... Done

Building dependency tree... Done

Reading state information... Done

The following additional packages will be installed:

conntrack cri-tools kubernetes-cni

The following NEW packages will be installed:

conntrack cri-tools kubeadm kubectl kubelet kubernetes-cni

0 upgraded, 6 newly installed, 0 to remove and 57 not upgraded.

Need to get 95.4 MB of archives.

After this operation, 351 MB of additional disk space will be used.

Get:1 [http://archive.ubuntu.com/ubuntu
jammy/main](http://archive.ubuntu.com/ubuntu%20jammy/main) amd64
conntrack amd64 1:1.4.6-2build2 \[33.5 kB\]

Get:2
<https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb>
cri-tools 1.33.0-1.1 \[17.3 MB\]

Get:3
<https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb>
kubeadm 1.33.2-1.1 \[12.7 MB\]

Get:4
<https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb>
kubectl 1.33.2-1.1 \[11.7 MB\]

Get:5
<https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb>
kubernetes-cni 1.6.0-1.1 \[37.8 MB\]

Get:6
<https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb>
kubelet 1.33.2-1.1 \[15.9 MB\]

Fetched 95.4 MB in 23s (4202 kB/s)

Selecting previously unselected package conntrack.

(Reading database ... 64264 files and directories currently installed.)

Preparing to unpack .../0-conntrack_1%3a1.4.6-2build2_amd64.deb ...

Unpacking conntrack (1:1.4.6-2build2) ...

Selecting previously unselected package cri-tools.

Preparing to unpack .../1-cri-tools_1.33.0-1.1_amd64.deb ...

Unpacking cri-tools (1.33.0-1.1) ...

Selecting previously unselected package kubeadm.

Preparing to unpack .../2-kubeadm_1.33.2-1.1_amd64.deb ...

Unpacking kubeadm (1.33.2-1.1) ...

Selecting previously unselected package kubectl.

Preparing to unpack .../3-kubectl_1.33.2-1.1_amd64.deb ...

Unpacking kubectl (1.33.2-1.1) ...

Selecting previously unselected package kubernetes-cni.

Preparing to unpack .../4-kubernetes-cni_1.6.0-1.1_amd64.deb ...

Unpacking kubernetes-cni (1.6.0-1.1) ...

Selecting previously unselected package kubelet.

Preparing to unpack .../5-kubelet_1.33.2-1.1_amd64.deb ...

Unpacking kubelet (1.33.2-1.1) ...

Setting up conntrack (1:1.4.6-2build2) ...

Setting up kubectl (1.33.2-1.1) ...

Setting up cri-tools (1.33.0-1.1) ...

Setting up kubernetes-cni (1.6.0-1.1) ...

Setting up kubeadm (1.33.2-1.1) ...

Setting up kubelet (1.33.2-1.1) ...

Processing triggers for man-db (2.10.2-1) ...

Scanning processes...

Scanning linux images...

 

Running kernel seems to be up-to-date.

 

No services need to be restarted.

 

No containers need to be restarted.

 

No user sessions are running outdated binaries.

 

No VM guests are running outdated hypervisor (qemu) binaries on this
host.

kubelet set on hold.

kubeadm set on hold.

kubectl set on hold.

 

 

3.Creating a cluster with kubeadm
<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/>

 

\- Install a container runtime on all nodes
<https://kubernetes.io/docs/setup/production-environment/container-runtimes/>,
<https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd>

sudo apt update

sudo apt install -y containerd

\- Configuring cgroup driver: On Linux, control groups are used to
constrain resources that are allocated to processes in containers. There
are two cgroup drivers available such as cgroupfs, systemd. Both the
kubelet and container runtime must be configured to use same cgroup
driver.

When systemd is chosen as the init system for a Linux distribution, the
init process generates and consumes a root control group (cgroup) and
acts as a cgroup manager.

\> ps -p 1 ; to check init system in order to set relevant cgroup driver
for kubelet process

<https://kubernetes.io/docs/setup/production-environment/container-runtimes/>

i\. To set the cgroup driver on Kubelet, create a kubelet configuration
file. In version 1.22 or later, if user does not set the cgroup driver
field under kubelet configuration, kubeadm will default to systemd
<https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/>

If you want to setup cgroup driver on Kubelet manually, follow this
configuration

\# kubeadm-config.yaml

kind: ClusterConfiguration

apiVersion: kubeadm.k8s.io/v1beta4

kubernetesVersion: v1.21.0

---

kind: KubeletConfiguration

apiVersion: kubelet.config.k8s.io/v1beta1

cgroupDriver: systemd

 

Such a configuration file can then be passed to the kubeadm command:

 

\>kubeadm init --config kubeadm-config.yaml

 

ii\. Make sure that container runtime is set to utilize the cgroup
driver so that create config file for containerd which store in
/etc/containerd/config.toml. By default, it is not going to be using
systemd.
<https://kubernetes.io/docs/setup/production-environment/container-runtimes/>,
<https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cgroupfs-cgroup-driver>

\>sudo mkdir -p /etc/containerd ; create directory /etc/containerd on
all nodes

Now the containerd process allows to generate default configuration. If
you don't specify config file, it uses default configurations. Take a
look the configuration by running 'containerd config default'. With the
following command, set SystemdCgroup to true and send the config to
/etc/containerd/config.toml

\> containerd config default ; check containerd default configuration

\> containerd config default \| sed 's/SystemdCgroup =
false/SystemdCgroup = true/' \| sudo tee /etc/containerd/config.toml ;
set SystemdCgroup to true and send the config to
/etc/containerd/config.toml

\> cat /etc/containerd/config.toml \| grep -i SystemdCgroup -B 50 ;
printing out whole bunch of lines before the grep keyword to make sure
it is right section
\[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options\]

\> sudo systemctl restart containerd ; restart containerd

 

 

\- Initializing control-plane node
<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/>

1.(Recommended) If you have plans to upgrade this single control-plane
kubeadm cluster to high availability you should specify the
--control-plane-endpoint to set the shared endpoint for all
control-plane nodes. Such an endpoint can be either a DNS name or an IP
address of a load-balancer.

2.Choose a Pod network add-on, and verify whether it requires any
arguments to be passed to kubeadm init. Depending on which third-party
provider you choose, you might need to set the --pod-network-cidr to a
provider-specific value. See Installing a Pod network add-on.

 

3.(Optional) kubeadm tries to detect the container runtime by using a
list of well-known endpoints. To use different container runtime or if
there are more than one installed on the provisioned node, specify the
--cri-socket argument to kubeadm. See Installing a runtime.

Considerations about apiserver-advertise-address and
ControlPlaneEndpoint

While --apiserver-advertise-address can be used to set the advertised
address for this particular control-plane node's API server,
--control-plane-endpoint can be used to set the shared endpoint for all
control-plane nodes.

 

--control-plane-endpoint allows both IP addresses and DNS names that can
map to IP addresses. Please contact your network administrator to
evaluate possible solutions with respect to such mapping.

 

--upload-certs this make sure that we upload all of the certificates to
a secret so all of the other nodes will have access to the certificates.

\> sudo kubeadm init --apiserver-advertise-address 192.168.56.11
--pod-network-cidr "10.244.0.0/16" --upload-certs

\*\*sudo kubeadm init --apiserver-advertise-address 192.168.183.209
--apiserver-cert-extra-sans=controlplane
--pod-network-cidr=172.17.0.0/16 --service-cidr=172.20.0.0/16

\*\*\*If the command throws the following error, enable IPv4 packet
forwarding
<https://kubernetes.io/docs/setup/production-environment/container-runtimes/>

vagrant@controlplane:~\$ sudo kubeadm init --apiserver-advertise-address
192.168.56.11 --pod-network-cidr "10.244.0.0/16" --upload-certs

\[init\] Using Kubernetes version: v1.33.2

\[preflight\] Running pre-flight checks

error execution phase preflight: \[preflight\] Some fatal errors
occurred:

\[ERROR FileContent--proc-sys-net-ipv4-ip_forward\]:
/proc/sys/net/ipv4/ip_forward contents are not set to 1

\[preflight\] If you know what you are doing, you can make a check
non-fatal with \`--ignore-preflight-errors=...\`

To see the stack trace of this error execute with --v=5 or higher

vagrant@controlplane:~\$ sudo kubeadm init --apiserver-advertise-address
192.168.56.11 --pod-network-cidr "10.244.0.0/16" --upload-certs

\[init\] Using Kubernetes version: v1.33.2

\[preflight\] Running pre-flight checks

\[preflight\] Pulling images required for setting up a Kubernetes
cluster

\[preflight\] This might take a minute or two, depending on the speed of
your internet connection

\[preflight\] You can also perform this action beforehand using 'kubeadm
config images pull'

W0619 09:06:19.462904 4637 checks.go:846\] detected that the sandbox
image "registry.k8s.io/pause:3.8" of the container runtime is
inconsistent with that used by kubeadm.It is recommended to use
"registry.k8s.io/pause:3.10" as the CRI sandbox image.

\[certs\] Using certificateDir folder "/etc/kubernetes/pki"

\[certs\] Generating "ca" certificate and key

\[certs\] Generating "apiserver" certificate and key

\[certs\] apiserver serving cert is signed for DNS names \[controlplane
kubernetes kubernetes.default kubernetes.default.svc
kubernetes.default.svc.cluster.local\] and IPs \[10.96.0.1
192.168.56.11\]

\[certs\] Generating "apiserver-kubelet-client" certificate and key

\[certs\] Generating "front-proxy-ca" certificate and key

\[certs\] Generating "front-proxy-client" certificate and key

\[certs\] Generating "etcd/ca" certificate and key

\[certs\] Generating "etcd/server" certificate and key

\[certs\] etcd/server serving cert is signed for DNS names
\[controlplane localhost\] and IPs \[192.168.56.11 127.0.0.1 ::1\]

\[certs\] Generating "etcd/peer" certificate and key

\[certs\] etcd/peer serving cert is signed for DNS names \[controlplane
localhost\] and IPs \[192.168.56.11 127.0.0.1 ::1\]

\[certs\] Generating "etcd/healthcheck-client" certificate and key

\[certs\] Generating "apiserver-etcd-client" certificate and key

\[certs\] Generating "sa" key and public key

\[kubeconfig\] Using kubeconfig folder "/etc/kubernetes"

\[kubeconfig\] Writing "admin.conf" kubeconfig file

\[kubeconfig\] Writing "super-admin.conf" kubeconfig file

\[kubeconfig\] Writing "kubelet.conf" kubeconfig file

\[kubeconfig\] Writing "controller-manager.conf" kubeconfig file

\[kubeconfig\] Writing "scheduler.conf" kubeconfig file

\[etcd\] Creating static Pod manifest for local etcd in
"/etc/kubernetes/manifests"

\[control-plane\] Using manifest folder "/etc/kubernetes/manifests"

\[control-plane\] Creating static Pod manifest for "kube-apiserver"

\[control-plane\] Creating static Pod manifest for
"kube-controller-manager"

\[control-plane\] Creating static Pod manifest for "kube-scheduler"

\[kubelet-start\] Writing kubelet environment file with flags to file
"/var/lib/kubelet/kubeadm-flags.env"

\[kubelet-start\] Writing kubelet configuration to file
"/var/lib/kubelet/config.yaml"

\[kubelet-start\] Starting the kubelet

\[wait-control-plane\] Waiting for the kubelet to boot up the control
plane as static Pods from directory "/etc/kubernetes/manifests"

\[kubelet-check\] Waiting for a healthy kubelet at
<http://127.0.0.1:10248/healthz>. This can take up to 4m0s

\[kubelet-check\] The kubelet is healthy after 1.002661742s

\[control-plane-check\] Waiting for healthy control plane components.
This can take up to 4m0s

\[control-plane-check\] Checking kube-apiserver at
<https://192.168.56.11:6443/livez>

\[control-plane-check\] Checking kube-controller-manager at
<https://127.0.0.1:10257/healthz>

\[control-plane-check\] Checking kube-scheduler at
<https://127.0.0.1:10259/livez>

\[control-plane-check\] kube-controller-manager is healthy after
8.65447605s

\[control-plane-check\] kube-scheduler is healthy after 10.58169592s

\[control-plane-check\] kube-apiserver is healthy after 13.006786736s

\[upload-config\] Storing the configuration used in ConfigMap
"kubeadm-config" in the "kube-system" Namespace

\[kubelet\] Creating a ConfigMap "kubelet-config" in namespace
kube-system with the configuration for the kubelets in the cluster

\[upload-certs\] Storing the certificates in Secret "kubeadm-certs" in
the "kube-system" Namespace

\[upload-certs\] Using certificate key:

08954bf2d62f000c09d0e0ff918e261f093dd96e9c0deb191f61521ad5505b56

\[mark-control-plane\] Marking the node controlplane as control-plane by
adding the labels: \[node-role.kubernetes.io/control-plane
node.kubernetes.io/exclude-from-external-load-balancers\]

\[mark-control-plane\] Marking the node controlplane as control-plane by
adding the taints \[node-role.kubernetes.io/control-plane:NoSchedule\]

\[bootstrap-token\] Using token: o4cfas.3534vrh4aiup3qo5

\[bootstrap-token\] Configuring bootstrap tokens, cluster-info
ConfigMap, RBAC Roles

\[bootstrap-token\] Configured RBAC rules to allow Node Bootstrap tokens
to get nodes

\[bootstrap-token\] Configured RBAC rules to allow Node Bootstrap tokens
to post CSRs in order for nodes to get long term certificate credentials

\[bootstrap-token\] Configured RBAC rules to allow the csrapprover
controller automatically approve CSRs from a Node Bootstrap Token

\[bootstrap-token\] Configured RBAC rules to allow certificate rotation
for all node client certificates in the cluster

\[bootstrap-token\] Creating the "cluster-info" ConfigMap in the
"kube-public" namespace

\[kubelet-finalize\] Updating "/etc/kubernetes/kubelet.conf" to point to
a rotatable kubelet client certificate and key

\[addons\] Applied essential addon: CoreDNS

\[addons\] Applied essential addon: kube-proxy

 

Your Kubernetes control-plane has initialized successfully!

 

To start using your cluster, you need to run the following as a regular
user:

 

mkdir -p \$HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf \$HOME/.kube/config

sudo chown \$(id -u):\$(id -g) \$HOME/.kube/config

 

Alternatively, if you are the root user, you can run:

 

export KUBECONFIG=/etc/kubernetes/admin.conf

 

You should now deploy a pod network to the cluster.

Run "kubectl apply -f \[podnetwork\].yaml" with one of the options
listed at:

<https://kubernetes.io/docs/concepts/cluster-administration/addons/>

 

Then you can join any number of worker nodes by running the following on
each as root:

 

kubeadm join 192.168.56.11:6443 --token 5xf60f.9cs2ig2lm2snvx7c \\

--discovery-token-ca-cert-hash
sha256:5d071e1fc348a1cc9ccdef728fae2b387e99e2649de3ac100fb6fac0ee7cc9de

 

 

Then it's going to have provisioned a config file so that we can
actually connect to our Kubernetes cluster.        

 

vagrant@controlplane:~\$ sudo cat /etc/kubernetes/admin.conf ; check
kubeconfig file so that we can actually communicate to master node

apiVersion: v1

clusters:

\- cluster:

certificate-authority-data:
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJS2xaT0lsN3djWm93RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBMk1UWXdOVEF6TXpGYUZ3MHpOVEEyTVRRd05UQTRNekZhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURFYllwL3RNNTNDOTMvWVI3TUdtREM5UkNXWncydmV4ZlNNcFgzallCU0xmS2tLdVh5S1VoNHN2N1MKK29LOFZvQ08xeGxULzdQdG02YnpKMzNwcGRVeE9KdVZuZzBiL0djbUxKSkpocVcyT2FaWlFGN1BMSUMycVhsWQpUeU80cDBnZ2Q3K20vRVUwekVmVXkxalpPekdjK095Yk9VRkVnMUNja0RzNFBpRU02RjljYmh4aUh6YTlVNkdTCnFsaElpMGRxSkhFYzZJeVc5M2xDV3I4SjlGNnpTVnpya1l2cTZLS290RGJkenlhTzRnNTZTSzdTQ1daMHpBa0EKTVNKVGY5dU0xaGNHVklESldKc3pMZ2U4eVZZSkd2SVBob1gzMGdWUkJPd2hTK2lqSEMvTjNOZVV4VEdSTERFVQppNGJOaHlOK1dqTHYybUVUVThZU1R2ZjFjcHBMQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJUYU9NeE5ZcE9mV0ZsTXNNVnJCUXpRUVZRbndEQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQlRqSnl5NHQzcAo2d2pWc29zaTEza2ptdzdQNi9ndVRYMk9qMjZ0WCtiOEhvRkNzZ1ZJd0NFTG9XMFdzeXNDVWMvTDZZb0oxUjJuCk9Ldm12RkpsaUtWdmhleVAzWWt6UW9oYjlha1l6NjNSTUlyRWxSdGxVRUR6NDEwWmZHdDU0V1Y5TlprNFVZbXUKVmYvMHM3M0Y3cXJYdDVBSXMvYTVrRklmeTZCTEp1YktrekRVcXJGdDhFYXU3Y0JYSlY0WWx4RzEvY0pIWlJ0TApWbThsVDJWZm9XRWlJY2I4c3lkNUdXb2E1TFdHQjlKdm10cDJ4U0tpR1hpV0xpci9sNkYzR2haVnN3OEUxSUMvCmxFcm5zQVh4bmhqUTVYa2JIZlVidW5iZ0lOd091SE90dnZYZkpocEk2WS9MSi9mcHdxN2diTFhjTldmMEZycFgKakZadENWbmN2TEx5Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K

server: <https://192.168.1.6:6443>

name: kubernetes

contexts:

\- context:

cluster: kubernetes

user: kubernetes-admin

name: kubernetes-admin@kubernetes

current-context: kubernetes-admin@kubernetes

kind: Config

preferences: {}

users:

\- name: kubernetes-admin

user:

client-certificate-data:
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURLVENDQWhHZ0F3SUJBZ0lJU3A4aFF6LzhtQ0l3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBMk1UWXdOVEF6TXpGYUZ3MHlOakEyTVRZd05UQTRNekZhTUR3eApIekFkQmdOVkJBb1RGbXQxWW1WaFpHMDZZMngxYzNSbGNpMWhaRzFwYm5NeEdUQVhCZ05WQkFNVEVHdDFZbVZ5CmJtVjBaWE10WVdSdGFXNHdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFDblA1V2YKRXNNUTFUVE5HVzNoTERBS3VobGFwblFoYU9aUkNEamFqN2xVbkk2V0FqWlg2cGxDelNUbUppUGxvb0Q4T3ZJOQpXWFJnbi8vWTJJeWczTVFJNGovVzFiYUZkRWJseGYyeEdHTEk4YnRqRDVLTWV5R1k5WHBQWlV4eXZxZW1Na1dnCmwzd1FBL2hVMURRZjlrUkJsZUZyZE55S3ExYkUrbm41T1ZnVU9aSU1YK0dwV0ljWUpWeVNZS2tKaFIzQnRFYWwKKzVCaVR6N0tqQ0EwMnlrVFRkaFJXbHdLQlhiY2xiSkc0QzMyajZoMjRMaTRvWFhBSW5ER2M4RlIxekw1QVJjdAozdlBBakZtWlp6Mjlmems2ZTJadHc2SnlhZ2JEbWlUcFZxMFhqeXNVazROblc5YkJ2SW84NzVneEpFY2xpQzlPCngzUlg5VE9mUStKWUhIOGpBZ01CQUFHalZqQlVNQTRHQTFVZER3RUIvd1FFQXdJRm9EQVRCZ05WSFNVRUREQUsKQmdnckJnRUZCUWNEQWpBTUJnTlZIUk1CQWY4RUFqQUFNQjhHQTFVZEl3UVlNQmFBRk5vNHpFMWlrNTlZV1V5dwp4V3NGRE5CQlZDZkFNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUJBUUN4UjZVVTlWb3h5cDJueTVOaDlZdk5KUTRTCjBPMlAxUlNHb3k4QjRrSmF1a0IxaGl1cHE5Vld5dUQvTTVPenlZRXIwZy9lR1I5MndpM1pQenBVay8wNVVXY1EKeG9tOGxEakpVSTJPVW9xWFZhUmRwb2Q0QXRaZlBWcmRJUmxvUVBvQ2hNQWExT2EyMndYQnNZUENrL1lhcE5kKwo3OGpyZ1V4anlYS3NTcnc5RHVua3FybTBLZ1Y5OFJKcEVxQm5LWHNhVDhvamdqNUpkUUVQbURiWDFwaVdzcjYrCkZYd3IwcFdseElmYXp2MEo2Rnh5VGZPZml5SWd1eGRNZnFMUXVLNnM2WU1hbVJ0b1d3VkVaNlZPenVoazgweVcKbDFyYitlVVNwWXA5T0ZLQmRvcE1CcVZ4OEFtY1dhRnJHTEt2ak9YUHB3a0oydWlnZkVJeHV0TmtrcGJQCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K

client-key-data:
LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBcHorVm54TERFTlUwelJsdDRTd3dDcm9aV3FaMElXam1VUWc0Mm8rNVZKeU9sZ0kyClYrcVpRczBrNWlZajVhS0EvRHJ5UFZsMFlKLy8yTmlNb056RUNPSS8xdFcyaFhSRzVjWDlzUmhpeVBHN1l3K1MKakhzaG1QVjZUMlZNY3I2bnBqSkZvSmQ4RUFQNFZOUTBIL1pFUVpYaGEzVGNpcXRXeFBwNStUbFlGRG1TREYvaApxVmlIR0NWY2ttQ3BDWVVkd2JSR3BmdVFZazgreW93Z05Oc3BFMDNZVVZwY0NnVjIzSld5UnVBdDlvK29kdUM0CnVLRjF3Q0p3eG5QQlVkY3krUUVYTGQ3endJeFptV2M5dlg4NU9udG1iY09pY21vR3c1b2s2VmF0RjQ4ckZKT0QKWjF2V3dieUtQTytZTVNSSEpZZ3ZUc2QwVi9Vem4wUGlXQngvSXdJREFRQUJBb0lCQUVWd1p1UzIyM3AvcXY2RQowbFF0RG4xRnF3VXlHd0ZNRm1RS2IrQit6OG5MZm4vVngwNVZQdkMzVDl5bWRUMEFiV0ZMOEF0OHVvN3JuMFFmCkZMUXROTThqY0xIMzVXSWZWdGhMamZzYlR0aHpWUVBhRVZSMlF3ckkwTkZxRTRXSWx2NnYzS3pDZUdZRjMvTUkKYitQZFVaV1JieGc0V3VEUUpxUitBbXB4WHBwMmFHS0FvNytBK3N2OGFOR1hoSzhtSjB6SDhSdTZZUG9MMEpPMwpiWE9BOGN4bU5HZHdRSEN3SldKNlZoUnFTTzlYd095dGovaDhYK2l4VnJQdDZBdU9xKyswcUY5bW9vdCs3RDhuCk5pQ3ROUjhwMEdUTTNIczc4enN5Z0tSV0JYOTlvekJEdnRKUGJkaEFsS1FWSk1HNXhrRSs3em9XOHRqS3h6cDIKSUR1VktzRUNnWUVBd0kyc1h3VnkrRVBxNGxuYkU3N1hhK054Y0RlZnNWR1hiTW1adDkwaWQ3NWdyNDcrd1FyegpNeW45SmdDZG1vbld2K245NDZmQ1RJRDlRamRrK1dXVWFDS1hJU3BRRTRnenJuOTB4VllKL0NnVFZtSE0xakEzCnhvRTJSVzdZRjd2MnZ3OGcxYUNXS1lwYjFSd3kwbHRQVGQ0ZjVzcUJSRG5LdTZOU2Q5WForM0VDZ1lFQTNsdGYKYjNJb1ZkTU02akxNWXpJVHloMHc0enEvN25RTEltbFhaM2ZrNWtUZ05MRTZnaGZZNlNWdnNSdzlPQ0ZIcjdPSAo0ckJOTnp4Mko4cXVqWlBmWHNCYThkRTIxVnlYbC9JQkxFRWRyOC9tV0ZwcTRtTFIreGoxWFZEc0hkSUpDSEhYCmVCeG5lcTJLeGVmemJWbjFDWlZWcG43OExHVURreFpVZXA1YjBkTUNnWUJQL2dzdjVubWV2Y2hraEdQSXppMTgKT3A2enVjZnp5Q3VhSEI1cURjbjE2UGNOdCtBSm5senVVbkZDU2xzdkNqaFd2Z1NsZ05udEltRVVFeU0xb0llSQp3Mlg1ZHpTUkNMSS91ZDl1NDVGK1diKyswUjVobE9KMjZ0TDdBVk53ZUx2SEg2ZktWODVLTDVwNjdCNEkyYm9XClpVTG5oQlV6bkxhbVFQRlBxMTNGWVFLQmdHSE9PVVZSSm5MUjJaMXVhTDY5T1QwdTUrRWJaMjhndGdFVjZ1OUYKS25jckZwKzIwVENHQXRCU3VqVmlZWDgrREVVNXArekxkYnQxS2RmSTdFeTBIWTJVK0t2alJjWkY2eHhYdzdKTwpsaHRPVEZqU3RLOE5ucVZmVGhZVTBUY3d1dTV1L01NRlkzTHZ3Tm5hZEJYR2V1VVRQZEIxak1DT1Vpc3hxRWtqClBPb25Bb0dBT3A2UUdTa2MybnJTUmp0bUVlODVNR1ZzdUowNnA1cTR0Vi9tTTVqL0RWS3FLWExEdFFjYThWbjUKaVZ6QTFYQjFjZk9xQU5CcDY2ZEFUTXdBazRDNHcxd0V0VEpjcFR4VDU1U0wyR05QNjlSeW9LYXRSeWlNQmJPWApYd1BUaXcyRVZ3ZGVoeVZCR2IrbFBzUFc4bkVTYjVhVzBNVTBBNExDWlNwY3UvOVJZdGM9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==

 

 

To start using your cluster, copy kubeconfig file to the default home
directory. Then you need to run the following as a regular user:

 

mkdir -p \$HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf \$HOME/.kube/config

sudo chown \$(id -u):\$(id -g) \$HOME/.kube/config

 

Alternatively, if you are the root user, you can run:

 

export KUBECONFIG=/etc/kubernetes/admin.conf

Check node status. Note that node status is currently in a not ready
state as we haven't set up the network plugin up and running. The node
will be moved to ready status

vagrant@controlplane:~\$ kubectl get nodes

NAME STATUS ROLES AGE VERSION

controlplane NotReady control-plane 56m v1.33.1

\*\*If you have multiple master nodes, the output looks like below. Run
the command so that you can bootstrap the other control plane nodes and
connect them to the cluster.
<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/>

The output looks similar to:

 

...

You can now join any number of control-plane node by running the
following command on each as a root:

kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv
--discovery-token-ca-cert-hash
sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
--control-plane --certificate-key
f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07

 

Please note that the certificate-key gives access to cluster sensitive
data, keep it secret!

As a safeguard, uploaded-certs will be deleted in two hours; If
necessary, you can use kubeadm init phase upload-certs to reload certs
afterward.

 

Then you can join any number of worker nodes by running the following on
each as root:

kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv
--discovery-t

\- Deploy a pod network for your cluster which is netowrk add on or CNI
plugin
<https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy>

We choose flannel as CNI plugin
<https://github.com/flannel-io/flannel#deploying-flannel-manually>

\> wget
<https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml>

vagrant@controlplane:~\$ wget
<https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml>

--2025-06-16 08:43:16--
<https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml>

Resolving github.com (github.com)... 20.205.243.166

Connecting to github.com (github.com)\|20.205.243.166\|:443...
connected.

HTTP request sent, awaiting response... 302 Found

Location:
<https://github.com/flannel-io/flannel/releases/download/v0.27.0/kube-flannel.yml>
\[following\]

--2025-06-16 08:43:17--
<https://github.com/flannel-io/flannel/releases/download/v0.27.0/kube-flannel.yml>

Reusing existing connection to github.com:443.

HTTP request sent, awaiting response... 302 Found

Location:
<https://release-assets.githubusercontent.com/github-production-release-asset/21704134/f2a1f6c0-b18a-4ff2-8c7c-e6ae8fdbe3eb?sp=r&sv=2018-11-09&sr=b&spr=https&se=2025-06-16T08%3A48%3A17Z&skoid=96c2d410-5711-43a1-aedd-ab1947aa7ab0&sktid=398a6654-997b-47e9-b12b-9515b896b4de&skt=2025-06-16T08%3A43%3A17Z&ske=2025-06-16T09%3A43%3A18Z&sks=b&skv=2018-11-09&sig=qh%2Bk39cZlektJ5k0qc6rB0EApJPh7TWyMx73MN9yibc%3D&jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmVsZWFzZS1hc3NldHMuZ2l0aHVidXNlcmNvbnRlbnQuY29tIiwia2V5Ijoia2V5MSIsImV4cCI6MTc1MDA2MzY5NywibmJmIjoxNzUwMDYzMzk3LCJwYXRoIjoicmVsZWFzZWFzc2V0cHJvZHVjdGlvbi5ibG9iLmNvcmUud2luZG93cy5uZXQifQ.aMSm_GgBZ6y7lvyutBv4sXhLfQnPLS8Ftf4bYuWPWmQ&response-content-disposition=attachment%3B%20filename%3Dkube-flannel.yml&response-content-type=application%2Foctet-stream>
\[following\]

--2025-06-16 08:43:18--
<https://release-assets.githubusercontent.com/github-production-release-asset/21704134/f2a1f6c0-b18a-4ff2-8c7c-e6ae8fdbe3eb?sp=r&sv=2018-11-09&sr=b&spr=https&se=2025-06-16T08%3A48%3A17Z&skoid=96c2d410-5711-43a1-aedd-ab1947aa7ab0&sktid=398a6654-997b-47e9-b12b-9515b896b4de&skt=2025-06-16T08%3A43%3A17Z&ske=2025-06-16T09%3A43%3A18Z&sks=b&skv=2018-11-09&sig=qh%2Bk39cZlektJ5k0qc6rB0EApJPh7TWyMx73MN9yibc%3D&jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmVsZWFzZS1hc3NldHMuZ2l0aHVidXNlcmNvbnRlbnQuY29tIiwia2V5Ijoia2V5MSIsImV4cCI6MTc1MDA2MzY5NywibmJmIjoxNzUwMDYzMzk3LCJwYXRoIjoicmVsZWFzZWFzc2V0cHJvZHVjdGlvbi5ibG9iLmNvcmUud2luZG93cy5uZXQifQ.aMSm_GgBZ6y7lvyutBv4sXhLfQnPLS8Ftf4bYuWPWmQ&response-content-disposition=attachment%3B%20filename%3Dkube-flannel.yml&response-content-type=application%2Foctet-stream>

Resolving release-assets.githubusercontent.com
(release-assets.githubusercontent.com)... 185.199.111.133,
185.199.108.133, 185.199.109.133, ...

Connecting to release-assets.githubusercontent.com
(release-assets.githubusercontent.com)\|185.199.111.133\|:443...
connected.

HTTP request sent, awaiting response... 200 OK

Length: 4409 (4.3K) \[application/octet-stream\]

Saving to: ‘kube-flannel.yml’

 

kube-flannel.yml
100%\[===================================================================\>\]
4.31K --.-KB/s in 0s

 

2025-06-16 08:43:18 (13.9 MB/s) - ‘kube-flannel.yml’ saved \[4409/4409\]

 

vagrant@controlplane:~\$ ls

kube-flannel.yml

vagrant@controlplane:~\$ cat kube-flannel.yml

apiVersion: v1

kind: Namespace

metadata:

labels:

k8s-app: flannel

pod-security.kubernetes.io/enforce: privileged

name: kube-flannel

---

apiVersion: v1

kind: ServiceAccount

metadata:

labels:

k8s-app: flannel

name: flannel

namespace: kube-flannel

---

apiVersion: rbac.authorization.k8s.io/v1

kind: ClusterRole

metadata:

labels:

k8s-app: flannel

name: flannel

rules:

\- apiGroups:

\- ""

resources:

\- pods

verbs:

\- get

\- apiGroups:

\- ""

resources:

\- nodes

verbs:

\- get

\- list

\- watch

\- apiGroups:

\- ""

resources:

\- nodes/status

verbs:

\- patch

---

apiVersion: rbac.authorization.k8s.io/v1

kind: ClusterRoleBinding

metadata:

labels:

k8s-app: flannel

name: flannel

roleRef:

apiGroup: rbac.authorization.k8s.io

kind: ClusterRole

name: flannel

subjects:

\- kind: ServiceAccount

name: flannel

namespace: kube-flannel

---

apiVersion: v1

data:

cni-conf.json: \|

{

"name": "cbr0",

"cniVersion": "0.3.1",

"plugins": \[

{

"type": "flannel",

"delegate": {

"hairpinMode": true,

"isDefaultGateway": true

}

},

{

"type": "portmap",

"capabilities": {

"portMappings": true

}

}

\]

}

net-conf.json: \|

{

"Network": "10.244.0.0/16", \>\>\>\>\>\>\> The CIDR should be matched
the network specified with option --pod-network-cidr in kubeadm init
command

"EnableNFTables": false,

"Backend": {

"Type": "vxlan"

}

}

kind: ConfigMap

metadata:

labels:

app: flannel

k8s-app: flannel

tier: node

name: kube-flannel-cfg

namespace: kube-flannel

---

apiVersion: apps/v1

kind: DaemonSet

metadata:

labels:

app: flannel

k8s-app: flannel

tier: node

name: kube-flannel-ds

namespace: kube-flannel

spec:

selector:

matchLabels:

app: flannel

k8s-app: flannel

template:

metadata:

labels:

app: flannel

k8s-app: flannel

tier: node

spec:

affinity:

nodeAffinity:

requiredDuringSchedulingIgnoredDuringExecution:

nodeSelectorTerms:

\- matchExpressions:

\- key: kubernetes.io/os

operator: In

values:

\- linux

containers:

\- args:

\- --ip-masq

\- --kube-subnet-mgr

command:

\- /opt/bin/flanneld

env:

\- name: POD_NAME

valueFrom:

fieldRef:

fieldPath: metadata.name

\- name: POD_NAMESPACE

valueFrom:

fieldRef:

fieldPath: metadata.namespace

\- name: EVENT_QUEUE_DEPTH

value: "5000"

image: ghcr.io/flannel-io/flannel:v0.27.0

name: kube-flannel

resources:

requests:

cpu: 100m

memory: 50Mi

securityContext:

capabilities:

add:

\- NET_ADMIN

\- NET_RAW

privileged: false

volumeMounts:

\- mountPath: /run/flannel

name: run

\- mountPath: /etc/kube-flannel/

name: flannel-cfg

\- mountPath: /run/xtables.lock

name: xtables-lock

hostNetwork: true

initContainers:

\- args:

\- -f

\- /flannel

\- /opt/cni/bin/flannel

command:

\- cp

image: ghcr.io/flannel-io/flannel-cni-plugin:v1.7.1-flannel1

name: install-cni-plugin

volumeMounts:

\- mountPath: /opt/cni/bin

name: cni-plugin

\- args:

\- -f

\- /etc/kube-flannel/cni-conf.json

\- /etc/cni/net.d/10-flannel.conflist

command:

\- cp

image: ghcr.io/flannel-io/flannel:v0.27.0

name: install-cni

volumeMounts:

\- mountPath: /etc/cni/net.d

name: cni

\- mountPath: /etc/kube-flannel/

name: flannel-cfg

priorityClassName: system-node-critical

serviceAccountName: flannel

tolerations:

\- effect: NoSchedule

operator: Exists

volumes:

\- hostPath:

path: /run/flannel

name: run

\- hostPath:

path: /opt/cni/bin

name: cni-plugin

\- hostPath:

path: /etc/cni/net.d

name: cni

\- configMap:

name: kube-flannel-cfg

name: flannel-cfg

\- hostPath:

path: /run/xtables.lock

type: FileOrCreate

name: xtables-lock

 

 

vagrant@controlplane:~\$ kubectl apply -f kube-flannel.yml

namespace/kube-flannel created

serviceaccount/flannel created

clusterrole.rbac.authorization.k8s.io/flannel created

clusterrolebinding.rbac.authorization.k8s.io/flannel created

configmap/kube-flannel-cfg created

daemonset.apps/kube-flannel-ds created

\- Join any worker nodes to the cluster by running following command on
worker nodes

sudo kubeadm join 192.168.56.11:6443 --token o4cfas.3534vrh4aiup3qo5 \\

--discovery-token-ca-cert-hash
sha256:02a2f75b4f2cdfcd8cdf85a46f1ec6188de8943c9dc287b525b6d0f053728255

 

Output:

 

vagrant@node01:~\$ sudo kubeadm join 192.168.56.11:6443 --token
f3vxmn.ext263v8hshqh4xh --discovery-token-ca-cert-hash
sha256:5d071e1fc348a1cc9ccd

ef728fae2b387e99e2649de3ac100fb6fac0ee7cc9de

\[preflight\] Running pre-flight checks

\[preflight\] Reading configuration from the "kubeadm-config" ConfigMap
in namespace "kube-system"...

\[preflight\] Use 'kubeadm init phase upload-config --config
your-config-file' to re-upload it.

\[kubelet-start\] Writing kubelet configuration to file
"/var/lib/kubelet/config.yaml"

\[kubelet-start\] Writing kubelet environment file with flags to file
"/var/lib/kubelet/kubeadm-flags.env"

\[kubelet-start\] Starting the kubelet

\[kubelet-check\] Waiting for a healthy kubelet at
<http://127.0.0.1:10248/healthz>. This can take up to 4m0s

\[kubelet-check\] The kubelet is healthy after 1.00294237s

\[kubelet-start\] Waiting for the kubelet to perform the TLS Bootstrap

 

This node has joined the cluster:

\* Certificate signing request was sent to apiserver and a response was
received.

\* The Kubelet was informed of the new secure connection details.

 

Run 'kubectl get nodes' on the control-plane to see this node join the
cluster.

 

\*Check node status and check pods in network plugin namespace. It's
going to deploy a instance of the that the network plugin agent on every
single one of the nodes so that it can actually handle network on all of
them.

controlplane ~ ➜ kubectl get nodes

NAME STATUS ROLES AGE VERSION

controlplane Ready control-plane 12m v1.32.0

node01 Ready \<none\> 10m v1.32.0

 

controlplane ~ ➜ kubectl get pods -A

NAMESPACE NAME READY STATUS RESTARTS AGE

kube-flannel kube-flannel-ds-8hrm6 1/1 Running 0 66s

kube-flannel kube-flannel-ds-skghj 1/1 Running 0 66s

kube-system coredns-668d6bf9bc-2x2t2 1/1 Running 0 12m

kube-system coredns-668d6bf9bc-dn7hf 1/1 Running 0 12m

kube-system etcd-controlplane 1/1 Running 0 12m

kube-system kube-apiserver-controlplane 1/1 Running 0 12m

kube-system kube-controller-manager-controlplane 1/1 Running 0 12m

kube-system kube-proxy-2rrkj 1/1 Running 0 10m

kube-system kube-proxy-rpgqj 1/1 Running 0 12m

kube-system kube-scheduler-controlplane 1/1 Running 0 12m

--------------------------------------------------------------------------------------------

curl -fsSL <https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key> \|
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb \[signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg\]
<https://pkgs.k8s.io/core:/stable:/v1.32/deb/> /' \| sudo tee
/etc/apt/sources.list.d/kubernetes.list

 

 

sudo apt-get install -y kubelet=1.32.0-1.1 kubeadm=1.32.0-1.1
kubectl=1.32.0-1.1

sudo kubeadm init --apiserver-advertise-address 192.168.63.130
--apiserver-cert-extra-sans=controlplane
--pod-network-cidr=172.17.0.0/16 --service-cidr=172.20.0.0/16

kubeadm join 192.168.63.130:6443 --token upc4ig.dqoey239zjv0flrw \\

--discovery-token-ca-cert-hash
sha256:0f182b5435d3e6351380d3523d391127b836760ad531275e8b8d5325e876a51a

 

---------------------------------------------------------------------------------------------------------------------------

 

These steps have to be performed on both nodes.

 

set net.bridge.bridge-nf-call-iptables to 1:

 

cat \<\<EOF \| sudo tee /etc/modules-load.d/k8s.conf

br_netfilter

EOF

 

cat \<\<EOF \| sudo tee /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-ip6tables = 1

net.bridge.bridge-nf-call-iptables = 1

EOF

 

sudo sysctl --system

 

The container runtime has already been installed on both nodes, so you
may skip this step.

Install kubeadm, kubectl and kubelet on all nodes:

 

sudo apt-get update

 

sudo apt-get install -y apt-transport-https ca-certificates curl

 

curl -fsSL <https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key> \|
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

 

echo 'deb \[signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg\]
<https://pkgs.k8s.io/core:/stable:/v1.32/deb/> /' \| sudo tee
/etc/apt/sources.list.d/kubernetes.list

 

sudo apt-get update

 

\# To see the new version labels

sudo apt-cache madison kubeadm

 

sudo apt-get install -y kubelet=1.32.0-1.1 kubeadm=1.32.0-1.1
kubectl=1.32.0-1.1

 

sudo apt-mark hold kubelet kubeadm kubectl

 

 

controlplane ~ ➜ kubeadm join 192.168.183.209:6443 --token
06xstl.4n1dai9jw3d0ftct \\

--discovery-token-ca-cert-hash
sha256:7380597faec2d05085ea4c4ec635347799c1248b15f6667849b421776e0cec54

\[preflight\] Running pre-flight checks

\[WARNING SystemVerification\]: cgroups v1 support is in maintenance
mode, please migrate to cgroups v2

error execution phase preflight: \[preflight\] Some fatal errors
occurred:

\[ERROR FileAvailable--etc-kubernetes-kubelet.conf\]:
/etc/kubernetes/kubelet.conf already exists

\[ERROR Port-10250\]: Port 10250 is in use

\[ERROR FileAvailable--etc-kubernetes-pki-ca.crt\]:
/etc/kubernetes/pki/ca.crt already exists

\[preflight\] If you know what you are doing, you can make a check
non-fatal with \`--ignore-preflight-errors=...\`

To see the stack trace of this error execute with --v=5 or higher

 

 

 

To install a network plugin, we will go with Flannel as the default
choice. For inter-host communication, we will utilize the eth0 interface
and update the Network field accordingly.

 

Ensure that the Flannel manifest includes the appropriate options for
this configuration.

 

 

For detailed instructions, refer to the official documentation linked in
the upper right corner above the terminal.

 

On the controlplane node, run the following set of commands to deploy
the network plugin:

 

Download the original YAML file and save it as kube-flannel.yml:

curl -LO
<https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml>

Open the kube-flannel.yml file using a text editor.

 

We are using a custom PodCIDR (172.17.0.0/16) instead of the default
10.244.0.0/16 when bootstrapping the Kubernetes cluster. However, the
Flannel manifest by default is configured to use 10.244.0.0/16 as its
network, which does not align with the specified PodCIDR. To resolve
this, we need to update the Network field in the kube-flannel-cfg
ConfigMap to match the custom PodCIDR defined during cluster
initialization.

 

net-conf.json: \|

{

"Network": "10.244.0.0/16", \# Update this to match the custom PodCIDR

"Backend": {

"Type": "vxlan"

}

Locate the args section within the kube-flannel container definition. It
should look like this:

args:

\- --ip-masq

\- --kube-subnet-mgr

Add the additional argument - --iface=eth0 to the existing list of
arguments.

 

Now apply the modified manifest kube-flannel.yml file using kubectl:

 

kubectl apply -f kube-flannel.yml

After applying the manifest, wait for all the pods to become in the
Ready state. You can use the watch command to monitor the pod status:

 

watch kubectl get pods -A

Example of expected pods:

 

controlplane ~ ➜ kubectl get pods -A

NAMESPACE NAME READY STATUS RESTARTS AGE

kube-flannel kube-flannel-ds-gc5kf 1/1 Running 0 54s

kube-flannel kube-flannel-ds-mtjd6 1/1 Running 0 54s

kube-system coredns-668d6bf9bc-7lf7s 1/1 Running 0 3m31s

kube-system coredns-668d6bf9bc-jl8t6 1/1 Running 0 3m31s

kube-system etcd-controlplane 1/1 Running 0 3m37s

kube-system kube-apiserver-controlplane 1/1 Running 0 3m37s

kube-system kube-controller-manager-controlplane 1/1 Running 0 3m37s

kube-system kube-proxy-t5wrt 1/1 Running 0 3m31s

kube-system kube-proxy-trmhs 1/1 Running 0 3m8s

kube-system kube-scheduler-controlplane 1/1 Running 0 3m37s

After all the pods are in the Ready state, the status of both nodes
should now become Ready:

 

controlplane ~ ➜ kubectl get nodes

NAME STATUS ROLES AGE VERSION

controlplane Ready control-plane 15m v1.32.0

node01 Ready \<none\> 15m v1.32

 

Set up kubectl autocomplete in bash
<https://kubernetes.io/pt-br/docs/reference/kubectl/cheatsheet/>

 

--------------------------------------------------------------

 

Flannel CNI plugin and coredns pods issue

 

vagrant@controlplane:~\$ kubectl get pods -A --watch

NAMESPACE NAME READY STATUS RESTARTS AGE

kube-flannel kube-flannel-ds-84vjb 0/1 Error 2 (24s ago) 29s

kube-system coredns-674b8bbfcf-chht7 0/1 ContainerCreating 0 44h

kube-system coredns-674b8bbfcf-h7dqq 0/1 ContainerCreating 0 44h

kube-system etcd-controlplane 1/1 Running 0 44h

kube-system kube-apiserver-controlplane 1/1 Running 0 44h

kube-system kube-controller-manager-controlplane 1/1 Running 2 (12h ago)
44h

kube-system kube-proxy-nl9gx 1/1 Running 0 44h

kube-system kube-scheduler-controlplane 1/1 Running 2 (12h ago) 44h

kube-flannel kube-flannel-ds-84vjb 0/1 CrashLoopBackOff 2 (12s ago) 32s

kube-flannel kube-flannel-ds-84vjb 1/1 Running 3 (26s ago) 46s

kube-flannel kube-flannel-ds-84vjb 0/1 Error 3 (27s ago) 47s

kube-flannel kube-flannel-ds-84vjb 0/1 CrashLoopBackOff 3 (13s ago) 60s

kube-flannel kube-flannel-ds-84vjb 1/1 Running 4 (50s ago) 97s

kube-flannel kube-flannel-ds-84vjb 0/1 Error 4 (52s ago) 99s

 

 

kubectl logs kube-flannel-ds-hmx78 -n kube-flannel

Defaulted container "kube-flannel" out of: kube-flannel,
install-cni-plugin (init), install-cni (init)

I0626 14:55:50.972557 1 main.go:211\] CLI flags config:
{etcdEndpoints:http://127.0.0.1:4001,http://127.0.0.1:2379
etcdPrefix:/coreos.com/network etcdKeyfile: etcdCertfile: etcdCAFile:
etcdUsername: etcdPassword: version:false kubeSubnetMgr:true kubeApiUrl:
kubeAnnotationPrefix:flannel.alpha.coreos.com kubeConfigFile: iface:\[\]
ifaceRegex:\[\] ipMasq:true ifaceCanReach:
subnetFile:/run/flannel/subnet.env publicIP: publicIPv6:
subnetLeaseRenewMargin:60 healthzIP:0.0.0.0 healthzPort:0
iptablesResyncSeconds:5 iptablesForwardRules:true
netConfPath:/etc/kube-flannel/net-conf.json
setNodeNetworkUnavailable:true}

W0626 14:55:50.972707 1 client_config.go:659\] Neither --kubeconfig nor
--master was specified. Using the inClusterConfig. This might not work.

I0626 14:55:51.002503 1 kube.go:469\] Starting kube subnet manager

I0626 14:55:51.003880 1 kube.go:139\] Waiting 10m0s for node controller
to sync

I0626 14:55:52.004515 1 kube.go:146\] Node controller sync successful

I0626 14:55:52.005278 1 main.go:231\] Created subnet manager: Kubernetes
Subnet Manager - controlplane

I0626 14:55:52.005771 1 main.go:234\] Installing signal handlers

I0626 14:55:52.006828 1 main.go:479\] Found network config - Backend
type: vxlan

E0626 14:55:52.007168 1 main.go:268\] Failed to check br_netfilter: stat
/proc/sys/net/bridge/bridge-nf-call-iptables: no such file or directory

 

The error you're seeing in the Flannel logs (Failed to check
br_netfilter: stat /proc/sys/net/bridge/bridge-nf-call-iptables: no such
file or directory) indicates that the br_netfilter kernel module is not
loaded on your node.

 

This module is required for Flannel's VXLAN backend to work properly, as
it enables iptables to see bridged traffic. Without this module, network
connectivity between pods might not work correctly.

 

Here's how to fix this:

 

Load the br_netfilter module:

Run this command on your controlplane node:

 

bash

sudo modprobe br_netfilter

 

Make it persistent across reboots:

 

bash

echo 'br_netfilter' \| sudo tee /etc/modules-load.d/k8s.conf

 

Verify the module is loaded:

 

bash

lsmod \| grep br_netfilter

Check the sysctl settings:

 

These should be set to 1:

 

bash

cat /proc/sys/net/bridge/bridge-nf-call-iptables

cat /proc/sys/net/bridge/bridge-nf-call-ip6tables

 

If they're not set to 1, you can set them with:

 

bash

sudo sysctl -w net.bridge.bridge-nf-call-iptables=1

sudo sysctl -w net.bridge.bridge-nf-call-ip6tables=1

 

Make the sysctl settings persistent:

Add these lines to /etc/sysctl.conf or create a new file in
/etc/sysctl.d/:

 

text

net.bridge.bridge-nf-call-iptables = 1

net.bridge.bridge-nf-call-ip6tables = 1

 

 

Use the following to add configs

 

cat \<\<EOF \| sudo tee /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-ip6tables = 1

net.bridge.bridge-nf-call-iptables = 1

EOF

 

 

After making these changes, the Flannel pod should automatically
recover, but if it doesn't, you might need to restart it:

 

bash

kubectl delete pod -n kube-flannel kube-flannel-ds-hmx78

 

This is a common requirement for Kubernetes networking and should be set
up on all nodes in your cluster.

 

-----------------------------------------------

 

 

set net.bridge.bridge-nf-call-iptables to 1:

 

cat \<\<EOF \| sudo tee /etc/modules-load.d/k8s.conf

br_netfilter

EOF

 

cat \<\<EOF \| sudo tee /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-ip6tables = 1

net.bridge.bridge-nf-call-iptables = 1

EOF

 

sudo sysctl --system

 

 

kubeadm join 192.168.56.11:6443 --token 5xf60f.9cs2ig2lm2snvx7c \\

--discovery-token-ca-cert-hash
sha256:5d071e1fc348a1cc9ccdef728fae2b387e99e2649de3ac100fb6fac0ee7cc9de

 

--------------------------------------------------------------------

 

kubeadm join error

 

agrant@node02:~\$ sudo kubeadm join 192.168.56.11:6443 --token
5xf60f.9cs2ig2lm2snvx7c \\

--discovery-token-ca-cert-hash
sha256:5d071e1fc348a1cc9ccdef728fae2b387e99e2649de3ac100fb6fac0ee7cc9de

\[preflight\] Running pre-flight checks

error execution phase preflight: couldn't validate the identity of the
API Server: could not find a JWS signature in the cluster-info ConfigMap
for token ID "5xf60f"

To see the stack trace of this error execute with --v=5 or higher

 

The error you're encountering during kubeadm join suggests that the
token you're using (5xf60f.9cs2ig2lm2snvx7c) is either invalid or has
expired. Here's why this happens and how to fix it:

 

Cause of the Error:

Expired Token – Kubernetes bootstrap tokens are valid for only 24 hours
by default.

 

Invalid Token – The token might have been mistyped or revoked.

 

Cluster-Info ConfigMap Issue – The cluster-info ConfigMap in the
kube-public namespace might be corrupted or missing the JWS (JSON Web
Signature) for the token.

 

 

Solution:

1\. Generate a New Token (Recommended)

Run this on the control plane node:

 

bash

kubeadm token create --print-join-command

This will output a new kubeadm join command with a fresh token. Use this
to join the worker node.

 

2\. Verify the Token (Optional)

 

Check existing tokens:

 

bash

kubeadm token list

If your token (5xf60f) is listed but still fails, it might be invalid.

 

3\. Manually Check the cluster-info ConfigMap

On the control plane node, verify the cluster-info ConfigMap:

 

bash

kubectl get cm -n kube-public cluster-info -o yaml

If it's missing or corrupted, you may need to re-upload it:

 

bash

kubeadm init phase upload-certs --upload-certs

4\. Re-Generate CA Cert Hash (If Needed)

If the token is correct but still failing, get a fresh
discovery-token-ca-cert-hash:

 

bash

openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt \| openssl rsa
-pubin -outform der 2\>/dev/null \| openssl dgst -sha256 -hex \| sed
's/^.\* //'

Then use this hash in your kubeadm join command.

 

Final Working Command (Example)

After generating a new token, your join command should look like:

 

bash

sudo kubeadm join 192.168.56.11:6443 --token NEW_TOKEN_HERE
--discovery-token-ca-cert-hash sha256:NEW_HASH_HERE

 

Additional Checks:

Ensure network connectivity between the worker node and the control
plane (ping 192.168.56.11).

 

Verify that ports 6443 (API server) and 10250 (kubelet) are open.

 

If using a firewall (like ufw), allow necessary traffic:

 

bash

sudo ufw allow 6443/tcp

sudo ufw allow 10250/tcp

 

 

vagrant@controlplane:~\$ kubeadm token create --print-join-command

kubeadm join 192.168.56.11:6443 --token f3vxmn.ext263v8hshqh4xh
--discovery-token-ca-cert-hash
sha256:5d071e1fc348a1cc9ccdef728fae2b387e99e2649de3ac100fb6fac0ee7cc9de

 

 

-------------------------------------------------

 

CrashLoopBackOff state on flannel pod on worker nodes

 

 

vagrant@controlplane:~\$ kubectl get pods -A

NAMESPACE NAME READY STATUS RESTARTS AGE

kube-flannel kube-flannel-ds-5lgj4 0/1 CrashLoopBackOff 160 (4m23s ago)
3d10h

kube-flannel kube-flannel-ds-cxnvk 0/1 CrashLoopBackOff 160 (4m38s ago)
3d10h

kube-flannel kube-flannel-ds-fgks2 1/1 Running 0 3d10h

kube-system coredns-674b8bbfcf-chht7 1/1 Running 0 5d7h

kube-system coredns-674b8bbfcf-h7dqq 1/1 Running 0 5d7h

kube-system etcd-controlplane 1/1 Running 0 5d7h

kube-system kube-apiserver-controlplane 1/1 Running 0 5d7h

kube-system kube-controller-manager-controlplane 1/1 Running 6 (20m ago)
5d7h

kube-system kube-proxy-j4vtd 1/1 Running 0 3d10h

kube-system kube-proxy-nl9gx 1/1 Running 0 5d7h

kube-system kube-proxy-pqgvc 1/1 Running 0 3d10h

kube-system kube-scheduler-controlplane 1/1 Running 7 (20m ago) 5d7h

 

 

The error in your Flannel pod (Failed to check br_netfilter: stat
/proc/sys/net/bridge/bridge-nf-call-iptables: no such file or directory)
occurs because the br_netfilter kernel module is not loaded on the
worker node (node02). This module is required for Kubernetes networking
(especially CNI plugins like Flannel) to function correctly.

 

Why This Happens

 

br_netfilter is missing – The kernel module is needed for bridged
traffic to be processed by iptables, which is essential for pod-to-pod
communication.

 

Flannel relies on this – Flannel's vxlan backend requires br_netfilter
to enforce network policies and NAT rules.

 

How to Fix It

 

1\. Load the br_netfilter Kernel Module

 

Run this on the worker node (node02):

 

bash

sudo modprobe br_netfilter

 

Verify it's loaded:

 

bash

lsmod \| grep br_netfilter

(Should show br_netfilter in the output.)

 

2\. Enable bridge-nf-call-iptables

 

Ensure these sysctl settings are set to 1:

 

bash

sudo sysctl -w net.bridge.bridge-nf-call-iptables=1

sudo sysctl -w net.bridge.bridge-nf-call-ip6tables=1

 

3\. Make Changes Persistent (Survive Reboot)

 

Add the module and sysctl settings permanently:

 

bash

\# Load br_netfilter at boot

 

echo "br_netfilter" \| sudo tee /etc/modules-load.d/k8s.conf

 

\# Set sysctl rules permanently

 

echo "net.bridge.bridge-nf-call-iptables = 1" \| sudo tee -a
/etc/sysctl.d/k8s.conf

echo "net.bridge.bridge-nf-call-ip6tables = 1" \| sudo tee -a
/etc/sysctl.d/k8s.conf

 

\# Apply changes

 

sudo sysctl --system

 

4\. Restart the Flannel Pod (Optional)

 

If Flannel still fails, restart it:

 

bash

kubectl delete pod -n kube-flannel kube-flannel-ds-cxnvk

 

(It will automatically restart with the correct settings.)

 

Verify the Fix

 

Check Flannel logs again:

 

bash

kubectl logs -n kube-flannel kube-flannel-ds-cxnvk

The br_netfilter error should be gone.

 

Test pod networking:

 

bash

kubectl get pods -A -o wide

Ensure pods on node02 are running and can communicate with others.

 

Additional Checks

Ensure iptables is available (some minimal Linux distros lack it):

 

bash

which iptables

If using firewalls (like ufw), allow Kubernetes traffic:

 

bash

sudo ufw allow 6443/tcp \# API Server

sudo ufw allow 8472/udp \# Flannel VXLAN

Conclusion

The error occurs because the worker node lacks br_netfilter support.
Loading the module and enabling bridge-nf-call-iptables resolves it.
After applying these changes, Flannel should work correctly, and pod
networking will function as expected.
