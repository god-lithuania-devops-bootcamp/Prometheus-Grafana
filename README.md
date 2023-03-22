# Monitoring Stack using Prometheus and Grafana (Swarm and k8s)

## Vagrant VM Access and Kubernetes Cluster Configuration
Instructions for accessing Vagrant VMs externally from my home LAN and setting up a 3-node Kubernetes cluster using Vagrant VM environment. Additionally, it includes a resource for configuring a local kubectl to access a remote Kubernetes cluster.

## Accessing Vagrant VMs Externally from Home LAN
There are two methods to access Vagrant VMs externally from your home LAN:

### 1. Create a Bridge Interface
Create a bridge interface (br0) using Netplan YAML files located in `/etc/netplan/...` on your Vagrant host.

Modify your Vagrantfile network configuration as follows:

Replace:
```sh
config.vm.network :private_network
```
With:
```sh
node.vm.network "public_network", bridge: "br0"
```
Destroy existing Vagrant VMs and recreate them with the modified Vagrantfile:
```sh
vagrant destroy -f
vagrant up
```
**Note**: This method is destructive and requires recreating all Vagrant VMs with the modified Vagrantfile.

### 2. Create a NAT Rule (Non-destructive)
Enable IP forwarding by editing `/etc/sysctl.conf`:
```sh
net.ipv4.ip_forward=1
```
Apply the changes:
```sh
sudo sysctl -p
```
Update packages and install `iptables-persistent`:
```sh
sudo apt-get update
sudo apt-get install -y iptables-persistent
```
Create a NAT rule:
```sh
sudo iptables -t nat -A POSTROUTING -s 192.168.56.0/24 -o your_interface_name -j MASQUERADE
```
Save the `iptables` configuration:
```sh
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```
Configure the VMs on the VirtualBox private network `192.168.56.0/24` to use the Ubuntu 22.04 machine as their default gateway by modifying the `/etc/netplan/xx-netcfg.yaml` file on each VM:

Replace "192.168.56.x" with the IP address of the VM:
```yaml
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: no
      addresses: [192.168.56.x/24]
      gateway4: 192.168.56.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

## Bootstrap a 3-Node Kubernetes Cluster using Vagrant VM Environment
### 1- Setting up the Vagrant and Vitualbox environments
```sh
sudo apt update
sudo apt install vagrant virtualbox -y
```
**Note**: Use the CKA course vagrant file to spin up required 3 machines
```sh
vagrant status
vagrant up
vagrant ssh kubemaster
vagrant halt
```
### 2- Prepare container runtime and systemd drivers on cluster nodes
- Start here: [Installing kubeadm | Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- Container runtime requirements: [Container Runtimes | Kubernetes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

-- Forwarding IPv4 and letting iptables see bridge traffic  -->  Apply all commands shown here on all nodes
-- click on the "containerd" section
   > open the "getting started with containerd" link which goes to it's GitHub page
   > - install containerd using apt. it's using the docker page as source. just be careful to install ONLY "containerd.io" package.
   > - systemctl status containerd  --> is it "active" & "running" with no error logs !?
   
   > read through Cgroup drivers section. note that both kubelet and container runtime need to interface with Cgroups to perform various resource management and constraint tasks.
   > "cgroupfs" is the default driver, however if you use a systemd system, you have to use the "systemd" driver. Also note that both container runtime and kubelet should use the same driver on the node.
     >- ps -p 1   
// gonna show systemd, if this system uses systemd 
     >- "containerd" section  -->  Configure the systemd cgroup driver  --> apply the proposed changes (sudo vim /etc/containerd/config.toml). delete all existing config and paste the new config. Note that you gotta do this for all nodes.
     >- sudo systemctl restart containerd

### 3- Install and prepare Kubeadm
-- go to "Install kubeadm, kubelet and kubectl" section
>- follow instrictions to install these apps on all nodes

-- go to the "What's next" section down at the bottom of the page. (Create a cluster page)
>- read through "Initializing your control-plane node"
>- on master node: sudo kubeadm init --pod-network-cidr=10.20.0.0/16 >-apiserver-advertise-address=192.168.56.2
>- make sure to run the commands instructed at the end of the kubeadm initiation process AND copy and save the join command somewhere safe.

### 4- Install CNI for the cluster
-- It's now time to install CNI: follow the link provided in kubeadm output to install Addons
>- Install Weavenet: just one command and it will install Weavenet Daemonset in kube-system ns
>- k get ds -A    
// we now have to modify the weavent daemonset and introduce the pod-cidr to weavenet
>- k edit ds weavenet -n kube-system    
// instructions are available in Weavenet installation page  -->  Changing Configuration Options

-- Join the other nodes to the cluster

## Configure Local kubectl to Access Remote Kubernetes Cluster
For a detailed guide on how to configure your local kubectl to access a remote Kubernetes cluster, refer to the following article on DEV Community:

[Configure local kubectl to access remote Kubernetes cluster](https://dev.to/plutov/configure-local-kubectl-to-access-remote-kubernetes-cluster-2h2o)
