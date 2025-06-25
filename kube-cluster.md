Here is a **step-by-step guide** to install a Kubernetes cluster using **Kubeadm** based on your provided file. This includes instructions for **Master Node** and **Worker Nodes** separately.

---

## ✅ **Pre-Requisites**

* Use **Ubuntu EC2 Instances (t2.medium or higher)** with **internet access**
* All nodes should be in the **same Security Group**
* Open ports: `6443`, `2379-2380`, `10250`, `10251`, `10252`, `10255`, and `179`

---

## 🔁 **STEP 1: Setup (Run on Both Master and Worker Nodes)**

### 1️⃣ Disable Swap

```bash
sudo swapoff -a
```

### 2️⃣ Load Kernel Modules

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

### 3️⃣ Configure Sysctl Parameters

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

---

## 🔧 **STEP 2: Install CRI-O Runtime (Both Master and Worker)**

```bash
sudo apt-get update -y
sudo apt-get install -y software-properties-common curl apt-transport-https ca-certificates gpg
```

```bash
sudo curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/Release.key | \
sudo gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/ /" | \
sudo tee /etc/apt/sources.list.d/cri-o.list
```

```bash
sudo apt-get update -y
sudo apt-get install -y cri-o
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable crio --now
sudo systemctl start crio.service
```

---

## 📦 **STEP 3: Install Kubernetes Tools (Both Master and Worker)**

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | \
sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```bash
sudo apt-get update -y
sudo apt-get install -y kubelet="1.29.0-*" kubeadm="1.29.0-*" kubectl="1.29.0-*"
sudo apt-get install -y jq
```

```bash
sudo systemctl enable --now kubelet
sudo systemctl start kubelet
```

---

## 👑 **STEP 4: Initialize Master Node (Only on Master)**

### 1️⃣ Pull Kubernetes Images

```bash
sudo kubeadm config images pull
```

### 2️⃣ Initialize Cluster

```bash
sudo kubeadm init
```

### 3️⃣ Configure kubeconfig for Admin Access

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 🌐 **STEP 5: Install Network Plugin (Only on Master)**

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```

---

## 🔑 **STEP 6: Generate Join Command (Only on Master)**

```bash
kubeadm token create --print-join-command
```

> 📌 **Copy the output** — It will look something like this:

```bash
kubeadm join <MASTER_IP>:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxx
```

---

## 🤖 **STEP 7: Join Worker Nodes**

### 1️⃣ Reset pre-flight if needed

```bash
sudo kubeadm reset pre-flight checks
```

### 2️⃣ Join cluster (use the token from master)

```bash
sudo kubeadm join <MASTER_IP>:6443 --token <your-token> --discovery-token-ca-cert-hash sha256:<hash> --v=5
```

---

## ✅ **STEP 8: Verify Cluster (On Master Node)**

```bash
kubectl get nodes
```

> You should see your **Master and Worker Nodes** in `Ready` state.

---

### 🔚 Your Kubeadm Kubernetes cluster is ready!

