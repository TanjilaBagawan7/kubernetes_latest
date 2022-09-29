# Kubernetes Latest Version Setup (1.24)
Follow the below steps to install kubernetes cluster using Containerd as runtime environment in Centos 7

## VM requirements
1.	Create Master and Worker node VMs based on the cluster requirements
2.	Minimum machine configuration would be 2 Core and 2 GB for all the instances
3.	Full network connectivity between all machines in the cluster (public or private network is fine)
4.	Unique hostname, MAC address, and product_uuid for every node
5.	Update repository packages in all the instances

    `   sudo yum update`
    
6.	Set hostname for master-node on master node instance.

    `   sudo hostnamectl set-hostname master-node`
    
7.	Set hostname for worker-node on worker node instance
    
    `   sudo hostnamectl set-hostname worker-node`
    
8.	Update the /etc/hosts file in all instances

    ~~~    
    <master-node-ip> master-node
    <worker-node-ip> node1 worker-node
    ~~~
9.	Disable SELinux in all the instance and reboot.

    ~~~
    sudo setenforce 0
    sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
    sudo reboot
    ~~~  

10.	Disable SWAP in all Instance
    ~~~
    sudo sed -i '/swap/d' /etc/fstab
    sudo swapoff -a
    ~~~
11.	Update system settings for Kubernetes networking
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

12.	Execute below query for system configuration to take effect
    
    `   sudo sysctl --system`
        
##  Installing Pre-requisites Software’s and Configurations

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

12.	Install below conntrack package

    ~~~
    yum install conntrack
    ~~~
    
13.	Restart containerd service in all the Instances

    ~~~
    sudo systemctl restart containerd
    ~~~
    
14.	Install kubeadm, kubelet and kubectl using below commands

    ~~~
    DOWNLOAD_DIR=/usr/local/bin
    RELEASE="$(curl -sSL https://dl.k8s.io/release/stable.txt)"
    ARCH="amd64"
    cd $DOWNLOAD_DIR
    sudo curl -L --remote-name-all https://storage.googleapis.com/kubernetes-release/release/${RELEASE}/bin/linux/${ARCH}/{kubeadm,kubelet,kubectl}
    sudo chmod +x {kubeadm,kubelet,kubectl}
    ~~~
    
15.	Create symbolic link to use kubeadm, kubelet and kubectl

    ~~~
    ln -s /usr/local/bin/kubeadm /bin/kubeadm
    ln -s /usr/local/bin/kubelet /bin/kubelet
    ln -s /usr/local/bin/kubectl /bin/kubectl
    ln -s /usr/local/bin/ctr /bin/ctr
    ~~~
    
16.	Create kubelet service file using below commands    

    ~~~
    RELEASE_VERSION="v0.4.0"
    curl -sSL "https://raw.githubusercontent.com/kubernetes/release/${RELEASE_VERSION}/cmd/kubepkg/templates/latest/deb/kubelet/lib/systemd/system/kubelet.service" | sed "s:/usr/bin:${DOWNLOAD_DIR}:g" | sudo tee /etc/systemd/system/kubelet.service
    sudo mkdir -p /etc/systemd/system/kubelet.service.d
    curl -sSL "https://raw.githubusercontent.com/kubernetes/release/${RELEASE_VERSION}/cmd/kubepkg/templates/latest/deb/kubeadm/10-kubeadm.conf" | sed "s:/usr/bin:${DOWNLOAD_DIR}:g" | sudo tee /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
~~~

17.	Install CRICTL using below commands

    ~~~
    CRICTL_VERSION="v1.22.0"
    ARCH="amd64"
    curl -L "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-${ARCH}.tar.gz" | sudo tar -C $DOWNLOAD_DIR -xz
    ~~~
    
18.	Set below permissions for CRICTL 
    
    ~~~
    cd /usr/local/bin/
    chown root:root crictl
    ln -s /usr/local/bin/crictl /bin/crictl
    ~~~

19.	Enable and start kubelet service

    ~~~
    systemctl enable --now kubelet
    systemctl status kubelet
    ~~~
