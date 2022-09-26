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
