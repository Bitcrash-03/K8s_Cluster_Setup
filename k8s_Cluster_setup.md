
# Kubernetes Master Node Setup with Cilium

This script sets up a Kubernetes master node with Cilium as the CNI.

## Steps

1. **Update and Upgrade System**
   ```bash
   sudo apt-get update && sudo apt-get upgrade -y
   ```

2. **Disable Swap**
   ```bash
   sudo swapoff -a
   sudo sed -i '/ swap / s/^\(.*\)$/#/g' /etc/fstab
   ```

3. **Load Kernel Modules and Set Sysctl Configurations**
   ```bash
   sudo tee /etc/modules-load.d/containerd.conf <<EOF
   overlay
   br_netfilter
   EOF
   sudo modprobe overlay
   sudo modprobe br_netfilter

   sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   net.ipv4.ip_forward = 1
   EOF
   sudo sysctl --system
   ```

4. **Install Containerd**
   ```bash
   sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   sudo apt update
   sudo apt install -y containerd.io
   containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
   sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
   sudo systemctl restart containerd
   sudo systemctl enable containerd
   ```

5. **Install Kubernetes Packages**
   ```bash
   sudo apt-get update
   sudo apt-get install -y apt-transport-https ca-certificates curl gpg
   curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
   sudo apt-get update
   sudo apt-get install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

6. **Initialize Kubernetes Cluster**
   ```bash
   sudo kubeadm init --pod-network-cidr=10.0.0.0/16
   ```

7. **Set Up `kubectl` Configuration**
   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

8. **Install Cilium CNI**
   ```bash
   curl -LO https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz
   sudo tar xzvf cilium-linux-amd64.tar.gz -C /usr/local/bin
   rm cilium-linux-amd64.tar.gz
   cilium install
   ```

---

# Kubernetes Worker Node Setup with Cilium

This script sets up a Kubernetes worker node and joins it to the cluster with Cilium as the CNI.

## Steps

1. **Update and Upgrade System**
   ```bash
   sudo apt-get update && sudo apt-get upgrade -y
   ```

2. **Disable Swap**
   ```bash
   sudo swapoff -a
   sudo sed -i '/ swap / s/^\(.*\)$/#/g' /etc/fstab
   ```

3. **Load Kernel Modules and Set Sysctl Configurations**
   ```bash
   sudo tee /etc/modules-load.d/containerd.conf <<EOF
   overlay
   br_netfilter
   EOF
   sudo modprobe overlay
   sudo modprobe br_netfilter

   sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   net.ipv4.ip_forward = 1
   EOF
   sudo sysctl --system
   ```

4. **Install Containerd**
   ```bash
   sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   sudo apt update
   sudo apt install -y containerd.io
   containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
   sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
   sudo systemctl restart containerd
   sudo systemctl enable containerd
   ```

5. **Install Kubernetes Packages**
   ```bash
   sudo apt-get update
   sudo apt-get install -y apt-transport-https ca-certificates curl gpg
   curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
   sudo apt-get update
   sudo apt-get install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

6. **Join Kubernetes Cluster** (Replace with actual command)
   ```bash
   kubeadm join 10.207.26.185:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef
   ```
