# Fully Automated K3S etcd High Availability Install

![](https://i.imgur.com/waxVImv.png)
### [View all Roadmaps](https://github.com/nholuongut/all-roadmaps) &nbsp;&middot;&nbsp; [Best Practices](https://github.com/nholuongut/all-roadmaps/blob/main/public/best-practices/) &nbsp;&middot;&nbsp; [Questions](https://www.linkedin.com/in/nholuong/)

Setting up k3s is hard.That’s why we made it easy.Today we’ll set up a High Availability K3s cluster using etcd, MetalLB, kube-vip, and Ansible.We’ll automate the entire process giving you an easy, repeatable way to create a k3s cluster that you can run in a few minutes.

# Prep

You’ll need to be sure you have Ansible installed on your machine and that it is at least 2.11+. If you don’t, you can use the install Ansible post on how to install and update it.

Second, you’ll need to provision the VMs. Here’s an easy way to create perfect Proxmox templates with cloud image and cloud init and a video if you need.

## Next, you’ll need to fork and clone the repo.While you’re at it, give it a ⭐ too :).

```git clone https://github.com/nholuongut/k3s-ansible```

Next you’ll want to create a local copy of ansible.example.cfg.

```cp ansible.example.cfg ansible.cfg```

## You’ll want to adapt this to suit your needs however the defaults should work without issue.If you’re looking for the old defaults, you can see them in this PR that remove the file.

Next you’ll need to install some requirements for ansible

```ansible-galaxy install -r ./collections/requirements.yml```

Next, you’ll want to cd into the repo and copy the sample directory within the inventory directory.

### (Be sure you’re using the latest template)

```cp -R inventory/sample inventory/my-cluster```

## Installing k3s

Next, edit the inventory/my-cluster/hosts.ini to match your systems.DNS works here too.

```
[master]
192.168.30.38
192.168.30.39
192.168.30.40

[node]
192.168.30.41
192.168.30.42

[k3s_cluster:children]
master
node
```

* Edit inventory/my-cluster/group_vars/all.yml to your liking.See comments inline.

It’s best to start using these args, and optionally include traefik if you want it installed with k3s however I would recommend installing it later with helm

It’s best to start with the default values in the repo.

## change these to your liking, the only required are: --disable servicelb, --tls-san 
```
extra_server_args: >-
  
  --node-taint node-role.kubernetes.io/master=true:NoSchedule
  --tls-san 
  --disable servicelb
  --disable traefik
extra_agent_args: >-
```
  
I would not change these values unless you know what you are doing.It will most likely not work for you but listing for posterity.

# Note: These are for an advanced use case. There isn’t a one size fits all setting for everyone and their needs, I would try using k3s with the above values before changing them.This could have undesired effects like nodes going offline, pods jumping or being removed, etc… Using these args might come at the cost of stability Also, these will not work anymore without some modifications
```
extra_server_args: "--disable servicelb --disable traefik --write-kubeconfig-mode 644 --kube-apiserver-arg default-not-ready-toleration-seconds=30 --kube-apiserver-arg default-unreachable-toleration-seconds=30 --kube-controller-arg node-monitor-period=20s --kube-controller-arg node-monitor-grace-period=20s --kubelet-arg node-status-update-frequency=5s"
extra_agent_args: "--kubelet-arg node-status-update-frequency=5s"
```

### Start provisioning of the cluster using the following command:

```ansible-playbook ./site.yml -i ./inventory/my-cluster/hosts.ini```

# Note: note: add –ask-pass –ask-become-pass if you are using password SSH login.

After deployment control plane will be accessible via virtual ip address which is defined in inventory/my-cluster/group_vars/all.yml as apiserver_endpoint

```kube config```

To get access to your Kubernetes cluster and copy your kube config locally run:

```scp ansibleuser@192.168.30.38:~/.kube/config ~/.kube/config```

# Testing your cluster

Be sure you can ping your VIP defined in inventory/my-cluster/group_vars/all.yml as apiserver_endpoint

```ping 192.168.30.222```

# Getting nodes

```kubectl get nodes```

# Deploying a sample nginx workload

```kubectl apply -f example/deployment.yml```

# Check to be sure it was deployed

```kubectl describe deployment nginx```

# Deploying a sample nginx service with a LoadBalancer

```kubectl apply -f example/service.yml```

# Check service and be sure it has an IP from metal lb as defined in inventory/my-cluster/group_vars/all.yml

```kubectl describe service nginx```

# Visit that url or curl

```curl http://192.168.30.80```

 ## => You should see the nginx welcome page.

# Notes: You can clean this up by running
```
kubectl delete -f example/deployment.yml
kubectl delete -f example/service.yml
Resetting your cluster
```

## This will remove k3s from all nodes.These nodes should be rebooted afterwards.

```ansible-playbook ./reset.yml -i ./inventory/my-cluster/hosts.ini```

I'm are always open to your feedback.  Please contact as bellow information:
### [Contact ]
* [Name: nho Luong]
* [Skype](luongutnho_skype)
* [Github](https://github.com/nholuongut/)
* [Linkedin](https://www.linkedin.com/in/nholuong/)
* [Email Address](luongutnho@hotmail.com)

![](https://i.imgur.com/waxVImv.png)
![](bitfield.png)
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/nholuong)

# License
* Nho Luong (c). All Rights Reserved.
