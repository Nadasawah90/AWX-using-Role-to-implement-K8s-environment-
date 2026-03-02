# Deploy a Production Ready Kubernetes Cluster using Role of AWX 
prrequests : 
# 1- AWX VM : 
✅Step 1: Install Required Packages (CentOS)
sudo dnf update -y
sudo dnf install -y git make curl wget
sudo yum install -y git make curl wget
✅Step 2: Install Docker (CentOS)
sudo dnf remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
Add Docker repo
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
Install Docker
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
Start & Enable Docker
sudo systemctl enable docker
sudo systemctl start docker
Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
✅ Step 3: Install Minikube (CentOS)
Install kubectl
sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
Start Minikube
minikube start --driver=docker --cpus=2 --memory=8192 --disk-size=40g --addons=ingress
✅ Step 4: Deploy AWX Operator

Clone operator:
git clone https://github.com/ansible/awx-operator.git
cd awx-operator
git checkout 2.19.0

Set namespace:

export NAMESPACE=ansible-awx
make deploy

Check pods:

kubectl get pods -n ansible-awx

Wait until all pods are Running.

✅ Step 5: Create AWX Instance

Copy demo file:

cp awx-demo.yml awx-centos.yml

Edit:

vi awx-centos.yml

Change name if needed:

metadata:
  name: awx-centos

Apply:

kubectl apply -f awx-centos.yml -n ansible-awx

Check pods:

kubectl get pods -n ansible-awx

Wait until awx-centos-* pods are Running.

✅ Step 6: Access AWX Dashboard

kubectl port-forward --address 0.0.0.0 svc/awx-centos-service 8086:80 -n ansible-awx

Access from browser:

http://192.168.142.168:8086
✅ Step 7: Get Admin Password

List secrets:

kubectl get secret -n ansible-awx | grep password

Decode password:

kubectl get secret awx-centos-admin-password -o jsonpath="{.data.password}" -n ansible-awx | base64 --decode; echo

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a9dcb74c-1c28-4b79-8d12-8b11f718ffff" />

# 2-  Master " 192.168.142.173" & 192.168.142.174 Workers
SSH access between AWX " 12.168.142.169" and nodes
Passwordless sudo

# 3- STEP-BY-STEP – Use Kubespray in AWX

1️⃣ Clone Kubespray

On your Git machine:

git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray

Checkout stable version (recommended):

git checkout release-2.24

Push this repo to your GitHub/GitLab (if needed).

2️⃣ Create Inventory for Your Cluster

Copy sample inventory:

cp -rfp inventory/sample inventory/mycluster

Edit:

vi inventory/mycluster/inventory.ini

Example:

[all]
master1 ansible_host=192.168.142.173 ip=192.168.142.168
worker1 ansible_host=192.168.142.174 ip=192.168.142.168

[kube_control_plane]
master1

[etcd]
master1

[kube_node]
worker1

[k8s_cluster:children]
kube_control_plane
kube_node

Commit and push to Git.

3️⃣ Create Project in AWX

AWX → Resources → Projects → Add

Name: Kubespray

Source: Git

URL: https://github.com/Nadasawah90/AWX-using-Role-to-implement-K8s-environment-.git


Click Sync

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cc76e1a7-2247-41a6-a799-e116c0001077" />


4️⃣ Create Inventory in AWX

AWX → Resources → Inventories → Add

Name:

K8S-Kubeadm

Add hosts:

master : 
slave : 

(You can also import via file if preferred)

5️⃣ Add Machine Credential

AWX → Credentials → Add

Type: Machine
Add:

SSH user

Private key

Enable privilege escalation

6️⃣ Create Job Template

AWX → Templates → Add Job Template

Fill:

Name: Deploy Kubernetes Cluster

Inventory: K8S-Kubeadm

Project: Kubespray

Playbook:

cluster.yml

Credentials: SSH credential

Save.

