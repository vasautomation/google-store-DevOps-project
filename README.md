![Project Image](https://i.ibb.co/t9WR8c8/drawing.png)
#!/bin/bash

# --- 1. EKSCTL AND KUBECTL INSTALLATION ---


# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | \
[cite_start]tar xz -C /tmp [cite: 1]
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

# Install kubectl
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

# --- 2. SYSTEM TOOLS INSTALLATION (Docker and Git) ---

echo "--- 2. Installing Docker and Git ---"

yum install docker -y
systemctl start docker 
yum install git -y

# --- 3. EKS CLUSTER CREATION ---

echo "--- 3. Creating EKS Cluster named 'google' ---"

eksctl create cluster --name google \
   --region ap-northeast-1 \
   --node-type c7i-flex.large \
   --nodes-min 2 \
   --nodes-max 2

# --- 4. KUBERNETES INGRESS SETUP ---

echo "--- 4. Installing Ingress Controller ---"

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

echo "--- 4.1 Verifying Ingress ---"
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
kubectl get ingress -n google

# --- 5. MARIADB INSTALLATION ---

echo "--- 5. Installing MariaDB 10.5 ---"

sudo yum update -y
sudo dnf install -y mariadb105

# Note: The database creation commands (SQL) require interactive shell access to MariaDB.
# These cannot be easily run in a single script block without piping or expecting a password.
# They are listed here for reference.
echo ""
echo "!!! Manual Step Required: Run SQL commands in MariaDB client !!!"
echo "CREATE DATABASE IF NOT EXISTS cloud;
[cite_start]USE cloud; [cite: 3]
DROP TABLE IF EXISTS users;
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    full_name VARCHAR(150) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
[cite_start]exit [cite: 4]"
echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!"
echo ""

# --- 6. ARGOCD INSTALLATION AND CONFIGURATION ---

echo "--- 6. Installing ArgoCD ---"

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Expose ArgoCD Server
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl get svc -n argocd

# Create custom namespace
echo "--- 6.1 Creating custom namespace 'google' ---"
kubectl create namespace google

# Retrieve ArgoCD initial admin password
echo "--- 6.2 Retrieving ArgoCD initial admin password ---"
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" |
[cite_start]base64 -d [cite: 5]

echo ""
echo "--- Installation Script Complete ---"


