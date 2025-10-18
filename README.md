# A Hands-On Guide to ArgoCD on Kubernetes

## Introduction
In the modern DevOps world, GitOps has become a key practice for managing Kubernetes applications. It uses Git repositories as the single source of truth for both infrastructure and application configurations. Argo CD is one of the most popular GitOps tools — designed to continuously deliver applications to Kubernetes clusters in a declarative, automated, and secure way.
In this guide, we’ll walk you through setting up Argo CD on a Kubernetes cluster, deploying a sample application, and understanding how it simplifies continuous delivery.

## What is Argo CD?
Argo CD (Argo Continuous Delivery) is a declarative GitOps tool for Kubernetes.
It:
 - Automatically synchronizes Kubernetes manifests from a Git repository.
 - Provides a real-time view of the deployed resources through a web UI.
 - Ensures your cluster state always matches the desired state in Git.
In short: You commit code to Git → Argo CD applies it automatically to your cluster.

## Prerequisites
Before you start, make sure you have the following:

## Here’s a step-by-step guide to install docker on Ubuntu:

#### 1. Launch an EC2 instance with a t2.large instance type and Ubuntu OS
#### 2. Connect to the instance and perform system updates.
```bash
  sudp apt update -y
```
#### 3. Install required packages
These packages allow apt to use repositories over HTTPS:
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common lsb-release -y
```
#### 4. Add Docker’s official GPG key  
```bash
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
#### 5. Add Docker repository
```bash
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
#### 6. Update apt again
```bash
  sudo apt update
```
#### 7. Install Docker
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
#### 8. Verify Docker installation and status
```bash
  docker --version
  systemctl status docker
```
<img width="1910" height="622" alt="Screenshot 2025-10-18 at 11 26 48 AM" src="https://github.com/user-attachments/assets/83382def-aebd-4266-b72f-da77f4a64255" />

#### 9. Run Docker without sudo (optional but recommended)
```bash
  sudo usermod -aG docker $USER
```
#### 10. Test Docker
```bash
  docker run hello-world
```
If everything is set up correctly, you’ll see a “Hello from Docker!” message.

## Here’s a step-by-step guide to install Minikube on Ubuntu:

#### 1. Install dependencies
Minikube requires curl and a hypervisor like docker (or virtualbox):
```bash
 sudo apt install curl conntrack -y
```
Make sure Docker is installed if you want to use it as the driver.
#### 2. Download Minikube binary
Get the latest Minikube release:
```bash
 curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```
#### 3. Install Minikube
```bash
 sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
#### 4. Verify installation
```bash
minikube version
```
#### 5. Start Minikube
You can start Minikube using Docker as the driver:
```bash
 minikube start --driver=docker
```
#### 6. Test Minikube
Check cluster status:
```bash
 minikube status
```
<img width="762" height="247" alt="Screenshot 2025-10-18 at 11 30 22 AM" src="https://github.com/user-attachments/assets/9786f7f4-23fb-4608-909d-07a4c0033831" />

## Here’s a clean way to install kubectl (the Kubernetes command-line tool) on Ubuntu:

#### 1. Download the latest kubectl release
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
#### 2. Make it executable
```bash
chmod +x kubectl
```
#### 3. Move it to your PATH
```bash
sudo mv kubectl /usr/local/bin/
```
#### 4. Verify installation
```bash
kubectl version --client
```
<img width="1643" height="276" alt="Screenshot 2025-10-18 at 11 31 00 AM" src="https://github.com/user-attachments/assets/7c19f71e-4235-4cbf-888a-d100b1c1f15e" />

### Step 1: Install Argo CD
Create a dedicated namespace for Argo CD:
```bash
kubectl create namespace argocd
```
Install Argo CD components using the official YAML manifest:
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Verify the pods are running:
```bash
kubectl get pods -n argocd
```
<img width="1118" height="342" alt="Screenshot 2025-10-18 at 11 32 26 AM" src="https://github.com/user-attachments/assets/155ee4fa-aa79-4323-b3b3-2a0bfde31cad" />

### Step 2: Then, check the service. You should see argocd-server, which is accessible on port 80. You can verify this with the provided IP
```bash
kubectl get svc -n argocd
```
<img width="1353" height="239" alt="Screenshot 2025-10-18 at 12 29 21 PM" src="https://github.com/user-attachments/assets/e3b62b15-1f83-4f08-96ab-ba8ce136c933" />

### 3. Now, the issue is that there is a cluster IP, so we cannot access it. To resolve this, change the cluster IP to a node port.
```bash
kubectl edit svc argocd-server -n argocd
```
<img width="470" height="157" alt="Screenshot 2025-10-18 at 12 30 51 PM" src="https://github.com/user-attachments/assets/df0e0b20-ea93-452f-b19e-22c061a62ff5" />

### 4. Now, forward the port to make it accessible over the internet.
```bash
  kubectl port-forward --address 0.0.0.0 service/argocd-server 31124:80 -n argocd
```
<img width="1893" height="1015" alt="Screenshot 2025-10-18 at 12 38 33 PM" src="https://github.com/user-attachments/assets/a3405cfe-34a0-4f01-8858-c1bc3b8b61f4" />

### 5. Fetch the password for login.
```bash
 kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
<img width="1911" height="925" alt="Screenshot 2025-10-18 at 12 40 10 PM" src="https://github.com/user-attachments/assets/2247a0bf-d175-4fc5-9229-c1523f6ddb01" />

## Deploying Your First Application with ArgoCD
#### Step 1: Create a Git Repository
First, create a Git repository that contains your Kubernetes manifests or clone my repo. 
This repository will serve as the source for ArgoCD to deploy your application.

### Step 2: Create an Application in ArgoCD
Using the ArgoCD UI
 - Create an Application in ArgoCD:
 - In the ArgoCD dashboard, click on New App and fill in the following details:
 - Application Name: my-first-time
 - Project: defaul
 - Sync Policy: Manual or Automatic (your choice)
 - Repository URL: URL of your Git repository
 - Path: Path to your Kubernetes manifests in the repositor
 - Cluster URL: Leave as default for your current cluster
 - Namespace: The namespace where the application should be deployed

<img width="1791" height="975" alt="Screenshot 2025-10-18 at 12 44 01 PM" src="https://github.com/user-attachments/assets/a7c84287-8d1b-4d5b-806d-77f0deaf8f87" />
Step 1: ArgoCD Application Creation

<img width="1476" height="694" alt="Screenshot 2025-10-18 at 12 45 49 PM" src="https://github.com/user-attachments/assets/8068c558-aeb3-4300-a05a-9900f695e6b4" />
Step 2: ArgoCD Application Creation

### Once the application is created, click on Sync to deploy the application to your Kubernetes cluster. If Sync Policy Set to Automatic then application will be deployed to kubernetes automatically.

<img width="851" height="526" alt="Screenshot 2025-10-18 at 12 46 30 PM" src="https://github.com/user-attachments/assets/864ad3ec-a7cd-4798-9115-43f585c0a3a8" />

<img width="1693" height="578" alt="Screenshot 2025-10-18 at 12 46 41 PM" src="https://github.com/user-attachments/assets/68dc8513-9feb-4b0f-a768-bcdf27ac3589" />

## Conclusion
You’ve now set up Argo CD and deployed your first Kubernetes app using GitOps!
From here, you can:
 - Integrate CI/CD workflows (e.g., GitHub Actions → Argo CD).
 - Manage multiple clusters from one Argo CD instance.
 - Enforce policies and role-based access control (RBAC).
   
Argo CD bridges the gap between developers and operations — making deployments faster, traceable, and reliable.
