# Kubernetes Deployment Strategies 🚀

A comprehensive hands-on project demonstrating various Kubernetes deployment strategies with real-world examples, EC2 instances, and Kind clusters.

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)

## 📋 Table of Contents

- [Overview](#overview)
- [What Are Deployment Strategies?](#what-are-deployment-strategies)
- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Deployment Strategies](#deployment-strategies)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## 🎯 Overview

This project provides hands-on experience with different Kubernetes deployment strategies. Each strategy is implemented with practical examples, YAML configurations, and detailed explanations to help you understand when and how to use them in real-world scenarios.

### What You'll Learn

- ✅ **Recreate Strategy** - Simple deployments with downtime
- ✅ **Rolling Update Strategy** - Zero-downtime gradual updates
- ✅ **Blue-Green Deployment** - Instant switching between environments
- ✅ **Canary Deployment** - Risk-free gradual rollouts
- ✅ **Progressive Delivery** - Automated metric-based deployments
- ✅ **A/B Testing** - Feature testing with user segmentation
- ✅ **Shadow Deployment** - Testing with production traffic

## 🤔 What Are Deployment Strategies?

**Deployment** is the process of making an application available for use by the audience.

**Deployment Strategies** are techniques for changing or upgrading a running application from one version to another.

### Why Do We Need Deployment Strategies?

- 🚀 **Zero Downtime** - Keep services running during updates
- ⏰ **Reduce Time to Market** - Faster and safer releases
- 🔄 **Faster Rollback** - Quick recovery from issues
- 📈 **More Frequency** - Enable continuous deployment

## 🔧 Prerequisites

Before starting this project, ensure you have:

### Required Tools
- **AWS Account** with EC2 access
- **kubectl** CLI tool installed
- **Docker** installed locally
- **Git** for repository management
- **curl** for testing endpoints

### Knowledge Requirements
- Basic understanding of Kubernetes concepts (Pods, Services, Deployments)
- Familiarity with YAML configuration files
- Basic command-line experience
- Understanding of containerization concepts

### Installation Commands

```bash
# Install kubectl (Linux)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Install kubectl (macOS)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Verify installation
kubectl version --client
```

## 🚀 Environment Setup

### Option 1: Kind Cluster (Recommended for Learning)

Kind (Kubernetes in Docker) is perfect for local development and learning.

```bash
# Install Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Use the provided Kind configuration
cd kind-cluster
chmod +x install.sh
./install.sh

# Verify cluster
kubectl cluster-info
kubectl get nodes
```

### Option 2: EC2 Instance Setup

For a production-like environment, use the provided EC2 setup.

```bash
# Use the provided PEM file to connect to EC2
chmod 400 k8s-deployment-s.pem
ssh -i k8s-deployment-s.pem ubuntu@your-ec2-ip

# Follow the installation script in kind-cluster/install.sh
# Adapt it for your EC2 environment
```

## 📦 Deployment Strategies

### 1. 🔄 Recreate Strategy

**Location**: `Deployment_Strategies/Recreate-deployment/`

**When to Use**: Development environments, simple applications where brief downtime is acceptable.

#### How It Works
- Terminates all existing pods
- Creates new pods with updated configuration
- Brief downtime during the transition

#### Pros & Cons
| ✅ Pros | ❌ Cons |
|---------|---------|
| Simple implementation | Downtime during deployment |
| Complete application state renewal | Not suitable for production |
| Resource efficient | User experience interruption |

#### Demo Commands
```bash
cd Deployment_Strategies/Recreate-deployment/

# Apply the deployment
kubectl apply -f recreate-namespace.yml
kubectl apply -f recreate-deployment.yml
kubectl apply -f recreate-svc.yml

# Check status
kubectl get pods -n recreate-ns
kubectl get svc -n recreate-ns

# Update and observe downtime
kubectl set image deployment/recreate-deployment nginx=nginx:1.21 -n recreate-ns
kubectl get pods -n recreate-ns -w
```

### 2. 🔄 Rolling Update Strategy

**Location**: `Deployment_Strategies/Rolling-Update-Deployment/`

**When to Use**: Production environments requiring zero downtime, default Kubernetes strategy.

#### How It Works
- Creates new pods gradually
- Terminates old pods as new ones become ready
- Maintains desired replica count throughout

#### Pros & Cons
| ✅ Pros | ❌ Cons |
|---------|---------|
| Zero downtime | Rollout can take time |
| Built-in rollback | Mixed versions during rollout |
| Gradual risk exposure | No traffic control |

#### Demo Commands
```bash
cd Deployment_Strategies/Rolling-Update-Deployment/

# Apply the deployment
kubectl apply -f rolling-namespace.yml
kubectl apply -f rolling-update-deployment.yaml
kubectl apply -f rolling-update-svc.yml

# Monitor rolling update
kubectl rollout status deployment/rolling-deployment -n rolling-ns

# Update deployment
kubectl set image deployment/rolling-deployment nginx=nginx:1.21 -n rolling-ns

# Rollback if needed
kubectl rollout undo deployment/rolling-deployment -n rolling-ns
```

### 3. 🔵🟢 Blue-Green Deployment

**Location**: `Deployment_Strategies/Blue-green-deployment/`

**When to Use**: Critical applications requiring instant rollback, production environments with sufficient resources.

#### How It Works
- Maintains two identical environments (Blue & Green)
- Deploys new version to inactive environment
- Instantly switches traffic between environments

#### Pros & Cons
| ✅ Pros | ❌ Cons |
|---------|---------|
| Instant rollout/rollback | Requires double resources |
| Zero downtime | Complex infrastructure |
| Complete testing before switch | Database synchronization challenges |

#### Demo Commands
```bash
cd Deployment_Strategies/Blue-green-deployment/

# Create namespace
kubectl apply -f blue-green-ns.yaml

# Deploy Blue environment (current)
kubectl apply -f online-shop-without-footer-blue-deployment.yaml

# Deploy Green environment (new)
kubectl apply -f online-shop-green-deployment.yaml

# Check both environments
kubectl get pods -n blue-green-ns
kubectl get svc -n blue-green-ns

# Switch traffic to Green
kubectl patch service online-shop-service -n blue-green-ns \
  -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback to Blue if needed
kubectl patch service online-shop-service -n blue-green-ns \
  -p '{"spec":{"selector":{"version":"blue"}}}'
```

### 4. 🐤 Canary Deployment

**Location**: `Deployment_Strategies/Canary-deployment/` & `Deployment_Strategies/Simple-Canary-Example/`

**When to Use**: Risk mitigation, gradual feature rollout, performance testing with real users.

#### How It Works
- Deploys new version to a small subset of users
- Gradually increases traffic to new version
- Monitors metrics and user feedback
- Complete rollout or rollback based on results

#### Pros & Cons
| ✅ Pros | ❌ Cons |
|---------|---------|
| Risk mitigation | Complex traffic management |
| Real user feedback | Requires monitoring setup |
| Gradual rollout | Longer deployment time |

#### Advanced Canary Demo
```bash
cd Deployment_Strategies/Canary-deployment/

# Create namespace
kubectl apply -f canary-namespace.yaml

# Deploy V1 (stable - 90% traffic)
kubectl apply -f canary-v1-deployment.yaml

# Deploy V2 (canary - 10% traffic)
kubectl apply -f canary-v2-deployment.yaml

# Setup service and ingress
kubectl apply -f canary-combined-service.yaml
kubectl apply -f ingress.yaml

# Monitor traffic distribution
kubectl get pods -n canary-ns -l app=online-shop

# Gradually increase canary traffic
kubectl scale deployment online-shop-canary -n canary-ns --replicas=3
```

#### Simple Canary Demo
```bash
cd Deployment_Strategies/Simple-Canary-Example/

# Create namespace
kubectl apply -f namespace.yaml

# Deploy Apache (stable)
kubectl apply -f apache-configmap.yaml
kubectl apply -f apache-deployment.yaml

# Deploy Nginx (canary)
kubectl apply -f nginx-configmap.yaml
kubectl apply -f nginx-deployment.yaml

# Setup service and ingress
kubectl apply -f canary-service.yaml
kubectl apply -f ingress.yaml

# Test traffic distribution
curl -H "Host: web.local" http://localhost
```

### 5. 📈 Progressive Delivery

**When to Use**: Automated deployments based on metrics, advanced CI/CD pipelines.

Progressive delivery extends canary deployments with automated decision-making based on metrics and SLOs.

#### Key Features
- Automated traffic shifting
- Metric-based rollback
- Integration with monitoring tools
- SLO-driven deployments

### 6. 🧪 A/B Testing

**When to Use**: Feature testing, user experience optimization, business metric validation.

A/B testing routes traffic based on user characteristics to test different features or UX changes.

#### Implementation Approaches
- Header-based routing
- Cookie-based segmentation
- Geographic routing
- User profile-based routing

### 7. 👥 Shadow Deployment

**When to Use**: Testing with production traffic without user impact, performance validation.

Shadow deployment duplicates production traffic to test new versions without affecting users.

#### Benefits
- Zero user impact
- Real traffic patterns
- Performance comparison
- Risk-free testing

## 📁 Project Structure

```
k8s-DS/
├── README.md                                    # This comprehensive guide
├── k8s-deployment-s.pem                        # EC2 SSH key
├── Deployment_Strategies/
│   ├── Blue-green-deployment/
│   │   ├── blue-green-ns.yaml                  # Namespace definition
│   │   ├── online-shop-green-deployment.yaml   # Green environment
│   │   ├── online-shop-without-footer-blue-deployment.yaml  # Blue environment
│   │   └── readme.md                           # Blue-Green specific guide
│   ├── Canary-deployment/
│   │   ├── canary-combined-service.yaml        # Service for traffic splitting
│   │   ├── canary-namespace.yaml               # Namespace definition
│   │   ├── canary-v1-deployment.yaml           # Stable version
│   │   ├── canary-v2-deployment.yaml           # Canary version
│   │   ├── ingress.yaml                        # Ingress configuration
│   │   └── readme.md                           # Canary specific guide
│   ├── Recreate-deployment/
│   │   ├── recreate-deployment.yml             # Recreate deployment config
│   │   ├── recreate-namespace.yml              # Namespace definition
│   │   ├── recreate-svc.yml                    # Service configuration
│   │   └── readme.md                           # Recreate specific guide
│   ├── Rolling-Update-Deployment/
│   │   ├── rolling-namespace.yml               # Namespace definition
│   │   ├── rolling-update-deployment.yaml      # Rolling update config
│   │   ├── rolling-update-svc.yml              # Service configuration
│   │   └── readme.md                           # Rolling update guide
│   └── Simple-Canary-Example/
│       ├── apache-configmap.yaml               # Apache configuration
│       ├── apache-deployment.yaml              # Apache deployment
│       ├── canary-service.yaml                 # Service for canary
│       ├── ingress.yaml                        # Ingress configuration
│       ├── namespace.yaml                      # Namespace definition
│       ├── nginx-configmap.yaml                # Nginx configuration
│       ├── nginx-deployment.yaml               # Nginx deployment
│       └── readme.md                           # Simple canary guide
└── kind-cluster/
    ├── dashboard-admin-user.yml                # Dashboard admin setup
    ├── install.sh                              # Cluster installation script
    └── kind-config.yaml                        # Kind cluster configuration
```

## 🏃 Getting Started

### Quick Start (5 minutes)

```bash
# 1. Clone the repository
git clone https://github.com/Namrathaaaaaa/k8s-Deployment-Strategies.git
cd k8s-DS

# 2. Set up Kind cluster
cd kind-cluster
chmod +x install.sh
./install.sh

# 3. Verify setup
kubectl cluster-info
kubectl get nodes

# 4. Try your first deployment strategy
cd ../Deployment_Strategies/Recreate-deployment/
kubectl apply -f .
kubectl get pods -n recreate-ns
```

### Learning Path

1. **Start with Recreate** - Understand basic concepts
2. **Move to Rolling Update** - Learn zero-downtime deployments
3. **Try Blue-Green** - Experience instant switching
4. **Explore Canary** - Practice gradual rollouts
5. **Advanced Strategies** - Progressive delivery, A/B testing, Shadow

### Common Commands

```bash
# Check cluster status
kubectl cluster-info
kubectl get nodes

# Monitor deployments
kubectl get deployments -A
kubectl get pods -A
kubectl get services -A

# Watch resources in real-time
kubectl get pods -w
kubectl get deployments -w

# Clean up resources
kubectl delete namespace your-namespace
kubectl delete -f your-deployment-file.yaml
```

## 🏆 Best Practices

### General Guidelines

1. **Always test in staging first**
2. **Use health checks and readiness probes**
3. **Monitor key metrics during deployments**
4. **Have rollback procedures ready**
5. **Use resource limits and requests**
6. **Implement proper logging and monitoring**

### Strategy Selection Guide

| Scenario | Recommended Strategy | Reason |
|----------|---------------------|---------|
| Development environment | Recreate | Simple, downtime acceptable |
| Production web app | Rolling Update | Zero downtime, built-in |
| Critical financial system | Blue-Green | Instant rollback capability |
| New feature testing | Canary | Risk mitigation |
| Performance testing | Shadow | No user impact |
| UX experiments | A/B Testing | User segmentation |

### Monitoring Checklist

- ✅ Application response time
- ✅ Error rates and status codes
- ✅ Resource utilization (CPU, memory)
- ✅ Pod readiness and health
- ✅ Service availability
- ✅ User experience metrics

## 🔧 Troubleshooting

### Common Issues and Solutions

#### Pods Not Starting
```bash
# Check pod status
kubectl describe pod <pod-name> -n <namespace>

# Check logs
kubectl logs <pod-name> -n <namespace>

# Common causes:
# - Image pull errors
# - Resource constraints
# - Configuration issues
```

#### Service Not Accessible
```bash
# Check service endpoints
kubectl get endpoints <service-name> -n <namespace>

# Check service configuration
kubectl describe service <service-name> -n <namespace>

# Test connectivity
kubectl port-forward service/<service-name> 8080:80 -n <namespace>
```

#### Deployment Stuck
```bash
# Check deployment status
kubectl rollout status deployment/<deployment-name> -n <namespace>

# Check replica sets
kubectl get rs -n <namespace>

# Force restart
kubectl rollout restart deployment/<deployment-name> -n <namespace>
```

### Debug Commands

```bash
# Get all resources in namespace
kubectl get all -n <namespace>

# Describe deployment for detailed info
kubectl describe deployment <deployment-name> -n <namespace>

# Check events
kubectl get events --sort-by='.lastTimestamp' -n <namespace>

# Execute commands in pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash

# Copy files from pod
kubectl cp <namespace>/<pod-name>:/path/to/file ./local-file
```

## 🤝 Contributing

We welcome contributions! Here's how you can help:

### Ways to Contribute

1. **Fix bugs or typos**
2. **Add new deployment strategies**
3. **Improve documentation**
4. **Add monitoring examples**
5. **Create automation scripts**

### Contribution Steps

```bash
# 1. Fork the repository
# 2. Create a feature branch
git checkout -b feature/amazing-feature

# 3. Make your changes
# 4. Test thoroughly
# 5. Commit your changes
git commit -m 'Add amazing feature'

# 6. Push to the branch
git push origin feature/amazing-feature

# 7. Open a Pull Request
```

### Guidelines

- Follow existing code style
- Add documentation for new features
- Test all deployment strategies
- Update README if needed

## 📚 Additional Resources

### Documentation
- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Kubernetes Deployment Strategies Guide](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kind Documentation](https://kind.sigs.k8s.io/)
- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/)

### Learning Resources
- [Kubernetes by Example](https://kubernetesbyexample.com/)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)
- [Katacoda Kubernetes Scenarios](https://www.katacoda.com/courses/kubernetes)

### Tools and Extensions
- [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [k9s - Kubernetes CLI](https://k9scli.io/)
- [Lens - Kubernetes IDE](https://k8slens.dev/)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Kubernetes community for excellent documentation
- Kind project for local Kubernetes development
- AWS for cloud infrastructure
- All contributors and learners

---

## 🎯 Next Steps

1. **Complete the Quick Start** to verify your setup
2. **Work through each deployment strategy** in order
3. **Experiment with different scenarios** 
4. **Apply to your real projects**
5. **Share your experience** with the community

**Happy Learning and Deploying! 🚀**

For questions, issues, or support, please open an issue in the GitHub repository.

---

*Made with ❤️ for the Kubernetes community*
