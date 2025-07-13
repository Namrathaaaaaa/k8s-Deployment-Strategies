# Kubernetes Deployment Strategies 🚀

Learn and practice different Kubernetes deployment strategies with hands-on examples using EC2 instances and Kind clusters.

## 📋 Quick Navigation

- [Setup](#setup)
- [Deployment Strategies](#deployment-strategies)
- [Getting Started](#getting-started)

## 🔧 Prerequisites

- **kubectl** CLI tool
- **Docker** installed
- **AWS Account** (for EC2 setup)

## 🚀 Setup

### Option 1: Kind Cluster (Local)

```bash
# Install Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind && sudo mv ./kind /usr/local/bin/

# Setup cluster
cd kind-cluster && chmod +x install.sh && ./install.sh
```

### Option 2: EC2 Instance

```bash
chmod 400 k8s-deployment-s.pem
ssh -i k8s-deployment-s.pem ubuntu@your-ec2-ip
```

## 📦 Deployment Strategies

### 1. Recreate Strategy

**Location**: `Deployment_Strategies/Recreate-deployment/`

- **Use Case**: Development environments
- **Pros**: Simple, resource efficient
- **Cons**: Downtime during deployment

```bash
cd Deployment_Strategies/Recreate-deployment/
kubectl apply -f .
kubectl get pods -n recreate-ns
```

### 2. Rolling Update Strategy

**Location**: `Deployment_Strategies/Rolling-Update-Deployment/`

- **Use Case**: Production environments
- **Pros**: Zero downtime, built-in rollback
- **Cons**: Mixed versions during rollout

```bash
cd Deployment_Strategies/Rolling-Update-Deployment/
kubectl apply -f .
kubectl rollout status deployment/rolling-deployment -n rolling-ns
```

### 3. Blue-Green Deployment

**Location**: `Deployment_Strategies/Blue-green-deployment/`

- **Use Case**: Critical applications
- **Pros**: Instant rollback, zero downtime
- **Cons**: Requires double resources

```bash
cd Deployment_Strategies/Blue-green-deployment/
kubectl apply -f blue-green-ns.yaml
kubectl apply -f online-shop-without-footer-blue-deployment.yaml
kubectl apply -f online-shop-green-deployment.yaml

# Switch traffic to Green
kubectl patch service online-shop-service -n blue-green-ns \
  -p '{"spec":{"selector":{"version":"green"}}}'
```

### 4. Canary Deployment

**Location**: `Deployment_Strategies/Canary-deployment/` & `Simple-Canary-Example/`

- **Use Case**: Risk mitigation, gradual rollouts
- **Pros**: Real user feedback, risk mitigation
- **Cons**: Complex traffic management

```bash
cd Deployment_Strategies/Canary-deployment/
kubectl apply -f canary-namespace.yaml
kubectl apply -f canary-v1-deployment.yaml
kubectl apply -f canary-v2-deployment.yaml
kubectl apply -f canary-combined-service.yaml
```

## 🏃 Getting Started

```bash
# 1. Clone repository
git clone https://github.com/Namrathaaaaaa/k8s-Deployment-Strategies.git
cd k8s-DS

# 2. Setup Kind cluster
cd kind-cluster && ./install.sh

# 3. Try deployment strategies
cd ../Deployment_Strategies/Recreate-deployment/
kubectl apply -f .
```

## 📁 Project Structure

```
k8s-DS/
├── Deployment_Strategies/
│   ├── Blue-green-deployment/
│   ├── Canary-deployment/
│   ├── Recreate-deployment/
│   ├── Rolling-Update-Deployment/
│   └── Simple-Canary-Example/
├── kind-cluster/
│   ├── install.sh
│   └── kind-config.yaml
└── k8s-deployment-s.pem
```

## 🔧 Common Commands

```bash
# Monitor deployments
kubectl get pods -A
kubectl get deployments -A

# Check logs
kubectl logs <pod-name> -n <namespace>

# Rollback deployment
kubectl rollout undo deployment/<deployment-name> -n <namespace>

# Clean up
kubectl delete namespace <namespace>
```

## 🏆 Strategy Selection Guide

| Environment      | Strategy       | Reason              |
| ---------------- | -------------- | ------------------- |
| Development      | Recreate       | Simple, downtime OK |
| Production       | Rolling Update | Zero downtime       |
| Critical Systems | Blue-Green     | Instant rollback    |
| New Features     | Canary         | Risk mitigation     |

---

**Happy Learning! 🚀**

For issues or questions, open a GitHub issue.
