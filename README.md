# 🚀 KOps Setup on AWS (Amazon Linux / Ubuntu)
The easiest way to get a production grade Kubernetes cluster up and running.
### What is kOps?
-  We like to think of it as kubectl for clusters.
-  kops will not only help you create, destroy, upgrade and maintain production-grade, highly available, Kubernetes cluster, but it will also provision the necessary cloud infrastructure.
## Goal of the Day : Setup Kubernete using KOPS on EC2
### Realtime Recommendation:
For **development or testing**, you can start with:
-  1 x Master (t3.medium)
-  1 x Node (t3.medium)

For **production**, you’ll need:
-  3 x Masters (in 3 AZs, t3.medium or larger)
-  2+ Nodes depending on workload

### let me drive you into the prerequisites✅
1.  AWS Account with free tier or pay as go
2.  Amazon Linux 2 or Ubuntu EC2 instance for running kops
3.  IAM User or Role with admin-like access (EC2, S3, Route53, IAM, etc.)
4.  Set up AWS CLI configuration on your EC2 Instance
### Run 
```bash
aws configure
```
-  you will prompted to enter the following details:
```bash
AWS Access Key ID [****************NE5G]:
AWS Secret Access Key [****************FJE3]:
Default region name [us-east-1]:
Default output format [table]:
```
-  You need to enter the requested AWS Access Key ID, AWS Secret Access Key, Default region name, Default output format based on your IAM user.
5.  Registered domain (optional) or use .k8s.local for test clusters (gossip DNS)
6.  Ensure it’s in a public subnet with internet access

## 📦 Installation
### Step 1: Install Required Tools
#### Update packages and install deps
```bash
sudo yum update -y
sudo yum install -y curl unzip wget
```
(Use `apt` instead of `yum` on Ubuntu)
#### Install kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```
 #### Install kOps
 ```bash
curl -LO https://github.com/kubernetes/kops/releases/latest/download/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
```
### Step 2: Create an S3 Bucket for Cluster State
```bash
aws s3api create-bucket \
  --bucket <your-kops-state-bucket> \
  --region <your-region> \
  --create-bucket-configuration LocationConstraint=<your-region>
```
For Example:
```bash
aws s3api create-bucket \
  --bucket kops-abhi913-storage \
  --region us-east-1
```
Enable versioning (recommended):
```bash
aws s3api put-bucket-versioning \
  --bucket kops-abhi913-storage \
  --versioning-configuration Status=Enabled
```
### Step 3: Set Environment Variables
```bash
export KOPS_STATE_STORE=s3://kops-abhi913-storage
```
Make it persistent:
```bash
echo 'export KOPS_STATE_STORE=s3://kops-abhi913-storage' >> ~/.bash_profile
source ~/.bash_profile
```
### Step 4: Configure IAM Permissions for kOps execution role
Attach these AWS managed policies or equivalent to your EC2 IAM role:
-  AmazonEC2FullAccess
-  AmazonS3FullAccess
-  AmazonVPCFullAccess
-  IAMFullAccess
-  AmazonRoute53FullAccess (if using DNS)
-  AmazonEventBridgeFullAccess
-  AmazonSQSFullAccess
##### You may also need custom inline policies for:
-  sqs:TagQueue
-  events:ListRules
##### After creating the role attach this to your EC2 instance 

### Step 5: Create the Kubernetes Cluster
For Test Setup (Gossip DNS):
```bash
kops create cluster \
  --name=kopsk8scluster.k8s.local \
  --state=${KOPS_STATE_STORE} \
  --zones=us-east-1a \
  --node-count=1 \
  --node-size=t2.micro \
  --control-plane-size=t2.micro \
  --node-volume-size=8 \
  --control-plane-volume-size=8 \
  --yes
```
-  Replace with your preferred region, instance sizes, and cluster name.
-  This will take a few minutes to create............
  
### Step 6: Wait and Validate
Check EC2 Instances:
-  1 master node
-  1+ worker node(s)
#### Validate the cluster:
After a few mins, run the below command to verify the cluster installation.
```bash
kops validate cluster --name=kopsk8scluster.k8s.local
```
### Step 7: Test with kubectl
##### Export cluster config to kubectl:
```bash
kops export kubecfg --name=kopsk8scluster.k8s.local
```
#### Get node info:
```bash
kubectl get nodes
```

### Step 8: Deploy a Test Pod
```bash
kubectl run nginx --image=nginx --restart=Never --port=80
kubectl get pods
```
(Optional) Expose:
```bash
kubectl expose pod nginx --type=NodePort --port=80
kubectl get svc
```
Access:
```bash
http://<EC2 Public IP>:<NodePort>
```
### You can also write a manifest for creating the pod
1.  Create a file **nginx-pod.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
```
2.  Apply it:
```bash
kubectl apply -f nginx-pod.yaml
```
3.  Check status:
```bash
kubectl get pods
```
### Step 9: Delete Cluster (When Done)
```bash
kops delete cluster --name=kopsk8scluster.k8s.local --yes
```
Delete S3 bucket: (If ot used further)
```bash
aws s3 rb s3://kops-abhi913-storage --force
```
## Notes
-  .k8s.local = Gossip DNS (for test/dev). Use Route53 for production.
-  Ensure IAM role has full required permissions.
-  For multi-AZ and HA: use --zones=us-east-1a,us-east-1b,us-east-1c
