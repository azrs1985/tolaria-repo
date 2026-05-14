# [K8S] Install K8s Worker

Owner: Nam Tran
Last edited time: December 18, 2025 5:38 PM

```bash
dnf install -y policycoreutils-python-utils bind-utils zip unzip wget net-tools traceroute git tar bash-completion

sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
setenforce 0

systemctl stop firewalld.service
systemctl disable --now firewalld.service

cat <<EOF | tee -a /etc/hosts
#####Kubernetes#####
192.168.2.70   k8s-master
192.168.2.71   k8s-master-01
192.168.2.72   k8s-master-02
192.168.2.73   k8s-master-03
EOF

cat <<EOF | tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

modprobe overlay && modprobe br_netfilter

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
dnf install -y iproute-tc
dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
dnf install -y dnf-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf erase podman buildah
dnf install containerd.io -y

CONTAINDERD_CONFIG_PATH=/etc/containerd/config.toml && rm -f "${CONTAINDERD_CONFIG_PATH}" && containerd config default > "${CONTAINDERD_CONFIG_PATH}" && sed -i "s/SystemdCgroup = false/SystemdCgroup = true/g"  "${CONTAINDERD_CONFIG_PATH}"

systemctl enable --now containerd && systemctl restart containerd

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.25/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.25/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl
EOF

wget https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-amd64-v1.3.0.tgz
tar -xvf cni-plugins-linux-amd64-v1.3.0.tgz -C /opt/cni/bin/
ls /opt/cni/bin/

dnf clean all
dnf install -y kubelet-1.25.2 kubeadm-1.25.2 kubectl-1.25.2 --disableexcludes=kubernetes
systemctl enable kubelet && systemctl start kubelet
dnf install nfs-utils -y
kubeadm join k8s-master:8443 --token bceu3x.zaqs96h2hw3c7ct3 --discovery-token-ca-cert-hash sha256:11cba9b1acb0b7be7f0fb4b3947a12becb73674e42932508574ec9cea81b0bd4
systemctl status kubelet
systemctl restart kubelet

```