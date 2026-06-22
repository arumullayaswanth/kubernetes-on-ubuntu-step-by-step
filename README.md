# Kubernetes on Ubuntu - Step by Step Guide 🚀

This guide helps you install Kubernetes on Ubuntu in a very simple way.

## What We Are Going To Build

By the end of this guide, you will have:

✅ Kubernetes installed

✅ One Control Plane Node

✅ kubectl working

✅ Flannel Network installed

✅ Kubernetes cluster running

---

# Step 1: Update Ubuntu

Open Terminal and run:

```bash
sudo apt update
sudo apt upgrade -y
```

Why?

This updates your system packages.

---

# Step 2: Turn Off Swap

Kubernetes does not like swap.

Run:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

Check:

```bash
free -h
```

Swap should show 0.

---

# Step 3: Enable Required Kernel Modules

Create configuration file:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

Load modules:

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

---

# Step 4: Configure Network Settings

Run:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF
```

Apply settings:

```bash
sudo sysctl --system
```

---

# Step 5: Install Containerd

Install containerd:

```bash
sudo apt install -y containerd
```

Create default config:

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Enable systemd cgroup:

```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

Restart containerd:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Check status:

```bash
systemctl status containerd
```

---

# Step 6: Add Kubernetes Repository

Install required packages:

```bash
sudo apt install -y apt-transport-https ca-certificates curl gpg
```

Download Kubernetes key:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add repository:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update packages:

```bash
sudo apt update
```

---

# Step 7: Install Kubernetes

Install:

```bash
sudo apt install -y kubelet kubeadm kubectl
```

Prevent accidental upgrades:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

Verify:

```bash
kubectl version --client
```

---

# Step 8: Create Kubernetes Cluster

Initialize cluster:

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

This may take a few minutes.

Save the join command shown at the end.

---

# Step 9: Configure kubectl

Run:

```bash
mkdir -p $HOME/.kube
```

```bash
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

```bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Test:

```bash
kubectl get nodes
```

---

# Step 10: Install Flannel Network

Run:

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Wait a minute.

Check:

```bash
kubectl get pods -A
```

---

# Step 11: Verify Cluster

Check node:

```bash
kubectl get nodes
```

Expected:

```text
NAME       STATUS   ROLES           AGE
ubuntu     Ready    control-plane   xxm
```

---

# Useful Commands

See nodes:

```bash
kubectl get nodes
```

See all pods:

```bash
kubectl get pods -A
```

Cluster information:

```bash
kubectl cluster-info
```

See namespaces:

```bash
kubectl get ns
```

---

# Troubleshooting

Check kubelet:

```bash
sudo systemctl status kubelet
```

Check containerd:

```bash
sudo systemctl status containerd
```

View logs:

```bash
journalctl -xeu kubelet
```

---

# Success 🎉

Congratulations!

You have successfully installed Kubernetes on Ubuntu and created your first Kubernetes cluster.

Now you can start learning:

- Pods
- Deployments
- Services
- ConfigMaps
- Secrets
- Ingress
- Helm

Happy Learning 🚀
