# KOPS-Setup
Using Kops (Kubernetes Operations) I'm setting up Kubernetes on AWS for production (Its a demo work)
## 🚀 Our main motive is to setup Kubernete using KOPS on EC2
### Recommendation:
For **development or testing**, you can start with:
-  1 x Master (t3.medium)
-  1 x Node (t3.medium)

For **production**, you’ll need:
-  3 x Masters (in 3 AZs, t3.medium or larger)
-  2+ Nodes depending on workload

### let me drive you into the prerequisites✅
Create an EC2 instance with required **dependencies** :
1. **Phython3**
2. **AWS CLI**
3. **Kubectl**

## 📦 Installation

1. **Add kubernetes GPG Key**
   
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
2. **Add kubernetes API Repository**

```bash
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
```
3.  **Update APT and Install Kubernetes + Python Tools**

```bash
sudo apt-get update
sudo apt-get install -y python3-pip apt-transport-https kubectl
```
4.  **Install and Upgrade AWS CLI with pip**

```bash
pip3 install awscli --upgrade
```
5.  **Add pip's local bin directory to PATH**

```bash
export PATH="$PATH:/home/ubuntu/.local/bin/"
```
## Install KOPS (the key player)
```bash
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

chmod +x kops-linux-amd64

sudo mv kops-linux-amd64 /usr/local/bin/kops
```
## Provide the below permissions to your IAM user. If you are using the admin user, the below permissions are available by default
1.  `AmazonEC2FullAccess`
2.  `AmazonS3FullAccess`
3.  `IAMFullAccess`
4.  `AmazonVPCFullAccess`

## Set up AWS CLI configuration on your EC2 Instance or Laptop
### Run 
```bash
aws configure
```
  -you will prompted to enter the following details:
```bash
AWS Access Key ID [****************NE5G]:
AWS Secret Access Key [****************FJE3]:
Default region name [us-east-1]:
Default output format [table]:
```
You need to enter the requested AWS Access Key ID, AWS Secret Access Key, Default region name, Default output format based on your IAM user.
## Kubernetes Cluster Installation:
### Create S3 bucket for storing the KOPS objects.
```bash
aws s3api create-bucket --bucket kops-abhi-storage --region us-east-1
```
### Create the cluster
```bash
kops create cluster --name=demok8scluster.k8s.local --state=s3://kops-abhi-storage --zones=us-east-1a --node-count=1 --node-size=t2.micro --master-size=t2.micro  --master-volume-size=8 --node-volume-size=8
```
### Important: Edit the configuration as there are multiple resources created which won't fall into the free tier
```bash
kops edit cluster myfirstcluster.k8s.local
```
Build the cluster
```bash
kops update cluster demok8scluster.k8s.local --yes --state=s3://kops-abhi-storage
```
This will take a few minutes to create............
After a few mins, run the below command to verify the cluster installation.
```bash
kops validate cluster demok8scluster.k8s.local
```
