## Pre-requisite links
https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

## Table Of contents
---
1. Infra creation for this project
   - Create the required IAM policies 1. full-access for IAM user 2. AWSLoadBalancerControllerIAMPolicy for EKS cluster
   - Create the required IAM role "SSM-FullAccess" for EC2 instance and attach the IAM policy "full-access"
   - Create the required IAM role "EKS-ALB" for EKS cluster and attach the IAM policy "AWSLoadBalancerControllerIAMPolicy", "Administrator"
   - Follow the page for full VM setup with required applications [Setup an AWS EC2 Instance](https://sunitabachhav2007.hashnode.dev/jenkins-cicd-with-amazon-eks#heading-setup-an-aws-ec2-instance)
   - Create IAM power user, attach the IAM policy "full-access". Download the user secret & private access key for future use.
   - [Install_AWS_CLI_&_configure](https://sunitabachhav2007.hashnode.dev/jenkins-cicd-with-amazon-eks#heading-install-and-setup-aws-cli)
   - [Install_EKSCTL_KUBECTL](https://sunitabachhav2007.hashnode.dev/jenkins-cicd-with-amazon-eks#heading-install-and-setup-kubectl)
   - [Installing Helm](https://helm.sh/docs/intro/install/)
   - Once the VM is created attach the IAM role "SSM-FullAccess" to the created VM.
      - Go to EC2 instance >> select the newly created VM >> Go to Actions >> Security >> Modify IAM role >> Select the IAM role "SSM-FullAccess" >> click on Update IAM role
1. Create EKS cluster using eksctl command
1. Create IAM OIDC provider
1. Create a ServiceAccount
1. Install the TargetGroupBinding CRDs
1. Deploy the Helm chart
1. Configure AWS ALB (Apllication Load Balancer) to sit infront of Ingress
1. Deploy Sample Application
1. Verify Ingress
1. Get Ingress URL
1. Get EKS Pod data
1. Delete EKS cluster once your job is done
---


### 2.Create EKS cluster using eksctl command
   - This command will create EKS cluster with minimum 2 and maximum 3 nodes in ap-south-1 region. 
   - authenticator-role-arn will attach the IAM policy created for EKS cluster.
   - Node groups will be created automatically.
```
eksctl create cluster --name eksingressdemo\
--node-type t2.medium\
--nodes 2\
--nodes-min 2\
--nodes-max 3\
--region ap-south-1\
--zones=ap-south-1a,ap-south-1b\
--authenticator-role-arn=arn:aws:iam::XXXXXXXXXXX:instance-profile/SSM-FullAccess\
--auto-kubeconfig\
--asg-access\
--external-dns-access\
--appmesh-access\
--alb-ingress-access
```

![image](https://github.com/anand40090/ALB-springboot/assets/32446706/845184ab-cbee-4128-ab6e-b9e5bf1c9336)

- Verify the cluster once created
```
eksctl get cluster --name eksingressdemo --region ap-south-1
```
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/436dec35-b117-4bb5-8ea6-0c8c7c46ef4c)

### 3.Create IAM OIDC provider
```
eksctl utils associate-iam-oidc-provider --region ap-south-1 --cluster eksingressdemo --approve
```
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/784e6e3f-b64e-4572-b447-55da9917074c)

### 4. Create a ServiceAccount 
1. Create service account for EKS cluster
2. Attach the IAM policy "AWSLoadBalancerControllerIAMPolicy" which was created erlier
```
eksctl create iamserviceaccount --cluster eksingressdemo --namespace kube-system --name aws-load-balancer-controller --attach-policy-arn arn:aws:iam::XXXXX:policy/AWSLoadBalancerControllerIAMPolicy --override-existing-serviceaccounts --approve
```
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/81a3845d-f30b-4c65-b69c-3a3ea5d0f749)


### 5.Install the TargetGroupBinding CRDs
```
kubectl apply -k github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master
```
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/ef276481-1d59-4447-a8bf-d00381687187)

### 6.Deploy the Helm chart
```
helm repo add eks https://aws.github.io/eks-charts
```
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/81a00a67-2f8e-4509-b77e-7e917ebbf101)

### 7.Configure AWS ALB (Apllication Load Balancer) to sit infront of Ingress
```
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller --set clusterName=eksingressdemo --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller -n kube-system --set region=ap-south-1
```
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/3e1ac64a-44b1-429a-a736-ebe76f53dda6)

### 8.Deploy Sample Application
1. Apply the [2048_full_latest.yaml](https://github.com/anand40090/ALB-springboot/blob/master/2048_full_latest.yaml) yaml file to create service and deployment with ALB
2. Download the file locally and apply.

![image](https://github.com/anand40090/ALB-springboot/assets/32446706/6f0ed91a-6048-4f1d-bc59-aa01a47c1c39)

### 9.Verify Ingress
```
kubectl get ingress -A kubectl get ingress/ingress-2048 -n game-2048
```
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/a8c6815f-2abf-4ce9-a60c-f60c2ac9bfc6)

### 10.Get Ingress URL

```
kubectl get ingress/ingress-2048 -n game-2048 -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/165a82e5-9889-4e30-b616-9ac5050a4e4f)

### 11.Get EKS Pod data

```
kubectl get pods --all-namespaces
```
![image](https://github.com/anand40090/ALB-springboot/assets/32446706/14611068-00d2-4f70-8ae8-15215b2cf4fd)

### 12.Delete EKS cluster

```
eksctl delete cluster --name eksingressdemo --region us-east-1
```

