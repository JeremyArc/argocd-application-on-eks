# ArgoCD Application on EKS

This repository provides a setup for deploying applications on Amazon EKS using ArgoCD for continuous delivery. The setup ensures streamlined and automated deployments with GitOps principles.

# Table of Contents

- Prerequisites
- Architecture
- Implementation
- Clear resources

# Prerequisites

Before you begin, ensure you have the following installed:

- kubectl
- Helm
- AWS CLI
- AWS credential set on your local machine at `~/.aws/credential`
- eksctl
- ArgoCD CLI
- A domain name (recommend AWS Route53)

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

## Step 2: Get the created cluster's credential (specifiy which cluster to work with)

```bash
eksctl utils write-kubeconfig --cluster argocd-cluster --region ap-southeast-1
```

## Step 3: Install ingress controller in EKS cluster

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace kube-system
```
### Ref: https://helm.nginx.com/

## Step 4: Install ArgoCD in EKS cluster

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Step 5: Change the argocd-server service type to LoadBalancer

By default, the Argo CD API server is not exposed with an external IP.

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
Or you can forward port and access it via localhost:8080 

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## Step 6: Login to ArgoCD UI using load balancer's DNS ot localhost

- user: `admin`
- password: use this command to get default admin's password `argocd admin initial-password -n argocd`
- Remark: You should change password and delete this credential after the first login.

## Step 7: Deploy ArgoCD applications for both frontend and backend using helm

```bash
# frontend app
helm install argo-for-frontend-application ./argocd/k8s/helm -f ./argocd/k8s/helm-values/values-dev-frontend-argocd.yaml

# backend app
helm install argo-for-backend-application ./argocd/k8s/helm -f ./argocd/k8s/helm-values/values-dev-backend-argocd.yaml
```

## Step 8: Attach ingess load balancer DNS to your A alias record in Route53 for each `frontend` and `backend` domain that defined in manifest file.

## Step 9: Test ArgoCD application functionality

Edit one of frontend or backend manifest files then commit change to main branch and wait around 3 minute then ArgoCD will apply the new changes.

# Clear resources

If you need to remove all the resource, use this command

```bash
eksctl delete cluster --name argocd-cluster --region ap-southeast-1
```
