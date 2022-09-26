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
    
