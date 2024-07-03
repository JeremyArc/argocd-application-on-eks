# ArgoCD Application on EKS

This repository provides a setup for deploying applications on Amazon EKS using ArgoCD for continuous delivery. The setup ensures streamlined and automated deployments with GitOps principles.

# Table of Contents

- Prerequisites
- Architecture
- Implementation

# Prerequisites

Before you begin, ensure you have the following installed:

- kubectl
- Helm
- AWS CLI
- AWS credential set on your local machine at`~/.aws/credential`
- eksctl
- ArgoCD CLI

Additionally, you need an AWS account with the necessary permissions to create and manage EKS clusters and other associated resources.

# Architecture

This repository sets up the following architecture:

- Amazon EKS Cluster: Managed Kubernetes cluster on AWS.
- ArgoCD: A declarative continuous delivery tool for Kubernetes.
- Application Deployment: Automated deployment of applications on EKS using ArgoCD.

Remark: this is a ArgoCD focus project, so for the `frontend` and `backend` application's images are public `nginx bitnami` for mockup the both applications.

# Implementation

## Step 1: Create an EKS Cluster

Use eksctl to create an EKS cluster:

```bash
eksctl create cluster \
--name argocd-cluster \
--region ap-southeast-1 \
--nodegroup-name linux-nodes \
--node-type t2.medium \
--nodes 2
```

Change the configuration by your own setup.

## Step 2: Get the created cluster's credential (specific which cluster to work with)

```bash
eksctl utils write-kubeconfig --cluster argocd-cluster --region ap-southeast-1
```

## Step 3: Install ingress controller in EKS cluster

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace kube-system
```

## Step 4: Install ArgoCD in EKS cluster

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Step 5: Change the argocd-server service type to LoadBalancer

By default, the Argo CD API server is not exposed with an external IP. To access the API server, choose one of the following techniques to expose the Argo CD API server:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

## Step 6: Forward ArgoCD port publicly

Kubectl port-forwarding can also be used to connect to the API server without exposing the service.

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Remark: If the terminal still in process on port forwarding, you can press `Ctrl` + `c`

## Step 7: Login to ArgoCD UI using load balancer's DNS

- user: `admin`
- password: use this command to get default admin's password `argocd admin initial-password -n argocd`
- Remark: You should change password and delete this credential after the first login.

## Step 8: Deploy ArgoCD applications for both frontend and backend using helm

```bash
# frontend app
helm install argo-for-frontend-application ./argocd/k8s/helm -f ./argocd/k8s/helm-values/values-dev-frontend-argocd.yaml

# backtend app
helm install argo-for-backend-application ./argocd/k8s/helm -f ./argocd/k8s/helm-values/values-dev-backend-argocd.yaml
```

## Step 9: Test ArgoCD application functionality

Edit one of frontend or backend manifest files then commit change to main branch and wait around 3 minute then ArgoCD will apply the new changes.
