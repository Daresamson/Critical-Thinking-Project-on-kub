# Critical-Thinking-Project-on-kub( THE SCREENSHOT IS BELOW UPLOADED AFTER EXPLAINING THE STEP OF ACHIEVING THIS PROJECT)

# 📘 README: Multi-Namespace Kubernetes Application on AWS EKS with Dashboard

## 🧠 Project Title
Deploying a Multi-Namespace Application on AWS EKS with Kubernetes Dashboard

---

## 📝 Objective
To set up an AWS EKS cluster with 2 worker nodes, install and access the Kubernetes Dashboard, create 5 namespaces, and deploy one NGINX pod per namespace. This README documents the step-by-step process, including challenges I faced and how I resolved them.

---

## 🛠️ Tools Used
- AWS CLI
- eksctl
- kubectl
- Helm
- Ubuntu EC2 Instance (AWS)

---

## 🔧 Step-by-Step Setup

### Step 1: Prepare the EC2 Instance
1. I launched an Ubuntu EC2 instance (t3.large) to allow for better performance.
2. Updated the packages:
```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install Required Tools
#### AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

#### eksctl
```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

#### kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

#### Helm
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## 🔐 AWS Configuration and IAM Challenges

I configured AWS CLI using:
```bash
aws configure
```

Initially, I attached multiple AWS managed policies to my user like:
- AmazonEKSClusterPolicy
- AmazonEKSServicePolicy
- AmazonEC2FullAccess
- IAMFullAccess

However, I kept getting the error:
```
AccessDeniedException: User is not authorized to perform: eks:DescribeClusterVersions
```

### ✅ Solution:
I created a **custom inline IAM policy** with all necessary permissions including:
- `eks:*`
- `ec2:*`
- `cloudformation:*`
- `iam:PassRole`

Once this inline policy was added directly to my IAM user, I was finally able to create the cluster.

---

## ☸️ Create the EKS Cluster
```bash
eksctl create cluster \
  --name multi-ns-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 2 \
  --managed
```

---

## 📊 Install Kubernetes Dashboard
### Add Helm Repo and Install Dashboard
```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm repo update
helm install k8s-dashboard kubernetes-dashboard/kubernetes-dashboard \
  --namespace kubernetes-dashboard \
  --create-namespace
```

### Create Admin Access Token
```bash
kubectl create serviceaccount dashboard-admin-sa -n kubernetes-dashboard
kubectl create clusterrolebinding dashboard-admin-sa \
  --clusterrole=cluster-admin \
  --serviceaccount=kubernetes-dashboard:dashboard-admin-sa
kubectl -n kubernetes-dashboard create token dashboard-admin-sa
```

### Accessing the Dashboard
Ran:
```bash
kubectl proxy
```
But I couldn’t access the dashboard link in my browser.

### ✅ Solution:
I opened a new terminal session (another CLI window), reconnected to the same EC2 instance, and ran `kubectl proxy` again. That worked, and I was able to access the dashboard using:
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

---

## 📂 Create Namespaces and Deploy Pods
### Namespaces
```bash
for ns in dev test staging prod monitoring; do
  kubectl create namespace $ns
done
```

### Deploy NGINX Pods
```bash
for ns in dev test staging prod monitoring; do
  kubectl run nginx --image=nginx --namespace=$ns
done
```

### Verify Pods
```bash
kubectl get pods --all-namespaces
```

---

## 🧭 Monitoring via Dashboard
- I logged in using the generated service account token.
- Navigated through each namespace and confirmed that NGINX pods were running.
- Viewed node metrics and pod logs directly in the dashboard.

---

## 🧠 Lessons Learned
- Inline IAM policies can solve permission issues better than managed policies.
- Helm makes deploying applications like the Kubernetes Dashboard very simple.
- Kubernetes namespaces allow logical isolation of workloads.
- Sometimes you need a second terminal or session for `kubectl proxy` to work properly.

---

## 📌 Summary
| Resource         | Description                  |
|------------------|------------------------------|
| Cluster Name     | `multi-ns-cluster`           |
| Region           | `us-east-1`                  |
| Node Count       | 2 `t3.medium` worker nodes   |
| Namespaces       | dev, test, staging, prod, monitoring |
| Pods Deployed    | 5 (1 NGINX in each namespace)|
| Dashboard Access | via `kubectl proxy`          |

---

This concludes my step-by-step deployment and the journey through the issues and successes of this project.



![Screenshot (159)](https://github.com/user-attachments/assets/7a61bc37-ddc7-46a3-a7c6-6ad1694d23fb)
![Screenshot (160)](https://github.com/user-attachments/assets/1ba4a8c8-fe6c-4479-9141-db2fb16b646f)
![Screenshot (162)](https://github.com/user-attachments/assets/8122b30f-9e7f-41cc-a36a-6ab44d9fb185)
![Screenshot (163)](https://github.com/user-attachments/assets/0a6c40a1-fc39-4a43-9f35-a3b336ca18a5)
![Screenshot (164)](https://github.com/user-attachments/assets/bae97daa-75cc-4346-ab7b-82f76ac3f6cd)
![Screenshot (165)](https://github.com/user-attachments/assets/75c2545f-fd95-441b-b29c-f02a23770fc2)
![Screenshot (166)](https://github.com/user-attachments/assets/70312876-5446-4a5f-8fd5-dd9e012cc4cf)
![Screenshot (167)](https://github.com/user-attachments/assets/c0a3d9bd-171e-4689-9bc7-04d1ebb836be)
![Screenshot (168)](https://github.com/user-attachments/assets/b0870ce9-dd1f-49d8-8f92-e8108bbd6543)
![Screenshot (174)](https://github.com/user-attachments/assets/46b45bd1-2579-447e-8859-23d50925276a)
![Screenshot (175)](https://github.com/user-attachments/assets/1ae8e099-ace0-44dc-bf35-8de3f7215489)
![Screenshot (176)](https://github.com/user-attachments/assets/e3840e55-c63f-43e6-a8cb-d0767b9f4f5f)

