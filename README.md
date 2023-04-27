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

Now let's create the cluster
```
eksctl create cluster -f config/cluster.yaml
```

## Setup GitHub Actions

## Install ArgoCD on EKS

