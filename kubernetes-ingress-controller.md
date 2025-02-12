## Setup OIDC Connector
```bash
export cluster_name=my-eks-cluster
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5) 
```
## Check if there is an IAM OIDC provider configured already
```bash
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
## Download IAM Policy
```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```
## Create IAM Policy
```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

## Create IAM Role
```bash
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

## Deploy ALB Controller
add helm repo
```bash
helm repo add eks https://aws.github.io/eks-charts
```
## Update the repo

```bash
helm repo update eks
```

## Install the ALB Controller
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```

## Verify that the deployments are running.
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
## Download the current policy
```bash
aws iam get-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --version-id $(aws iam get-policy --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --query 'Policy.DefaultVersionId' --output text) \
    --query 'PolicyVersion.Document' --output json > policy.json
```

## Edit policy.json to add the missing permissions
```bash
{
  "Effect": "Allow",
  "Action": "elasticloadbalancing:DescribeListenerAttributes",
  "Resource": "*"
}
```

## Create a new policy version
```bash
aws iam create-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://policy.json \
    --set-as-default
```