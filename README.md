### Description
A Node.js application which allows users to login and create users.

### Tech Stack
- AWS EKS
- Node.js
- MongoDB Atlas
- AWS CloudFormation
- AWS EFS



| What K8s will do | What K8s Requires |
| ------------- |:-------------:|
| Create your objects and manage them | Create the Cluster and the Node Instances |
| Monitor Pods and re-create them      | Setup API Server, kubelet and other services on Nodes      |
| Utilize the provided resources to apply your config | Create other resources that might be needed (e.g. Load Balancer, Filesystems)      |

# Cluster
### Configure cluster

IAM -> Create New Role for EKS Cluster -> AWS Service: EKS -> EKS - Cluster -> Role name: eksClusterRole

### Specify networking
[Creating a VPC for your Amazon EKS cluster using CloudFormation](https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html#create-vpc)

Cluster endpoint access: Public and private

### Configure logging

### Configure kubectl to connect to EKS
[Configures kubectl so that you can connect to an Amazon EKS cluster.](https://docs.aws.amazon.com/cli/latest/reference/eks/update-kubeconfig.html)

`aws eks --region us-east-1 update-kubeconfig --name Kub-dep-demo`


# Node
### Add Node Groups
Compute -> Add Node Groups

### Configure Node Group
IAM -> Create New Role for EKS Node -> AWS Service: EC2 -> AmazonEKSWorkerNodePolicy, AmazonEKS_CNI_Policy, AmazonEC2ContainerRegistryReadOnly -> Role name: eksNodeGroup

### Set compute and scaling configuration
t3.micro might cause app in pending state. Choose t3.small and above.

# Apply Kubernetes Config

`kubectl apply -f .`

### Check cluster
`kubectl get deployments`

auth-deployment    1/1     1            1           13s

users-deployment   1/1     1            1           11s


`kubectl get pods`

auth-deployment-8664687865-w2p8t    1/1     Running   0          14m

users-deployment-686f7c6949-lk5ml   1/1     Running   0          14m

`kubectl get services`
auth-service    ClusterIP      10.100.121.105   <none>                                                                   3000/TCP       20m
  
kubernetes      ClusterIP      10.100.0.1       <none>                                                                   443/TCP        78m    
  
users-service   LoadBalancer   10.100.105.48    af9761ba97cfa4fb48448545f261de12-896108335.us-east-1.elb.amazonaws.com   80:32609/TCP   20m    
  
### Check Postman
POST: af9761ba97cfa4fb48448545f261de12-896108335.us-east-1.elb.amazonaws.com/signup
  
POST: af9761ba97cfa4fb48448545f261de12-896108335.us-east-1.elb.amazonaws.com/login

# Adding EFS as a CIS Volume
### Deploy the driver:
https://github.com/kubernetes-sigs/aws-efs-csi-driver
  
`kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.3"`

### Create EC2 Security Group
Security group name: eks-efs

VPC: eksVpc-VPC

Inbound rules: NFS, eksVpc-VPC CIDR

### Create EFS Service
Name: eks-efs

VPC: eksVpc-VPC

Network access -> Security groups: eks-efs

File system ID: fs-00185cf69e92cabb7

### Update Config file
`kubectl delete deployment users-deployment`

`kubectl apply -f users.yaml`

storageclass.storage.k8s.io/efs-sc created
  
persistentvolume/efs-pv created
  
persistentvolumeclaim/efs-pvc created
  
service/users-service unchanged
  
deployment.apps/users-deployment created

