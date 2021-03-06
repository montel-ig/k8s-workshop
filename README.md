# Kubernetes Workshop

Installing 3 machine Kubernetes cluster on bare metal with the help of Ansible

Based on [Digital Ocean Tutorial](https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-cluster-using-kubeadm-on-ubuntu-20-04)

## What you need?
- Ansible installed on your local machine
- SSH access to 3 machines (check that connecting works with root user)
- Ubuntu 20.04 installed
- At least 2 vCPUs and 2GB RAM for each machine

## Creating Kubernetes cluster with Ansible 

- Make a copy of the `hosts.example` file and rename it to `hosts
- Edit the `hosts` file and add the IP addresses of the machines
- Also add path to your SSH public key, which you are using for connecting to the machines

### Initial setup

We will create `ubuntu` user and add sudo privileges on all machines. Also the public keys will be added, so you can 
SSH with same keys

Run Ansible playbook:

    ansible-playbook -i hosts 01-initial.yaml

Now we are able to SSH with the `ubuntu` user

### Installing Kubernetes dependencies

Now we install tools needed by Kubernetes like Docker, `kubelet` and `kubeadm`

Run Ansible playbook:

    ansible-playbook -i hosts 02-kube-dependencies.yaml

### Setting up the control-plane node

We will initialize the cluster with `kubeadm` and install networking and some configurations. Also the `kubectl` 
tool will be installed to control-plane node.

Run Ansible playbook:

    ansible-playbook -i hosts 03-control-plane.yaml

### Setting up the worker nodes

Now we first get the `kubeadm join` command from control plane, and then we will join the worker nodes to the cluster

Run Ansible playbook:

    ansible-playbook -i hosts 04-workers.yaml

### Setup local kubectl

At the control-plane setup stage we copied the `KUBECONFIG` file, so we can use `kubectl` from local machine to 
control cluster.

Run: `export KUBECONFIG=$(pwd)KUBECONFIG`

Now check that you are connected to correct cluster: `kubectl cluster-info` and/or `kubectl get nodes`

### Deploying example application

In `example-app` folder you can find two files: `deployment.yaml` and `service.yaml`

- `deployment.yaml` will deploy very basic nginx container
- `service.yaml` will create _NodePort_ service, so you can connect to application with worker node ip and specific port

Deploy nginx by running:

    kubectl apply -f example-app/deployment.yaml

Check that it's deployed:

    kubectl get deployments

Deploy Service by running:

    kubectl apply -f example-app/service.yaml

Check that service is created and also port is assigned:

    kubectl get services

Now you can open up the application in browser with `http://<worker-node-ip>:<port>`

Assigned port range is usually 30000-32767
