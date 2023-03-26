## Create aws resources
terraform init
terraform apply

## Connect
- Configure kubectl
aws eks --region ap-northeast-1 update-kubeconfig --name demo

- Check connection with Kubernetes
kubectl get svc

## install cert-manager
helm search repo cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm search repo cert-manager

helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.11.0 --set prometheus.enabled=false --set installCRDs=true --debug

## install runner controller

kubectl create ns actions

kubectl create secret generic controller-manager -n actions --from-literal=github_token=${PAT}

helm repo add actions-runner-controller https://actions-runner-controller.github.io/actions-runner-controller
helm repo update
helm search repo actions

helm install actions actions-runner-controller/actions-runner-controller --namespace actions --version 0.22.0 --set syncPeriod=1m --debug

kubectl get pods -n actions

kubectl apply -f ./k8s/runner.yaml

## Destroy aws resource

- List all resources
terraform state list
aws_eip.nat
aws_eks_cluster.demo      
aws_eks_node_group.general
aws_iam_role.demo
aws_iam_role.nodes
aws_iam_role_policy_attachment.demo-AmazonEKSClusterPolicy
aws_iam_role_policy_attachment.nodes-AmazonEC2ContainerRegistryReadOnly
aws_iam_role_policy_attachment.nodes-AmazonEKSWorkerNodePolicy
aws_iam_role_policy_attachment.nodes-AmazonEKS_CNI_Policy
aws_internet_gateway.igw
aws_nat_gateway.nat
aws_route_table.private
aws_route_table.public
aws_route_table_association.private-ap-northeast-1a
aws_route_table_association.private-ap-northeast-1c
aws_route_table_association.public-ap-northeast-1a
aws_route_table_association.public-ap-northeast-1c
aws_subnet.private-ap-northeast-1a
aws_subnet.private-ap-northeast-1c
aws_subnet.public-ap-northeast-1a
aws_subnet.public-ap-northeast-1c
aws_vpc.main

- Destroy all resources
terraform apply -destroy

## Quick Start
https://github.com/actions/actions-runner-controller/blob/master/docs/quickstart.md

helm upgrade --install --namespace actions-runner-system --create-namespace --set=authSecret.create=true --set=authSecret.github_token="PAT" --wait actions-runner-controller actions-runner-controller/actions-runner-controller

kubectl apply -f https://github.com/actions/actions-runner-controller/releases/download/v0.22.0/actions-runner-controller.yaml