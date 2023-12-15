# Multi Master K8s cluster

## Pre-requisite
we will need below instances to create multi master k8s cluster

* 3 Ubuntu VMs ( acts as master node )
* 3 Ubuntu VMs ( acts as worker node )
* 1 LoadBalancer acts as HA between masters and workers

**NOTE** : Here we use Azure LoadBalancer as HA


### 1 Create Azure VMs

* Login to azure portal
* search for Virtual Machines
* Create around 6 VM's with flavour **Ubuntu** (any latest version)
* Allow below ports in the VMs NSG
    - **"6443"**
    - **"22"**
    - **"2379-2380"**   
    - **"10250"**
    - **"10251"**
    - **"10252"**
    - **"30000-32767"**

### 2 Create Azure LoadBalancer

* Now go to azure loadbalancer
* create new loadbalancer
* create a public ip assign it as frontend configuration
* add the master Nodes (NIC) as backend pools
* create LoadBalancer rule and add the port no **6443**

### Install kubeadm,kubelet and docker on master and worker nodes

* Log in to any one master node ( ubuntu VM )
* switch to sudo ```sudo -i```
* update the repositories
```apt-get update```
* Turn off swap
```
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
* Install container runtime - docker

> [!NOTE]
> Follow Official docker site for installing docker runtime

* Install **kubeadm** and **kubelet**
```sh
apt-get update && apt-get install -y apt-transport-https curl ca-certificates gnupg-agent software-properties-common

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update

apt-get install -y kubelet kubeadm
apt-mark hold kubelet kubeadm 
systemctl daemon-reload 
systemctl start kubelet 
systemctl enable kubelet.service
```
> [!NOTE]
> Follow the above steps for all VMs (nodes)


.


### Set up kubeadm for cluster initialization

Follow the below steps on any 1 master node

### master-1

* Log in to master1
* Switch to root account - sudo -i
* execute below command
```sh
kubeadm init --control-plane-endpoint "<LOAD_BALANCER_IP>:6443" --upload-certs
```
**NOTE**: use the loadbalancer frontend ip address here

* Output should look alike below:
```sh
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join <LOADBALANCER_IP>:6443 --token abc.kasba.... \
    --discovery-token-ca-cert-hash sha256:123abc........ \
    --control-plane --certificate-key 123abc........

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use 
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join <LOADBALANCER_IP>:6443 --token abc.kasba.... \
    --discovery-token-ca-cert-hash sha256:123abc........
```

* The above output contains 3 main points

```sh
1) Setup kubeconfig
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```sh
2) Setup new control plane (master)
 kubeadm join <LOADBALANCER_IP>:6443 --token abc.kasba.... \
    --discovery-token-ca-cert-hash sha256:123abc........ \
    --control-plane --certificate-key 123abc........
```

```sh
3) Join worker node
kubeadm join <LOADBALANCER_IP>:6443 --token abc.kasba.... \
    --discovery-token-ca-cert-hash sha256:123abc........
```

* Run the 1st command to setup the kubeconfig file on the same server.
* Run 2nd command on remaining master nodes 
* Run 3rd command on all the worker nodes

### Setup CNI on the ks cluster

A Container Network Interface (CNI) plugin is required in a Kubernetes cluster to provide networking capabilities to the pods running on the cluster.

* Go to any master node
* run the below command

```sh
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

```

> [!NOTE]
> To execute kubectl commands and retrieve cluster information, you should first download the kubeconfig file to your local environment. Additionally, ensure that kubectl is installed on your system.

## Kubernetes Cluster setup completed
