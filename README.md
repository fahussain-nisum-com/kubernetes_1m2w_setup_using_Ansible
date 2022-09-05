# kubernetes 1 Master 2 Worker Nodes setup using Ansible

## Part 1: Ansible installation and Setup
### Step 1:
#### On Ansible control node

==> Update your os
```bash
$ sudo apt update
```	

==> Install ansible
```bash
$ sudo apt install ansible
```
### Step 2: 

==> First ssh all target machines manually

==> Now, in your users home directory generate keys 
```bash
$ ssh-keygen -t ed25519 -C "ansible"
```

==> Don't add passphrase in case of Ansible keys generation o.w it will prompt you for it

==> Name the key with ansible by giving path as /home/user1/.ssh/ansible

==> Now copy id to targets one by one using this command 
```bash
$ ssh-copy-id -i ~/.ssh/ansible 192.168.117.135
```

==> Lets test it now and it must not prompt you for password
```bash
$ ssh -i ~/.ssh/ansible 192.168.117.135
```
### Step 3:
Open and add following lines in file
```bash
$ sudo vim /etc/ansible/hosts
```
```
ansible-node1 ansible_host=192.168.117.135
ansible-node2 ansible_host=192.168.117.133
ansible-node3 ansible_host=192.168.117.134
```
Test all target machines are responding
```bash
$ ansible all --key-file ~/.ssh/ansible -m ping
```
View details of target machines
```bash
$ ansible all -a "df -h"
```

> Now Ansible setup is done and we will move towards Kubernetes deployment

## Part 2: Kubernetes deployment
We will be deploying: 
1. One Master Node
2. Two Worker Nodes

==> Clone repository in your project folder and get into it
```bash
$ git clone https://github.com/fahussain0/kubernetes_1m2w_setup_using_Ansible.git
```

### Step 1: Setup ssh to root of target machines:
> Set permitrootlogin to yes in sshd_config on target machines

> open .ssh folder in home directory, create if it don't exist

> add ansible.pub in authorized_keys

==> To test: Run this command on your Ansible controller
```bash
$ ssh -i ~/.ssh/ansible root@192.168.117.135
```
==> Machine must not ask for password because key is generated without paraphrase


### Step 2: Passwordless root user  creation
Open and change Target_machine's_IP in hosts file

Run this command to execute the playbook
```bash
$ ansible-playbook --key-file ~/.ssh/ansible -i hosts users.yml
```
Test target machines by running this command:
```bash
$ ansible all --key-file ~/.ssh/ansible -i hosts -m ping
```
> Note: User kube will be created which cannot be directly logged in.
TO login firstly login root and than switch user to kube

### Step 3: Install Kubernetes with Ansible Playbook

Run this command to execute the playbook
```bash
$ ansible-playbook --key-file ~/.ssh/ansible -i hosts install-k8s.yml
```	

### Step 4: Creating a Kubernetes Cluster on Master Node

Run this command to execute the playbook
```bash
$ ansible-playbook --key-file ~/.ssh/ansible -i hosts master.yml
```

### Step 5: Join Worker Nodes to Kubernetes Cluster using Ansible Playbook

> Note: This script has issue in joining, You need to get join command and copy it in your ansible controller's project folder and it will than copy them to remote machines

Run this command to execute the playbook
```bash
$ ansible-playbook --key-file ~/.ssh/ansible -i hosts join-workers.yml
```

To test all nodes are connected and ready, go on Kubernetes master node and list nodes are in READY state, They may take some time
```bash
$ kubectl get nodes
```

Done! Your nodes will be up and running

### For details you can follow:
This link to setup Ansible 
```
https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-20-04
```
This link to setup kubernetes using yml files provided
```
https://buildvirtual.net/deploy-a-kubernetes-cluster-using-ansible/
```
