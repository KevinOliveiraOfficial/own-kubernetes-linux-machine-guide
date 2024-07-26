# Create your own kubernetes machine without docker (own-linux-kubernetes-machine-guide)
Guide to create your own kubernetes machine without docker using linux, kubeadm, containerd, runc and CNI Plugins.
Only to create simulations for official in-production kubernetes like AWS, Google Cloud Platform, Azure.

## Step 1: UPDATE YOUR SYSTEM
```
sudo yum update -y
```
## Step 2: INSTALL REQUIRED PACKAGES
```
sudo yum install -y yum-utils
```
##  Step 3: OPEN REQUIRED PORTS
- If the machine is a Master node
```
sudo firewall-cmd --zone=public --add-port=6443/tcp --permanent
sudo firewall-cmd --zone=public --add-port=2379-2380/tcp --permanent
sudo firewall-cmd --zone=public --add-port=10250/tcp --permanent
sudo firewall-cmd --zone=public --add-port=10251/tcp --permanent
sudo firewall-cmd --zone=public --add-port=10252/tcp --permanent
```

- If the machine is a Worker node
```
sudo firewall-cmd --zone=public --add-port=10250/tcp --permanent
sudo firewall-cmd --zone=public --add-port=30000-32767/tcp --permanent
```

- Calico pods network
```
sudo firewall-cmd --zone=public --add-port=8285/udp --permanent
```

- Reload firewall
```
sudo firewall-cmd --reload
```

##  Step 4: CONTAINER RUNTIMES - INSTALL AND CONFIGURE PREREQUISITES
### Enable IPv4 packet forwarding
- sysctl params required by setup, params persist across reboots
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
```
- Apply sysctl params without reboot
sudo sysctl --system

## Step 5: INSTALLING CONTAINERD
### Installing containerd
```
wget https://github.com/containerd/containerd/releases/download/v1.7.20/containerd-1.7.20-linux-amd64.tar.gz
sudo tar -C /usr/local -xvzf containerd-1.7.20-linux-amd64.tar.gz
sudo mkdir -p /usr/local/lib/systemd/system/
sudo curl -o /usr/local/lib/systemd/system/containerd.service https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
systemctl daemon-reload
systemctl enable --now containerd
```
### Installing runc
```
wget https://github.com/opencontainers/runc/releases/download/v1.1.13/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```
### Installing CNI plugins
```
wget https://github.com/containernetworking/plugins/releases/download/v1.5.1/cni-plugins-linux-amd64-v1.5.1.tgz
mkdir -p /opt/cni/bin
sudo tar -C /opt/cni/bin -xvzf cni-plugins-linux-amd64-v1.5.1.tgz
```

### Generate default containerd configuration
```
sudo /usr/local/bin/containerd config default | sudo tee /etc/containerd/config.toml
```

## Configuring the systemd cgroup driver
- To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set:
```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```

## Restart containerd
```
sudo systemctl restart containerd
```

## STEP 6: INSTALLING KUBEADM, KUBELET AND KUBECTL
### Set SELinux in permissive mode (effectively disabling it)
```
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

- Add the Kubernetes yum repository
This overwrites any existing configuration in /etc/yum.repos.d/kubernetes.repo
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```

### Install kubelet, kubeadm and kubectl:
```
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

### (Optional) Enable the kubelet service before running kubeadm:
```
sudo systemctl enable --now kubelet
```

## STEP 7: DISABLE SWAP PERMAMENT
- comment swap line
```
sudo vim /etc/fstab
```

## STEP 8: Start cluster
```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

## STEP 9: Execute the command that appears after start cluster
```
echo "Execute the command that appears after start cluster"
```

## STEP 10: Install pods network
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

## Check node status
```
kubectl get nodes
```
