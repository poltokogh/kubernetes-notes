![[installation.kube-install.png]]
# Cluster Installation and Configuration 

## Preparation

### Section 1: First Move
> [!note]
> Sediakan  3 server/node yang terdiri dari dan setting semua server tersebut
> | node-master | node-worker-01 | node-worker-02 |
#### Hostname
Check hostname  
```
hostname
```

Set Hostname
```
hostnamectl set-hostname k8s-master/k8s-worker-01/k8s-worker-02
```

#### Timezone
Check date and time
```
date
```

Set Timezone
```
timedatectl set-timezone Asia/Jakarta
```

#### Static IP Address
Check IP Address
```
ip a
```

Check netplan status
```
netplan status
```

Check netplan config
```
netplan get
```

Check config directory
```
ls /etc/netplan
```

Config netplan to set static ip address
```
vim /etc/netplan/99-custom-config.yaml
```

99-custom-config.yaml
```
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:
      addresses:
        - 10.10.10.x/24
      routes:
        - to: default
          via: 10.10.10.254
      nameservers:
          addresses: [8.8.8.8, 1.1.1.1]
```

Load netplan config
```
netplan apply
```

#### Update and Upgrade Repository
Update Repository
```
apt update -y
```

Upgrade Package
```
apt upgrade -y
```

### Section 2: Install and Configure Container Runtime (Containerd) 
> [!NOTE]
> Lakukan Install dan Konfigurasi Container Runtime (Containerd) untuk semua Node 
> | node-master | node-worker-01 | node-worker-02 |

Add Docker's official GPG key
```
apt-get update
apt-get install ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
```

Add the repository to Apt sources
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update and install containerd
```
apt-get update
apt-get install containerd.io
```

Load default configuration containerd
```
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Configuring the systemd cgroup driver
```
vim /etc/containerd/config.toml
```

/etc/containerd/config.toml
```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = false
```

menjadi
```
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

atau menggunakan *sed*
```
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```

Restart service containerd
```
systemctl restart containerd
```

### Section 3: Enable and Configure Kernel Module  
> [!NOTE]
> Aktifkan dan konfigurasi untuk semua Node 
> | node-master | node-worker-01 | node-worker-02 |

Enable overlay filesystem and bridge filtering modules
```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

```
sudo modprobe overlay
sudo modprobe br_netfilter
```

Verify modules was enabled
```
lsmod | grep br_netfilter
lsmod | grep overlay
```

Configure necessary kernel modules
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

```
sudo sysctl --system
```

Verify kernel was configured
```
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

### Section 4: Disable Swap Partition 
> [!NOTE]
> Do Disable Swap Partition on All Node 
> | node-master | node-worker-01 | node-worker-02 |

Check swap partition
```
cat /proc/swaps
```

Disable swap partition (temporary)
```
swapoff -a
```

Disable swap partition (permanent)
```
vim /etc/fstab
```

lakukan command untuk swap nya
```
#/swap.img       none    swap    sw      0       0
```

Remount partition
```
mount -a
```

### Section 5: Install Kubeadm V1.34
> [!NOTE]
> Do Install Kubeadm on All Node 
> | node-master | node-worker-01 | node-worker-02 |

Update the apt package index and install packages needed to use the Kubernetes apt repository
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

Download the Google Cloud public signing key  v1.34
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add the Kubernetes apt repository
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Install kubelet, kubeadm and kubectl
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Bootstraping Cluster
### Section 1: Initialize Cluster on Master Node
> [!NOTE]
> Do Initialize Cluster on Master Node 
> | k8s-master |

Create kubeadm config
```
cat <<EOF> kubeadm-config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
---
apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
EOF
```

Download images core kubernetes services
```
kubeadm config images list
kubeadm config images pull
```

Bootstrap the Control Plane
```
kubeadm init --config kubeadm-config.yaml
```

Verify Installation
```
kubectl get node --kubeconfig=/etc/kubernetes/admin.conf
kubectl get pods -n kube-system --kubeconfig=/etc/kubernetes/admin.conf
```

Copy kubeadm-generated kubeconfig
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify Installation
```
kubectl get node
kubectl get pods -n kube-system
```

### Section 2: Join Cluster on Worker Node
> [!NOTE]
> Do Join Cluster on Worker Node 
> | k8s-worker-01/k8s-worker-02 |

Join cluster
```
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```

> [!NOTE]
> Do Verify Cluster on Master Node | k8s-master |

Verify node was joined
```
kubectl get node
```

(Optional) Generate new token for join cluster
```
kubeadm token create --print-join-command
```

## Addons Tools Instalation
### Section 1: Install CNI Plugins
> [!NOTE]
> Do Install CNI Plugins on Master Node | k8s-master |

Install Cilium CLI
```
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
```

Create cilium config
```
cat <<EOF> cilium-config.yaml
ipam:
  mode: "cluster-pool"
  operator:
    clusterPoolIPv4PodCIDRList: ["10.244.0.0/16"]
    clusterPoolIPv4MaskSize: 25
EOF
```

Install Cilium
```
cilium install --version 1.17.5 --values cilium-config.yaml
```

Check cilium status
```
cilium status --wait
```

Cilium check config
```
kubectl get cm -n kube-system cilium-config -o yaml
```

Check Allocation Pod IP on Node
```
kubectl describe node k8s-worker-01 | grep -i PodCIDR
```

### Section 2: Install Metric Server
> [!NOTE]
> Do Install Metric Server on Master Node | k8s-master |

Install metric server
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Edit deployment to enable metric server deployment to use certificate cluster
```
kubectl edit deployment.apps/metrics-server -n kube-system
```

deployment/metrics-server
```
spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls #add this
```

Check pod status
```
kubectl get pod -n kube-system
```

Check resource usage for pod & node
```
kubectl top node
kubectl top pod
```

### Section 3: Install Lens IDE
> [!NOTE]
> Do Install LENS IDE on Local PC [Download LENS IDE](https://k8slens.dev/)

> [!NOTE]
> Do Check Kubeconfig File on Master Node (k8s-master)

Check kubeconfig file
```
cat /etc/kubernetes/admin.conf
```
> [!NOTE]
> Do Copy kubeconfig file to Local PC and Load with LENS IDE

