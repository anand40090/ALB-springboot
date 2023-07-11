## Pre-requisite links
https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

## Table Of contents
---
1. Infra creation for this project
   - Create the required IAM policies 1. full-access for IAM user 2. AWSLoadBalancerControllerIAMPolicy for EKS cluster
   - Create the required IAM role "SSM-FullAccess" for EC2 instance.
   - Follow the page for full VM setup with required applications [Setup an AWS EC2 Instance](https://sunitabachhav2007.hashnode.dev/jenkins-cicd-with-amazon-eks#heading-setup-an-aws-ec2-instance)
   - [Install_AWS_CLI](https://sunitabachhav2007.hashnode.dev/jenkins-cicd-with-amazon-eks#heading-install-and-setup-aws-cli)
   - [Install_EKSCTL_KUBECTL](https://sunitabachhav2007.hashnode.dev/jenkins-cicd-with-amazon-eks#heading-install-and-setup-kubectl)
   - [Installing Helm](https://helm.sh/docs/intro/install/)
   - Once the VM is created attach the IAM role "SSM-FullAccess" to the created VM.
      - Go to EC2 instance >> select the newly created VM >> Go to Actions >> Security >> Modify IAM role >> Select the IAM role "SSM-FullAccess" >> click on Update IAM role
   - Create IAM power user, attach the IAM policy "full-access". Download the user secret & private access key for future use.

1. Create EKS cluster using eksctl command
1. Create IAM OIDC provider
1. Install the TargetGroupBinding CRDs
1. Deploy the Helm chart
1. Configure AWS ALB (Apllication Load Balancer) to sit infront of Ingress
1. Deploy Sample Application
1. Verify Ingress
1. Get Ingress URL
1. Get EKS Pod data
1. Delete EKS cluster once your job is done
---




















# Detailed steps 

### 1.Create the required IAM role and policy and attach it to the created VM, same policy to be attached to IAM user.

#### Step 1 :- Create IAM policy for full access || policy name - "full-access"
1. Copy the Json code and create IAM role for full access on the multiple AWS services
2. Goto AWS IAM >> Policies >> Create policy >> open Json editor >> copy - paste below mentioned json code >> Next >> Review and create the policy
3. This policy have full access on multiple AWS services that may be used during the execution on different services.
4. Once the policy is created attach this policy to IAM role and attach that IAM role to the any of the AWS instance to provide access on the AWS services listed in the Json code. 
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Action": [
                "ec2:*",
                "ecr:*",
                "s3:*",
                "iam:*",
                "eks:*",
                "rds:*",
                "cloudformation:*",
                "sns:*",
                "ses:*",
                "autoscaling:*",
                "autoscaling-plans:*",
                "codedeploy:*",
                "acm:*",
                "cognito-idp:*",
                "shield:*",
                "wafv2:*",
                "elasticloadbalancing:*",
                "waf-regional:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
#### Step 2 :- Create IAM role and attach the above created policy to that role
1. Go to IAM >> Roles >> Create role >> Select Trusted entity type "AWS service" >> Common use cases "EC2" >> Next >>
   
   ![image](https://github.com/anand40090/ALB-springboot/assets/32446706/9af3a0db-dd7f-4705-b184-ec3c1698dd7e)

3. Add permissions >> select the newly created policy "full-access" >> Next >> Give role name >> Create role

   ![image](https://github.com/anand40090/ALB-springboot/assets/32446706/765a7b8d-7551-40de-941d-3195806492ce)

#### Step 3 :- Create IAM policy for EKS cluster 

1. This IAM role to be attached to the EKS cluster once the cluster is created
2. Copy the Jason code from iam_policy.json [GitHub Pages](https://github.com/anand40090/ALB-springboot/blob/master/iam_policy.json)
3. Perform above steps to create IAM policy
4. Set policy name "AWSLoadBalancerControllerIAMPolicy", this IAM policy is needed while EKS cluster creation


### 1. Create VM to use EKSCTL & Kubectl 
(for this use any type of client machine, no dependency on vm type / os. Just EKSCTL and Kubectl to work fine)
> Once the VM is created, login to the VM and install the prerequsites applications
> To install git, Docker, openjdk-11,

![image](https://github.com/anand40090/ALB-springboot/assets/32446706/1adbb4ab-6b96-4062-93ad-3c1084494746)
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/b65078e3-e083-4ec6-9b27-ec309f22f630)
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/0eb3e1a2-1d47-4a40-8e8a-81db1f764ef1)
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/55598957-a221-49b7-9182-655eac72e6c6)
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/a4b41213-bcb9-4596-aa7c-01e15f43649b)




### 2. Create IAM power user to execute the tasks
1. Create user
2. Download the user creadentials (secret access key and private key), this to be used to configure the user in AWS cli
3. Attch the IAM policy "full-access" to poweruser

![image](https://github.com/anand40090/ALB-springboot/assets/32446706/7f39dd27-6930-482f-a57b-38db268fa2e2)





















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

