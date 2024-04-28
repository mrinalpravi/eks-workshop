# EKS Workshop
This is a hands-on guide to provision a EKS cluster from scratch in an AWS account

## Getting Started
Launch a helper EC2 instance and connect to the instance using any of the following method
* SSH using key pair
* EC2 Instance Connect

Once connected, switch to super user by executing ```sudo su```

### Install the following

* AWS CLI - Might be present if choosen AMazon linux AMI
* kubectl
  
	```curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"```

	```chmod +x ./kubectl```

  ```sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl```

  ```kubectl version```
* eksctl
  
  ```curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp```
  
	```sudo mv /tmp/eksctl /usr/local/bin```

	```eksctl version```

### Configure AWSCLI
Set up AWS CLI in the Helper Instance (Keep Access Key ID and Secret Access Key handy) ```aws configure```
Verify aws cli by executing any commands, example
	```aws s3 ls``` (If this lists s3 buckets in this account, you are good to proceed)

### Create EKS Cluster using EKSCTL
```
eksctl create cluster --name sre-eks --region us-east-1 --nodegroup-name sre-node-group --node-type t3.medium --nodes 3 --nodes-min 1  --nodes-max 4  --managed
```
This command creates an EKS cluster named "sre-eks" in the "us-east-1" region with a managed node group named "sre-node-group". The node group uses "t3.medium" instances, with an initial count of 3 nodes, a minimum of 1 node, and a maximum of 4 nodes. It automatically creates a Virtual Private Cloud (VPC) for you if you don't specify one explicitly.

--name mravi-eks: This specifies the name of the EKS cluster, in this case, it's set to "sre-eks".

--region us-east-1: This specifies the AWS region in which the EKS cluster will be created. In this case, it's set to "us-east-1".

--nodegroup-name sre-node-group: This specifies the name of the node group within the EKS cluster. A node group is a set of worker nodes that run Kubernetes pods.

--node-type t3.medium: This specifies the instance type for the worker nodes in the node group. In this case, it's set to "t3.medium".

--nodes 3: This specifies the initial number of worker nodes to be launched in the node group. In this case, it's set to 3.

--nodes-min 1: This specifies the minimum number of worker nodes that the node group can scale down to. In this case, it's set to 1.

--nodes-max 4: This specifies the maximum number of worker nodes that the node group can scale up to. In this case, it's set to 4.

--managed: This flag indicates that the node group is managed by Amazon EKS. Managed node groups are recommended as they are easier to manage and maintain.

Once the Cluster is created, you are good to go with your kubernetes commands



## Create Deployment and Service (From your local machine)
```git clone https://github.com/mrinalpravi/eks-workshop.git```

### Connecting to Kubernetes Cluster in EKS
```aws eks update-kubeconfig --name sre-eks --region us-east-1 --profile sre-kochi```

```kubectl apply -f deployment.yaml```

```kubectl apply -f service.yaml```


### Deploy Kubernetes Dashboard
```kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml```
### Create an eks-admin Service Account
```kubectl apply -f eks-admin-service-account.yaml```
### Create Auth Token
```kubectl create token eks-admin -n kube-system```
### Connect to Dashboard
Start Kubectl Proxy using ```kubectl proxy```
Access the dashbord at http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
