# KOPS-Setup
Using Kops (Kubernetes Operations) I'm setting up Kubernetes on AWS for production (Its a demo work)
## 🚀 Our main motive is to setup Kubernete using KOPS on EC2
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
1.  AmazonEC2FullAccess
2.  AmazonS3FullAccess
3.  IAMFullAccess
4.  AmazonVPCFullAccess

## Set up AWS CLI configuration on your EC2 Instance or Laptop
### Run 
```bash
aws configure
```
-you will prompted to enter the following
```bash
AWS Access Key ID [****************NE5G]:
AWS Secret Access Key [****************FJE3]:
Default region name [us-east-1]:
Default output format [table]:
```
You need to enter the requested AWS Access Key ID, AWS Secret Access Key, Default region name, Default output format based on your IAM user.
