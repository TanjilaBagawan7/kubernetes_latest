# Kubernetes Latest Version Setup (1.24)
Follow the below steps to install kubernetes cluster using Containerd as runtime environment in Centos 7

## VM requirements
1.	Create Master and Worker node VMs based on the cluster requirements
2.	Minimum machine configuration would be 2 Core and 2 GB for all the instances
3.	Full network connectivity between all machines in the cluster (public or private network is fine)
4.	Unique hostname, MAC address, and product_uuid for every node
5.	Update repository packages in all the instances

    `   sudo yum update`
