# eks-gitops-argocd

This repository walks through implementing GitOps with GitHub Actions and ArgoCD to deploy applications via helm to EKS

## Build EKS Cluster
For this demo, we will create the EKS cluster using EKSCTL. Before that, some housekeeping
```
export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
export AZS=($(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text --region $AWS_REGION))
export K8S_VERSION=1.23
```

If you do not have eksctl installed, install it.
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv -v /tmp/eksctl /usr/local/bin
eksctl completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
```

Create a CMK for the EKS cluster to use when encrypting your Kubernetes secrets:
```
aws kms create-alias --alias-name alias/eksworkshop --target-key-id $(aws kms create-key --query KeyMetadata.Arn --output text)
```

Let’s retrieve the ARN of the CMK to input into the create cluster command.
```
export MASTER_ARN=$(aws kms describe-key --key-id alias/eksworkshop --query KeyMetadata.Arn --output text)
```

We set the MASTER_ARN environment variable to make it easier to refer to the KMS key later.

Now, let’s save the MASTER_ARN environment variable into the bash_profile
```
echo "export MASTER_ARN=${MASTER_ARN}" | tee -a ~/.bash_profile
```


Now let's create the cluster
```
eksctl create cluster -f config/cluster.yaml
```

Test the cluster
```
kubectl get nodes
# if we see our 3 nodes, we know we have authenticated correctly
```

Update kubeconfig file to interact with the EKS cluster
```
aws eks update-kubeconfig --name eks-gitops --region ${AWS_REGION}
```

###
https://aws.amazon.com/blogs/security/use-iam-roles-to-connect-github-actions-to-actions-in-aws/

## Setup GitHub Actions

## Install ArgoCD on EKS

```
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd --set server.service.type=LoadBalancer
```