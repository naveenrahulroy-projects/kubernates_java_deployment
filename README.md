#  Docker-Java-Kubernetes Project

This project demonstrates how to containerize and deploy a set of Java microservices (Shopfront, Product Catalogue, and Stock Manager) using **Docker** and **Kubernetes (Minikube or EKS)**.

---

## 📦 Prerequisites

- Java 8+
- Maven
- Docker
- Minikube (for local deployment)
- AWS CLI & eksctl (for EKS deployment)
- kubectl

---

##  Local Setup Using Minikube

### 🔧 Install Minikube

Follow official instructions:  
🔗 https://minikube.sigs.k8s.io/docs/start/

###  Install kubectl

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.20.4/2021-04-12/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin
cp ./kubectl $HOME/bin/kubectl
export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
kubectl version --short --client
```

### 🐳 Install Docker

```bash
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
```

---

## ☁️ AWS EKS Setup

### Step 1: Launch EC2 Instance
- Use **t2.medium** type
- Attach **IAM Role** with **AdministratorAccess**

### Step 2: Install kubectl

```bash
curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin
cp ./kubectl $HOME/bin/kubectl
export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
kubectl version --short --client
```

### Step 3: Install eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/bin
eksctl version
```

### Step 4: Create EKS Cluster (Master Only)

```bash
eksctl create cluster --name=eksdemo \
  --region=us-west-1 \
  --zones=us-west-1b,us-west-1c \
  --without-nodegroup
```

### Step 5: Associate IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --region us-west-1 \
  --cluster eksdemo \
  --approve
```

### Step 6: Create Worker Nodes

```bash
eksctl create nodegroup --cluster=eksdemo \
  --region=us-west-1 \
  --name=eksdemo-ng-public \
  --node-type=t2.medium \
  --nodes=2 \
  --nodes-min=2 \
  --nodes-max=4 \
  --node-volume-size=10 \
  --ssh-access \
  --ssh-public-key=Praveen-test \
  --managed \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --appmesh-access \
  --alb-ingress-access
```

### ⛔ Optional Cleanup Commands

```bash
eksctl delete nodegroup --cluster=eksdemo --region=us-west-1 --name=eksdemo-ng-public
eksctl delete cluster --name=eksdemo --region=us-west-1
```

---

## 🧪 Hands-On: Deploy Java Applications on Kubernetes

### 1️⃣ Build Maven Projects

```bash
mvn clean install -DskipTests
```

### 2️⃣ Create Docker Hub Account (if not done)

- https://hub.docker.com/

### 3️⃣ Build Docker Images

```bash
docker build -t naveenrroy/shopfront:latest ./shopfront
docker build -t naveenrroy/productcatalogue:latest ./productcatalogue
docker build -t naveenrroy/stockmanager:latest ./stockmanager
```

### 4️⃣ Push Docker Images to Docker Hub

```bash
docker push naveenrroy/shopfront:latest
docker push naveenrroy/productcatalogue:latest
docker push naveenrroy/stockmanager:latest
```

### 5️⃣ Apply Kubernetes Manifests

```bash
kubectl apply -f kubernetes/shopfront-service.yaml
kubectl apply -f kubernetes/productcatalogue-service.yaml
kubectl apply -f kubernetes/stockmanager-service.yaml
```

### 6️⃣ Access Services via Minikube

```bash
minikube service shopfront
minikube service productcatalogue
minikube service stockmanager
```

### 7️⃣ Test the Services

Open the service URLs in your browser.

---

## 📚 Service Flow and API Endpoints

Deployment Order:
```
1. Shopfront
2. Product Catalogue
3. Stock Manager
```

| Service            | Endpoint         |
|--------------------|------------------|
| Product Catalogue  | `/products`      |
| Stock Manager      | `/stocks`        |

---

## 📌 Notes

- Ensure Docker is running before building or pushing images.
- Replace `naveenrroy` with your own Docker Hub username if different.
- All Kubernetes YAML files are located in the `kubernetes/` folder.
