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
3.  Configure IAM Permissions for kOps execution role to create and organize (EC2, S3, Route53, IAM, etc.)
Attach these AWS managed policies or equivalent to your EC2 IAM role:
-  `AmazonEC2FullAccess`
-  `AmazonS3FullAccess`
-  `AmazonVPCFullAccess`
-  `IAMFullAccess`
-  `AmazonRoute53FullAccess` (if using DNS)
-  `AmazonEventBridgeFullAccess`
-  `AmazonSQSFullAccess`
> After creating the role attach this to your EC2 instance 
4.  Set up AWS CLI configuration on your EC2 Instance admin-like access
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
5.  Registered domain (optional) or use .k8s.local for test clusters **(gossip DNS)**
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
curl -LO "https://dl.k8s.io/release/v1.30.1/bin/linux/amd64/kubectl"
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
In order to store the state of your cluster, and the representation of your cluster, we need to create a dedicated S3 bucket for kops to use. This bucket will become the source of truth for our cluster configuration.
```bash
aws s3api create-bucket \
    --bucket prefix-example-com-state-store \
    --region us-east-1
```
Note: S3 requires `--create-bucket-configuration LocationConstraint=<region>` for regions other than `us-east-1`.
#### For Example: if you choose other than `us-east-1`, lets say `ap-south-1`
```bash
 aws s3api create-bucket \
  --bucket kops-abhi303-storage \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1
```
##### For now lets us `us-east-1`
```bash
aws s3api create-bucket \
    --bucket kops-abhi303-storage \
    --region us-east-1
```
Expected result if s3 bucket is created:
```bash
---------------------------------------------------------------
|                        CreateBucket                         |
+----------+--------------------------------------------------+
|  Location|  http://kops-abhi303-storage.s3.amazonaws.com/   |
+----------+--------------------------------------------------+
```
Enable versioning (STRONGLY recommended) in case you ever need to revert or recover a previous state store:
```bash
aws s3api put-bucket-versioning \
  --bucket kops-abhi303-storage \
  --versioning-configuration Status=Enabled
```
## Cluster OIDC store:
In order for ServiceAccounts to use external permissions (aka IAM Roles for ServiceAccounts), you also need a bucket for hosting the OIDC documents. While you can reuse the bucket above if you grant it a public ACL, we do recommend a separate bucket for these files.
#### The ACL must be public so that the AWS STS service can access them.
```bash
# Step 1: Create the bucket
aws s3api create-bucket \
    --bucket kops-abhi303-storage-oidc \
    --region us-east-1 \
    --object-ownership BucketOwnerPreferred

# Step 2: Set public access block (only if you really want public)
aws s3api put-public-access-block \
  --bucket kops-abhi303-storage-oidc \
  --public-access-block-configuration '{
    "BlockPublicAcls": false,
    "IgnorePublicAcls": false,
    "BlockPublicPolicy": false,
    "RestrictPublicBuckets": false
}'

# Step 3: (Optional, risky) Set ACL to public-read
aws s3api put-bucket-acl \
  --bucket kops-abhi303-storage-oidc \
  --acl public-read
```

### Using S3 default bucket encryption
kops supports default bucket encryption to encrypt its state in an S3 bucket. This way, the default server side encryption set for your bucket will be used for the kOps state too. You may want to use this AWS feature, e.g., for easily encrypting every written object by default or when you need to use specific encryption keys (KMS, CMK) for compliance reasons.
> If your S3 bucket has a default encryption set up, kOps will use it:
```bash
aws s3api put-bucket-encryption \
  --bucket my-godown-storage \
  --server-side-encryption-configuration '{
  "Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]
  }'
```
### Step 3:  Create the Kubernetes Cluster
#### Prepare local environment
We're ready to start creating our first cluster! Let's first set up a few environment variables to make the process easier.
> For a gossip-based cluster, make sure the name ends with k8s.local. For example:
```bash
export NAME=amazing-app-cluster.k8s.local
export KOPS_STATE_STORE=s3://kops-abhi303-storage
```
Make it persistent:
```bash
echo 'export KOPS_STATE_STORE=s3://kops-abhi303-storage' >> ~/.bash_profile
source ~/.bash_profile
```
#### For Test Setup (Gossip DNS):
**The below command will generate a cluster configuration, but will not start building it. Make sure you have generated an SSH key pair before creating your cluster.**
```bash
kops create cluster \
  --name=${NAME} \
  --cloud=aws \
  --zones=us-east-1a \
  --discovery-store=${KOPS_STATE_STORE}/${NAME}/discovery \
  --state=${KOPS_STATE_STORE}
```
-  Replace with your preferred deatails
-  This will take a few minutes to create ............
> All instances created by kops will be built within ASG (Auto Scaling Groups), which means each instance will be automatically monitored and rebuilt by AWS if it suffers any failure.

#### Customize Cluster Configuration:
Now we have a cluster configuration, we can look at every aspect that defines our cluster by editing the description.
```bash
kops edit cluster --name ${NAME}
```

> This opens your editor (as defined by $EDITOR) and allows you to edit the configuration. The configuration is loaded from the S3 bucket we created earlier, and automatically updated when we save and exit the editor.

> We'll leave everything set to the defaults for now, but the rest of kops documentation covers additional settings and configuration you can enable.
### Build the Cluster
Now we take the final step of actually building the cluster. This'll take a while. Once it finishes you'll have to wait longer while the booted instances finish downloading Kubernetes components and reach a "ready" state.
```bash
kops update cluster --name ${NAME} --yes --admin
```
Expected result: 
```bash
Cluster is starting.  It should be ready in a few minutes.

Suggestions:
 * validate cluster: kops validate cluster --wait 10m
 * list nodes: kubectl get nodes --show-labels
 * ssh to a control-plane node: ssh -i ~/.ssh/id_rsa ubuntu@
 * the ubuntu user is specific to Ubuntu. If not using Ubuntu please use the appropriate user
 * read about installing addons at: https://kops.sigs.k8s.io/addons.
```
### Use the cluster:
> The configuration for your cluster was automatically generated and written to ~/.kube/config for you!
-  A simple Kubernetes API call can be used to check if the API is online and listening. Let's use kubectl to check the nodes.
```bash
kubectl get nodes
```
kops also ships with a handy validation tool that can be ran to ensure your cluster is working as expected.
After a few mins, run the below command to verify the cluster installation.
```bash
kops validate cluster --name=amazing-app-cluster.k8s.local
```
Check EC2 Instances:
-  1 master node
-  1+ worker node(s)

### Step 4: Deploy a Test Pod
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
### Step 5: Delete Cluster (When Done)
```bash
kops delete cluster --name=amazing-app-cluster.k8s.local --yes
```
Delete S3 bucket: (If ot used further)
```bash
aws s3 rb s3://kops-abhi913-storage --force
```
## Notes
-  .k8s.local = Gossip DNS (for test/dev). Use Route53 for production.
-  Ensure IAM role has full required permissions.
-  For multi-AZ and HA: use --zones=us-east-1a,us-east-1b,us-east-1c
