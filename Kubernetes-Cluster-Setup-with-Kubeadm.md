
# Kubernetes Cluster Setup with Kubeadm

# Prerequisites

Before you begin, ensure that you meet the following prerequisites:

    ● Operating System: Ubuntu 20.04 or later
    ● User Privileges: Root or a user with sudo privileges
    ● Network Configuration: Ensure all nodes can communicate with each other over the network
    ● Firewall: Open the necessary ports (6443, 2379-2380, 10250, 10251,10252, 10255)

# [ BOTH MASTER AND WORKER NODE ]
# OS Configuration





## Step 1: Disable Swap

Disable swap on all nodes:
``` bash
  $ sudo apt update
  $ sudo swapoff -a
```

## Step 2: Enable the br_netfilter Kernel Module

Enable the br_netfilter module on all nodes:

```bash
  $ sudo modprobe br_netfilter
```
Make it persist after reboots by including it in your system's modules-load
list:
```bash
  $ echo br_netfilter | sudo tee /etc/modules-load.d/kubernetes.conf
```
# Installing Kubernetes

The modern Kubernetes installation experience is largely automated by
kubeadm, the official cluster setup tool. Just follow these steps:

1. Install the containerd container runtime on each of your nodes.

2. Download and install kubeadm, kubelet, and kubectl on your master node.

3. Use kubeadm to initialize the Kubernetes control plane on your master node.

4. Download and install kubeadm and kubelet on your worker nodes.

5. Use kubeadm to start the kubelet agent process on each worker node to connect the node to your cluster.

## Step 1: Installing containerd
#### Adding Dependencies

Update the package list and install required packages on all nodes:

``` bash
  $ sudo apt-get update
  $ sudo apt-get install -y ca-certificates curl
  $ sudo install -m 0755 -d /etc/apt/keyrings
  $ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
  $ sudo chmod a+r /etc/apt/keyrings/docker.asc
```
Add Docker's repository to Apt sources:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```bash
 $ sudo apt-get update
 ```
#### Install containerd
Install containerd on all nodes:
```bash
 $ sudo apt install -y containerd.io
 $ sudo service containerd status
```
#### Configuring containerd
Generate the default containerd configuration and save it to "/etc/containerd/config.toml" :

```bash
 $ containerd config default | sudo tee /etc/containerd/config.toml
```
Edit the configuration file and set "SystemdCgroup" to "true" :
```bash
 $ sudo nano /etc/containerd/config.toml
 # Find and change the line:
 $ SystemdCgroup = true
 ```
Restart containerd:
``` bash
 $ sudo service containerd restart
 ```
#### Enable IP forwarding
Edit "/etc/sysctl.conf" to enable IP forwarding on all nodes:
```bash
 $ sudo nano /etc/sysctl.conf

 # Add the following line:
   net.ipv4.ip_forward = 1
  ```
Apply the changes:
``` bash
sudo sysctl -p
```
## Step 2: Installing kubeadm, kubectl, and kubelet
#### Add the Kubernetes Signing Key and Repository
Download the signing key to the Apt keyring directory:
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
Add the Kubernetes repository:
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Update the package index and install kubelet, kubeadm, and kubectl :
```bash
 $ sudo apt-get update
 $ sudo apt-get install -y kubelet kubeadm kubectl
 $ sudo apt-mark hold kubelet kubeadm kubectl
 ```
# [ MASTER NODE ]
### Setting Up the Kubernetes Master Node
Initialize the master node with a pod network CIDR:
```bash
 $ sudo kubeadm init --pod-network-cidr=10.244.0.0/16
 ```
 ### Configuring kubectl
 Set up the kubeconfig file for the kubectl command:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### Installing a Networking Plugin
Install Flannel as the networking plugin:
```bash
 $ kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```
# [ WORKER NODE ]
### Adding Worker Nodes
Use the "kubeadm join" command provided by the "kubeadm init" output on each worker node to connect to the cluster:
```bash
sudo kubeadm join <master-node-ip>:6443 --token <token> --discovery-token-ca-cert-hash <hash> --v=5

```
For example:
```bash
sudo kubeadm join 172.31.53.181:6443 --token 03yamu.g6v1gj6yu6xglr5a --discovery-token-ca-cert-hash sha256:3c9d2a4ab21fbb15c938b9644730cd2da3a6402cc9979068020f2f966fcf396b --v=5
```
## Conclusion
You have successfully set up a Kubernetes cluster using kubeadm. You can now manage your cluster using kubectl and deploy applications on it.

For more detailed instructions, refer to the official Kubernetes documentation.



