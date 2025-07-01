# KOPS-Setup
Using Kops (Kubernetes Operations) I'm setting up Kubernetes on AWS for production (Its a demo work)
## 🚀 Our main motive is to setup Kubernete using KOPS on EC2
### let me drive you into the prerequisites✅
Create an EC2 instance with required **dependencies** :
1. **Phython3**
2. **AWS CLI**
3. **Kubectl**

## 📦 Installation

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

```bash
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
