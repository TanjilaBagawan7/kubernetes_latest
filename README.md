# Kubernetes Latest Version Setup (1.24)
Follow the below steps to install kubernetes cluster using Containerd as runtime environment in Centos 7 Operating System

## VM Specifications
1.	Create Master and Worker node VMs based on the cluster requirements
2.	Minimum machine configuration would be 2 Core and 2 GB for all the instances
3.	Full network connectivity between all machines in the cluster (public or private network is fine)
4.	Unique hostname, MAC address, and product_uuid for every node

## Preparing the hosts

1.	Update repository packages in all the instances

    `   sudo yum update`
    
2.	Set hostname for master-node on master node instance.

    `   sudo hostnamectl set-hostname master-node`
    
3.	Set hostname for worker-node on worker node instance
    
    `   sudo hostnamectl set-hostname worker-node`
    
4.	Update the /etc/hosts file in all instances

    ~~~    
    <master-node-ip> master-node
    <worker-node-ip> node1 worker-node
    ~~~
    
5.	Disable SELinux in all the instances and reboot machines.

    ~~~
    sudo setenforce 0
    sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
    sudo reboot
    ~~~  

6.	Disable SWAP in all Instances
    ~~~
    sudo sed -i '/swap/d' /etc/fstab
    sudo swapoff -a
    ~~~
    
7.	Update system settings for Kubernetes networking
    ~~~
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    
    sudo modprobe overlay
    sudo modprobe br_netfilter

    cat >>/etc/sysctl.d/kubernetes.conf<<EOF
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    net.ipv4.ip_forward = 1
    EOF
    ~~~

8.	Execute below query for system configuration to take effect
    
    `   sudo sysctl --system`
    
        
##  Installing a container runtime environment

Follow the below steps to configure your Master and Worker node instances. All the below steps need to be executed for all the Master and Worker node instances. 

1.	Download latest container runtime release binary from the below link

    `   https://github.com/containerd/containerd/releases`
   
2.	Execute below command to setup containerd

    ~~~
    yum install -y wget
    cd /opt
    wget https://github.com/containerd/containerd/releases/download/v1.6.5/containerd-1.6.5-linux-amd64.tar.gz
    tar Cxzvf /usr/local /opt/containerd-1.6.5-linux-amd64.tar.gz
    ~~~
    
3.	If you intend to start containerd via system, then need to execute below command
    
    ~~~
    curl https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -o /usr/lib/systemd/system/containerd.service`
    ~~~

4.	Enable containerd to start on boot and start containerd in all Instance

    ~~~
    systemctl daemon-reload
    systemctl enable --now containerd
    systemctl status containerd
    ~~~

5.	Download runc binary from the below link

    `   https://github.com/opencontainers/runc/releases`

6.	Install runc package using below command

    ~~~
    wget https://github.com/opencontainers/runc/releases/download/v1.1.2/runc.amd64
    install -m 755 runc.amd64 /usr/local/sbin/runc
    ~~~
    
7.	Download CNI Plugin binaries from below link
    
    ~~~
    https://github.com/containernetworking/plugins/releases
    ~~~
    
8.	Install CNI plugin using below commands

    ~~~
    wget https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz
    mkdir -p /opt/cni/bin
    tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
    ~~~
    
9.	Create symbolic link for containerd in all the Instances

    ~~~
    ln -s /usr/local/bin/containerd /bin/containerd
    ~~~
    
10.	Execute below commands for containerd configuration changes to take effect

    ~~~
    mkdir -p /etc/containerd
    containerd config default > /etc/containerd/config.toml
    ~~~
    
11.	To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set

    ~~~
    SystemdCgroup = true
    ~~~

12.	Install below conntrack dependency package

    ~~~
    yum install conntrack
    ~~~
    
13.	Restart containerd service in all the Instances

    ~~~
    sudo systemctl restart containerd
    ~~~
    
## Installing kubeadm, kubelet and kubectl

Follow the below steps to install below packages in all of your cluster instances.

1.	Install kubeadm, kubelet and kubectl using below commands

    ~~~
    DOWNLOAD_DIR=/usr/local/bin
    RELEASE="$(curl -sSL https://dl.k8s.io/release/stable.txt)"
    ARCH="amd64"
    cd $DOWNLOAD_DIR
    sudo curl -L --remote-name-all https://storage.googleapis.com/kubernetes-release/release/${RELEASE}/bin/linux/${ARCH}/{kubeadm,kubelet,kubectl}
    sudo chmod +x {kubeadm,kubelet,kubectl}
    ~~~
    
2.	Create symbolic link to use kubeadm, kubelet and kubectl

    ~~~
    ln -s /usr/local/bin/kubeadm /bin/kubeadm
    ln -s /usr/local/bin/kubelet /bin/kubelet
    ln -s /usr/local/bin/kubectl /bin/kubectl
    ~~~
    
3.	Create kubelet service file using below commands    

    ~~~
    RELEASE_VERSION="v0.4.0"
    curl -sSL "https://raw.githubusercontent.com/kubernetes/release/${RELEASE_VERSION}/cmd/kubepkg/templates/latest/deb/kubelet/lib/systemd/system/kubelet.service" | sed "s:/usr/bin:${DOWNLOAD_DIR}:g" | sudo tee /etc/systemd/system/kubelet.service
    sudo mkdir -p /etc/systemd/system/kubelet.service.d
    curl -sSL "https://raw.githubusercontent.com/kubernetes/release/${RELEASE_VERSION}/cmd/kubepkg/templates/latest/deb/kubeadm/10-kubeadm.conf" | sed "s:/usr/bin:${DOWNLOAD_DIR}:g" | sudo tee /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
    ~~~

4.	Install CRICTL using below commands

    ~~~
    CRICTL_VERSION="v1.22.0"
    ARCH="amd64"
    curl -L "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-${ARCH}.tar.gz" | sudo tar -C $DOWNLOAD_DIR -xz
    ~~~
    
5.	Set below permissions for CRICTL 
    
    ~~~
    cd /usr/local/bin/
    chown root:root crictl
    ln -s /usr/local/bin/crictl /bin/crictl
    ln -s /usr/local/bin/ctr /bin/ctr
    ~~~

6.	Enable and start kubelet service

    ~~~
    systemctl enable --now kubelet
    systemctl status kubelet
    ~~~

##  Master Node Configuration

Follow below steps to start Kubernetes cluster inside master node instance

1.	Initialize Kubernetes cluster using below command

    ~~~
    kubeadm init
    ~~~
    
2.	Once the cluster is created configure kubectl with below command.

    ~~~
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    export KUBECONFIG=/etc/kubernetes/admin.conf
    ~~~
    
3.	Setup pod network using below commands

    ~~~
    curl https://docs.projectcalico.org/manifests/calico-typha.yaml -o calico.yaml
    kubectl apply -f calico.yaml
    ~~~
   
   
##  Worker Node Configuration

1.	Create below directory in all the worker node instances

    ~~~
    mkdir -p /etc/kubernetes/manifests
    ~~~
    
2.	Copy the join command provided in the output of step 1 of Master Node configuration the command looks as below and execute the command from worker node as root user.
    ~~~
    kubeadm join 30.0.0.53:6443 --token kfpasj.lnjj98ndjf7bk3rl \
        --discovery-token-ca-cert-hash sha256:3f74dbc4a9d589c78a9a6ae9d132d54655b429cbbf57210804540d1431953fc
    ~~~
    
3.	Label your worker node using below command

    ~~~
    kubectl label node worker-node node-role.kubernetes.io/worker=worker
    ~~~
    
4. Get status of your worker node instances using below command.

    ~~~
    kubectl get nodes --show-labels
    ~~~
