# Kubernetes Installation on Ubuntu 20.04

Get the detailed information about the installation from the below-mentioned websites of **Docker** and **Kubernetes**.

[Docker](https://docs.docker.com/)

[Kubernetes](https://kubernetes.io/)

### Set up the Docker and Kubernetes repositories:

> Download the GPG key for docker

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

> Add the docker repository

```bash
# Docker has release the repository for Ubuntu 20.04 Focal LTS.
# we can get the latest release versions from https://docs.docker.com

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

```

> Add the GPG key for kubernetes

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

> Add the kubernetes repository

**Check for the latest release in https://packages.cloud.google.com/apt/dists**

```bash
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

> Update the repository

```bash
# Update the repositiries
sudo apt-get update
```

> Install Docker and Kubernetes packages.

**Note that if you want to use a newer version of Kubernetes, change the version installed for kubelet, kubeadm, and kubectl and be sure that all three use the same version.
These version should support the Docker CE version.**

```bash
# Use the same versions to avoid issues with the installation.
sudo apt-get install -y docker-ce=5:20.10.14~3-0~ubuntu-$(lsb_release -cs) kubelet=1.23.6-00 kubeadm=1.23.6-00 kubectl=1.23.6-00
```


> To hold the versions so that the versions will not get accidently upgraded.

```bash
sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```

> Enable the iptables bridge

```bash
#Set a value in the sysctl file , to allow proper network settings for Kubernetes on all the servers.
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf

#To make the changes to take immediate effect for the iptables
sudo sysctl -p
```

### Disable swap

**Check if Swap is enabled**

```
sudo swapon --show
```

If swap is enabled you should see the path to the swap file and its size.

```
$ sudo swapon --show
NAME      TYPE SIZE USED PRIO
/swapfile file   2G   0B   -2

```
**Disable Swap**

Run the following command to disable Swap:

```
sudo swapoff -a
```

Now remove the Swap file:

```
sudo rm /swap.img
```

The next thing we need to do is modify the fstab file so that the Swap file is not re-created after a system reboot.

Remove following line from /etc/fstab

```
/swap.img       none    swap    sw      0       0
```

**Check Swap is disabled**

Run the following command to check Swap is disabled.

```
sudo swapon --show
```

There should be no output if it’s disabled.

### On the Kube master server

> Initialize the cluster by passing the cidr value and the value will depend on the type of network CLI you choose.

**Use either Flannel or Calico**

```bash
# For flannel network
# Copy your join command and keep it safe.
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Copy your join command and keep it safe.
# Below is a sample
kubeadm join 10.128.0.2:6443 --token swi0ci.jq9l75eg8lvpxz6g --discovery-token-ca-cert-hash sha256:2c3cdfa898334b0dfc0f73bbccb998d03f61252ee50f0405c85ba735ff90b4d1
```

```bash
# For Calico network
# Make sure to copy the join command
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

# Copy your join command and keep it safe.
# Below is a sample
kubeadm join 10.128.0.2:6443 --token swi0ci.jq9l75eg8lvpxz6g --discovery-token-ca-cert-hash sha256:2c3cdfa898334b0dfc0f73bbccb998d03f61252ee50f0405c85ba735ff90b4d1
```

> To start using the cluster with current user.

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

> Set the flannel networking

```bash
# Use this if you have initialised the cluster with Flannel network add on.
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

> To set up the Calico network

```bash
# Use this if you have initialised the cluster with Calico network add on.
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

> Check the nodes

```bash
# Check the status on the master node.
kubectl get nodes
```

### On each of Kube node server

> Joining the node to the cluster:

```bash
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash
```

**TIP**

> If the joining code is lost, it can retrieve using below command

```bash
kubeadm token create --print-join-command
```
