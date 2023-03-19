# Vagrant VM Access and Kubernetes Cluster Configuration
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
_TODO_: Add instructions for setting up a 3-node Kubernetes cluster using the kubeadm way in a Vagrant VM environment.

## Configure Local kubectl to Access Remote Kubernetes Cluster
For a detailed guide on how to configure your local kubectl to access a remote Kubernetes cluster, refer to the following article on DEV Community:

[Configure local kubectl to access remote Kubernetes cluster](https://dev.to/plutov/configure-local-kubectl-to-access-remote-kubernetes-cluster-2h2o)
