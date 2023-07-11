## Pre-requisite links
https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

## High level tasks 
---
1. Create VM to use EKSCTL & Kubectl (for this use any type of Linux OS VM, no dependency on vm type)
2. Create IAM power user to execute the tasks. 
3. Create IAM role and policy and attach it to the created VM, same policty to be attached to IAM user.
4. Install pre-requsite applications (Git, Docker,Maven,Jdk-11,EKSCTL,KUBECTL,HELM,AWSCLI).
5. Configure the IAM user in created vm.
6. Create EKS cluster using ekstcl command
7. Create IAM OIDC provider
8. Install the TargetGroupBinding CRDs
9. Deploy the Helm chart
10. Configure AWS ALB (Apllication Load Balancer) to sit infront of Ingress
11. Deploy Sample Application
12. Verify Ingress
13. Get Ingress URL
14. Get EKS Pod data
15. Delete EKS cluster
---


















## Have Helm installed in your VM
https://helm.sh/docs/intro/install/
manualy download your desired version
Unpack it (tar -zxvf helm-v3.0.0-linux-amd64.tar.gz)
Find the helm binary in the unpacked directory, and move it to its desired destination (mv linux-amd64/helm /usr/local/bin/helm)
helm version --short

Install helm from Script >> 
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh

## What Kubernetes Ingress
"Kubernetes Ingress is an API object that provides routing rules to manage external user`s access to the services in a Kubernetes cluster,
typically via HTTPS/HTTP"


## Create EKS cluster
eksctl create cluster --name eksingressdemo --node-type t2.medium --nodes 2 --nodes-min 2 --nodes-max 3 --region ap-south-1 --zones=ap-south-1a,ap-south-1b

## Get EKS Cluster service
eksctl get cluster --name eksingressdemo --region ap-south-1

## Create IAM OIDC provider
eksctl utils associate-iam-oidc-provider --region ap-south-1 --cluster eksingressdemo --approve

## Create an IAM policy called
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

## Create a IAM role and ServiceAccount
eksctl create iamserviceaccount --cluster eksingressdemo --namespace kube-system --name aws-load-balancer-controller --attach-policy-arn arn:aws:iam::400150977086:policy/AWSLoadBalancerControllerIAMPolicy --override-existing-serviceaccounts --approve

## Install the TargetGroupBinding CRDs
kubectl apply -k github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master
kubectl get crd

## Deploy the Helm chart
helm repo add eks https://aws.github.io/eks-charts

## Configure AWS ALB (Apllication Load Balancer) to sit infront of Ingress
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller --set clusterName=eksingressdemo --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller -n kube-system --set region=ap-south-1

helm ls -n kube-system

kubectl -n kube-system rollout status deployment aws-load-balancer-controller

## Deploy Sample Application
kubectl apply -f 2048_full_latest.yaml
kubectl apply -f springboot-deployment.yaml springboot-ingress.yaml springboot-service.yaml

## Verify Ingress
 kubectl get ingress -A
kubectl get ingress/ingress-2048 -n game-2048

## Get Ingress URL
kubectl get ingress/ingress-2048 -n game-2048 -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

## Get EKS Pod data.
kubectl get pods --all-namespaces

## Delete EKS cluster
eksctl delete cluster --name eksingressdemo --region us-east-1

