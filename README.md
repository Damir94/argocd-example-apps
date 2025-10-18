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

## Here’s a step-by-step guide to install Minikube on Ubuntu:

### 1. Launch an EC2 instance with a t2.large instance type and Ubuntu OS
### 2. Connect to the instance and perform system updates.
```bash
  sudp apt update -y
```
### 3. Install required packages
These packages allow apt to use repositories over HTTPS:
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common lsb-release -y
```
### 4. Add Docker’s official GPG key  
```bash
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
### 5. Add Docker repository
```bash
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
### 6. Update apt again
```bash
  sudo apt update
```
### 7. Install Docker
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
### 8. Verify Docker installation and status
```bash
  docker --version
  systemctl status docker
```
<img width="1910" height="622" alt="Screenshot 2025-10-18 at 11 26 48 AM" src="https://github.com/user-attachments/assets/83382def-aebd-4266-b72f-da77f4a64255" />

### 9. Run Docker without sudo (optional but recommended)
```bash
  sudo usermod -aG docker $USER
```
### 10. Test Docker
```bash
  docker run hello-world
```
If everything is set up correctly, you’ll see a “Hello from Docker!” message.

## Here’s a step-by-step guide to install Minikube on Ubuntu:
