# Under construction

# Configure Kubernetes With Kubeadm

## I'll be demonstrating with three CentOS 7 servers (at the following IP addresses)
```
192.168.33.10 Master
192.168.33.11 worker1
192.168.33.12 worker2
```
### In case you don’t have your own dns server then update /etc/hosts file on master and worker nodes
```
192.168.33.10 Master
192.168.33.11 worker1
192.168.33.12 worker2
```
### Before you start setting up Kubernetes cluster, it is recommended that you update your system to ensure all security updates are up-to-date
```sh
$sudo yum update -y
```
## `Note`: 
### Ensure swap is disabled on both master and worker nodes. Kubernetes requires swap to be disabled in order for it to successfully configure Kubernetes Cluster
### command to disable swap
```sh
$sudo swapoff -a
```
### check swap is disabled
```sh
$cat /proc/swaps
```
### command to update fstab so that swap remains disabled after a reboot
```sh
$sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
### In order for Kubernetes cluster to communicate internally, we have to disable `SELinux`
### Disable SELinux
```sh
$sudo setenforce 0
```
###To check SELinux:
```sh
$ sestatus
```
### If SELinux still enabled run add SELINUX=disabled to below file and comment SELINUX=enforcing
```sh
$sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```
## Install kubeadmn, kubectl, kubectl on all three nodes
```sh
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```
```sh
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```
### 3.  Initialize kubernets cluster on Master node(Run this only on master)
```sh
sudo kubeadm init
```
## 4. Do following setup to start using kubernetes cluster(Run this only on master)
```sh
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```
  ### 5. Add pod network add-ons (Run this only on master)
  ```sh
  kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
  ```
  ### 6. Add Pod Network add-ons by Calico
      1. Install Calico with Kubernetes API datastore
      2. Download the Calico networking manifest for the Kubernetes API datastore
  ```sh
  curl https://docs.projectcalico.org/manifests/calico-typha.yaml -o calico.yaml
  kubectl apply -f calico.yaml
  ```
  ### Install Calico with etcd datastore.
  ```sh
  curl https://docs.projectcalico.org/manifests/calico-etcd.yaml -o calico.yaml
  kubectl apply -f calico.yaml
  ```
  ## 7.Take note of kubeadm command and run on all workers
  ```sh
  Run this command(replace with your command) on every node to join the cluster

  sudo kubeadm join 172.31.44.226:6443 --token f6o4a3.jdagdd4e2h8xhzy1 \
    --discovery-token-ca-cert-hash sha256:d0528baca6a2cf15bfece995d7df6f5d018b233b54251716ce2fd984148ba6d6
 ```
 ## 8. Get list of nodes in the cluster (Run this on master)
 ```sh
 kubectly get nodes
 ```
  
  
