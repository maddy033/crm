# 🚀 School CRM DevOps Deployment on AWS EKS

A complete end-to-end DevOps project where a **School CRM application** is containerized using Docker, pushed to Docker Hub, deployed on **AWS EKS**, connected with PostgreSQL, and monitored using **Prometheus + Grafana**.

> Special thanks to **Shubham Pal** for the GitHub repository code used for this learning project.

---

## 📌 Project Overview

This project demonstrates how to deploy a real full-stack application on Kubernetes using AWS EKS.

The application contains:

- **Frontend:** React + Vite + Nginx
- **Backend:** Node.js + Express
- **Database:** PostgreSQL
- **ORM:** Prisma
- **Container Registry:** Docker Hub
- **Orchestration:** Kubernetes on AWS EKS
- **Monitoring:** Prometheus + Grafana
- **Package Manager:** Helm

---

## 🧭 Architecture

```text
User Browser
    |
    v
AWS Load Balancer
    |
    v
Frontend Service
    |
    v
Frontend Pod - React/Nginx
    |
    v
Backend Load Balancer / Backend Service
    |
    v
Backend Pod - Node.js/Express
    |
    v
PostgreSQL Service
    |
    v
PostgreSQL Pod
```

Monitoring:

```text
EKS Worker Nodes / Pods
        |
        v
Prometheus
        |
        v
Grafana Dashboard
```

---

## 📸 Project Screenshots

Create a folder named `screenshots/` and upload your screenshots.

Recommended names:

```text
screenshots/
├── 01-eks-cluster-ready.png
├── 02-kubectl-get-nodes.png
├── 03-docker-images.png
├── 04-dockerhub-images.png
├── 05-kubernetes-pods-running.png
├── 06-loadbalancer-url.png
├── 07-school-crm-login.png
├── 08-admin-dashboard.png
├── 09-prometheus-pods.png
├── 10-grafana-dashboard.png
```

Example:

```md
![EKS Cluster Ready](screenshots/01-eks-cluster-ready.png)
```

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Git & GitHub | Source code management |
| Docker | Containerization |
| Docker Hub | Image registry |
| AWS CLI | AWS resource management |
| eksctl | EKS cluster creation |
| kubectl | Kubernetes management |
| Helm | Kubernetes package manager |
| AWS EKS | Managed Kubernetes |
| PostgreSQL | Application database |
| Prisma | ORM and DB schema management |
| Prometheus | Metrics collection |
| Grafana | Monitoring dashboards |

---

## ✅ Prerequisites

- AWS account
- EC2 admin server
- IAM role attached to EC2
- Docker Hub account
- GitHub account
- Basic Linux, Docker, Kubernetes, and AWS knowledge

---

## 🖥️ EC2 Admin Server Setup

Recommended EC2:

```text
OS: Ubuntu 24.04 LTS
Instance Type: t3.small / t3.medium
Disk: 30 GB
Region: ap-south-1
IAM Role: AdministratorAccess for learning
```

Connect to EC2:

```bash
ssh -i your-key.pem ubuntu@<EC2-PUBLIC-IP>
```

Verify IAM role:

```bash
aws sts get-caller-identity
```

Expected output:

```text
arn:aws:sts::<ACCOUNT_ID>:assumed-role/<ROLE_NAME>/<INSTANCE_ID>
```

---

## 📦 Install Required Packages

### Update system

```bash
sudo apt update -y
sudo apt install unzip curl git -y
```

### Install AWS CLI v2

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### Install Docker

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo systemctl status docker --no-pager
sudo usermod -aG docker ubuntu
newgrp docker
docker --version
```

### Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

### Install eksctl

```bash
curl --silent --location \
"https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" \
| tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

---

## ☸️ Create AWS EKS Cluster

Check available AZs:

```bash
aws ec2 describe-availability-zones \
  --region ap-south-1 \
  --query "AvailabilityZones[].ZoneName" \
  --output table
```

Check Free Tier eligible instance types:

```bash
aws ec2 describe-instance-types \
  --filters Name=free-tier-eligible,Values=true \
  --query 'InstanceTypes[*].InstanceType' \
  --output table
```

Create EKS cluster:

```bash
eksctl create cluster \
  --name students-eks \
  --region ap-south-1 \
  --nodegroup-name workers \
  --node-type t3.small \
  --nodes 2 \
  --managed
```

Verify:

```bash
kubectl get nodes -o wide
kubectl get pods -A
```

Expected:

```text
2 worker nodes should be Ready
```

---

## ⚠️ Issue Faced: Nodegroup Creation Failed

Initial nodegroup creation with `t3.medium` failed.

Error:

```text
Could not launch On-Demand Instances.
InvalidParameterCombination - The specified instance type is not eligible for Free Tier.
```

Root cause:

```text
t3.medium was not allowed under the account's Free Tier eligible instance list.
```

Resolution:

```bash
aws ec2 describe-instance-types \
  --filters Name=free-tier-eligible,Values=true \
  --query 'InstanceTypes[*].InstanceType' \
  --output table
```

Then recreated cluster using:

```text
t3.small
```

---

## 📥 Clone the Project

```bash
cd ~
git clone git@github.com:maddy033/crm.git
cd crm
```

Project structure:

```text
crm/
├── backend/
├── frontend/
├── mobile/
├── docker-compose.yml
└── package-lock.json
```

---

## 🐳 Build Docker Images

Backend:

```bash
cd ~/crm/backend
docker build -t crm-backend:v1 .
```

Frontend:

```bash
cd ~/crm/frontend
docker build -t crm-frontend:v1 .
```

Verify:

```bash
docker images
```

---

## 🧪 Test Backend Locally with PostgreSQL

Run PostgreSQL:

```bash
docker run -d \
  --name postgres-test \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=1234 \
  -e POSTGRES_DB=school_system \
  -p 5432:5432 \
  postgres:16
```

Create network:

```bash
docker network create crm-net
docker network connect crm-net postgres-test
```

Run backend:

```bash
docker run -d \
  --name backend-test \
  --network crm-net \
  -p 5000:5000 \
  -e DATABASE_URL="postgresql://postgres:1234@postgres-test:5432/school_system?schema=public" \
  -e JWT_SECRET="testsecret" \
  -e JWT_REFRESH_SECRET="testrefresh" \
  -e PORT=5000 \
  crm-backend:v1
```

Check logs:

```bash
docker logs backend-test
```

Expected:

```text
Server is running on port 5000
```

---

## 🧬 Apply Prisma DB Schema Locally

Check tables:

```bash
docker exec -it postgres-test psql -U postgres -d school_system -c "\dt"
```

Before Prisma:

```text
Did not find any relations.
```

Run Prisma:

```bash
docker exec -it backend-test sh
npx prisma db push
exit
```

Verify:

```bash
docker exec -it postgres-test psql -U postgres -d school_system -c "\dt"
```

Expected tables:

```text
User
Student
Staff
School
Class
AttendanceStudent
...
```

---

## 🛠️ Fix Frontend API URL

Original frontend API config:

```ts
baseURL: 'http://localhost:5000/api'
```

This fails in Kubernetes because browser `localhost` means the user's laptop.

Update:

```text
frontend/src/services/api.ts
```

Change:

```ts
baseURL: 'http://localhost:5000/api',
```

To:

```ts
baseURL: import.meta.env.VITE_API_URL,
```

Create `.env`:

```bash
cd ~/crm/frontend
cat > .env <<'EOF'
VITE_API_URL=http://localhost:5000/api
EOF
```

Rebuild:

```bash
docker build -t crm-frontend:v2 .
```

---

## 📤 Push Images to Docker Hub

Docker Hub repo:

```text
mandar0333/crm
```

Login:

```bash
docker login
```

Tag:

```bash
docker tag crm-backend:v1 mandar0333/crm:backend-v1
docker tag crm-frontend:v2 mandar0333/crm:frontend-v2
```

Push:

```bash
docker push mandar0333/crm:backend-v1
docker push mandar0333/crm:frontend-v2
```

---

## ☸️ Kubernetes Deployment

Create folder:

```bash
cd ~/crm
mkdir -p k8s
cd k8s
```

### PostgreSQL Deployment

```bash
cat > postgres-deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:16
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_USER
          value: postgres
        - name: POSTGRES_PASSWORD
          value: "1234"
        - name: POSTGRES_DB
          value: school_system
EOF
```

Apply:

```bash
kubectl apply -f postgres-deployment.yaml
kubectl get pods
```

### PostgreSQL Service

```bash
cat > postgres-service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  type: ClusterIP
EOF
```

Apply:

```bash
kubectl apply -f postgres-service.yaml
kubectl get svc
```

### Backend Deployment

```bash
cat > backend-deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: crm-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crm-backend
  template:
    metadata:
      labels:
        app: crm-backend
    spec:
      containers:
      - name: crm-backend
        image: mandar0333/crm:backend-v1
        ports:
        - containerPort: 5000
        env:
        - name: DATABASE_URL
          value: "postgresql://postgres:1234@postgres-service:5432/school_system?schema=public"
        - name: JWT_SECRET
          value: "supersafejwtkey123"
        - name: JWT_REFRESH_SECRET
          value: "supersaferefreshkey456"
        - name: PORT
          value: "5000"
EOF
```

Apply:

```bash
kubectl apply -f backend-deployment.yaml
kubectl get pods
kubectl logs deployment/crm-backend
```

### Backend Internal Service

```bash
cat > backend-service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: crm-backend
  ports:
  - port: 5000
    targetPort: 5000
  type: ClusterIP
EOF

kubectl apply -f backend-service.yaml
```

### Backend LoadBalancer Service

```bash
cat > backend-lb-service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: backend-lb-service
spec:
  type: LoadBalancer
  selector:
    app: crm-backend
  ports:
  - port: 5000
    targetPort: 5000
EOF

kubectl apply -f backend-lb-service.yaml
kubectl get svc
```

Copy backend ELB DNS.

---

## Rebuild Frontend with Backend ELB URL

```bash
cd ~/crm/frontend
cat > .env <<'EOF'
VITE_API_URL=http://<BACKEND-ELB-DNS>:5000/api
EOF

docker build -t crm-frontend:v3 .
docker tag crm-frontend:v3 mandar0333/crm:frontend-v3
docker push mandar0333/crm:frontend-v3
```

### Frontend Deployment

```bash
cat > ~/crm/k8s/frontend-deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: crm-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crm-frontend
  template:
    metadata:
      labels:
        app: crm-frontend
    spec:
      containers:
      - name: crm-frontend
        image: mandar0333/crm:frontend-v3
        ports:
        - containerPort: 80
EOF

kubectl apply -f ~/crm/k8s/frontend-deployment.yaml
kubectl get pods
```

### Frontend Service

```bash
cat > ~/crm/k8s/frontend-service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  selector:
    app: crm-frontend
  ports:
  - port: 80
    targetPort: 80
EOF

kubectl apply -f ~/crm/k8s/frontend-service.yaml
kubectl get svc
```

Open:

```text
http://<FRONTEND-ELB-DNS>
```

---

## 🧬 Create DB Tables in EKS PostgreSQL

```bash
kubectl exec -it deployment/crm-backend -- npx prisma db push
kubectl exec -it deployment/postgres -- psql -U postgres -d school_system -c "\dt"
```

---

## 👤 Create Demo Admin User

Generate password hash:

```bash
kubectl exec -it deployment/crm-backend -- node -e "const bcrypt=require('bcryptjs'); bcrypt.hash('admin123',10).then(console.log)"
```

Open PostgreSQL:

```bash
kubectl exec -it deployment/postgres -- psql -U postgres -d school_system
```

Insert admin user:

```sql
INSERT INTO "User" ("id","email","password","role","createdAt","updatedAt")
VALUES (gen_random_uuid(),'admin@school.com','<PASTE_BCRYPT_HASH_HERE>','ADMIN',now(),now());

SELECT email, role FROM "User";

\q
```

Login:

```text
Email: admin@school.com
Password: admin123
```

---

## 🔍 Verify Students in Database

```bash
kubectl exec -it deployment/postgres -- psql -U postgres -d school_system -c 'SELECT id,email,role,"createdAt" FROM "User" ORDER BY "createdAt" DESC;'
```

---

## 📊 Install Prometheus and Grafana

Check resource usage:

```bash
kubectl top nodes
kubectl top pods -A
```

Add repos:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

Create namespace:

```bash
kubectl create namespace monitoring
```

Install Prometheus:

```bash
helm install prometheus prometheus-community/prometheus \
  --namespace monitoring \
  --set server.persistentVolume.enabled=false \
  --set alertmanager.enabled=false \
  --set pushgateway.enabled=false
```

Check:

```bash
kubectl get pods -n monitoring
```

Install Grafana:

```bash
helm install grafana grafana/grafana \
  --namespace monitoring \
  --set persistence.enabled=false \
  --set service.type=LoadBalancer
```

Check:

```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

Get password:

```bash
kubectl get secret grafana -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 --decode; echo
```

Login:

```text
Username: admin
Password: <password-from-command>
```

Prometheus datasource URL:

```text
http://prometheus-server.monitoring.svc.cluster.local
```

Import dashboards:

```text
Node Exporter Full: 1860
Kubernetes Cluster Dashboard: 15757
```

---

## 🧯 Troubleshooting Summary

### 1. EKS Nodegroup Creation Failed

Issue:

```text
t3.medium not eligible for Free Tier
```

Fix:

```text
Used t3.small
```

### 2. Frontend Login Failed

Issue:

```text
Frontend was calling http://localhost:5000/api
```

Fix:

```ts
baseURL: import.meta.env.VITE_API_URL
```

Then rebuilt frontend with backend ELB DNS.

### 3. PostgreSQL Tables Missing

Issue:

```text
relation "User" does not exist
```

Fix:

```bash
kubectl exec -it deployment/crm-backend -- npx prisma db push
```

### 4. ImagePullBackOff

Issue:

```text
Kubernetes deployment was updated before Docker image tag was pushed.
```

Fix:

```bash
docker push mandar0333/crm:frontend-v3
kubectl rollout status deployment/crm-frontend
```

### 5. Grafana Prometheus URL Issue

Issue:

```text
Used http://localhost:9090
```

Fix:

```text
http://prometheus-server.monitoring.svc.cluster.local
```

---

## 🧹 Cleanup / Destroy AWS Resources

Delete monitoring:

```bash
kubectl delete namespace monitoring
```

Delete LoadBalancer services:

```bash
kubectl delete svc frontend-service backend-lb-service
```

Delete EKS cluster:

```bash
eksctl delete cluster \
  --name students-eks \
  --region ap-south-1
```

Verify:

```bash
eksctl get cluster --region ap-south-1
aws eks list-clusters --region ap-south-1
aws elbv2 describe-load-balancers --region ap-south-1
```

Expected:

```text
No clusters found
LoadBalancers: []
```

Check running EC2:

```bash
aws ec2 describe-instances \
  --region ap-south-1 \
  --filters Name=instance-state-name,Values=running
```

Stop admin EC2 if no longer needed:

```bash
aws ec2 stop-instances \
  --region ap-south-1 \
  --instance-ids <INSTANCE_ID>
```

---

## 🎯 Interview Talking Points

You can explain this project like this:

> I deployed a full-stack School CRM application on AWS EKS. I containerized the frontend and backend using Docker, pushed images to Docker Hub, created an EKS cluster using eksctl, deployed PostgreSQL, backend, and frontend using Kubernetes Deployments and Services, exposed the application using AWS Load Balancers, handled Prisma schema creation, fixed frontend-backend API communication, and implemented monitoring using Prometheus and Grafana.

Key concepts learned:

- Docker image lifecycle
- Kubernetes Deployments and Services
- ClusterIP vs LoadBalancer
- AWS EKS managed node groups
- PostgreSQL in Kubernetes
- Prisma schema management
- Prometheus scraping
- Grafana dashboards
- Real troubleshooting workflow

---

## 📌 Future Enhancements

- Add GitHub Actions CI/CD
- Use Kubernetes Secrets instead of plain env values
- Use PersistentVolumeClaim for PostgreSQL
- Use Ingress instead of separate LoadBalancers
- Add AWS Load Balancer Controller
- Add Horizontal Pod Autoscaler
- Add Terraform for infrastructure automation
- Move database to Amazon RDS
- Add Trivy image scanning
- Add Argo CD GitOps deployment

---

## 🙌 Credits

Thanks to **Shubham Pal** for the School CRM application source code used in this DevOps learning project.

---

## 👨‍💻 Author

Maintained by **Mandar / Maddy**

Docker Hub:

```text
mandar0333
```

GitHub:

```text
maddy033
```
