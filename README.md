# Kubernetes Deployment on AWS EKS

> Production-style containerised workload deployment on AWS Elastic Kubernetes Service — with auto-scaling, load balancing, and day-to-day cluster management.

---

## Overview

This project deploys and manages containerised applications on AWS EKS, demonstrating real-world Kubernetes operations: cluster provisioning, workload deployment, horizontal pod autoscaling, load balancer configuration, and kubectl-based debugging.

**Built by:** Girish Dinni — DevOps & Cloud Engineer  
**Stack:** Kubernetes · AWS EKS · Docker · kubectl · HPA · ALB  
**Goal:** Demonstrate production-grade container orchestration for DevOps/Cloud/SRE roles

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS Region                            │
│                                                              │
│   ┌──────────────────────────────────────────────────────┐  │
│   │                   EKS Cluster                        │  │
│   │                                                      │  │
│   │   ┌─────────────────────────────────────────────┐   │  │
│   │   │              Node Group                     │   │  │
│   │   │                                             │   │  │
│   │   │  ┌──────────┐ ┌──────────┐ ┌──────────┐   │   │  │
│   │   │  │  Pod 1   │ │  Pod 2   │ │  Pod 3   │   │   │  │
│   │   │  │ (app)    │ │ (app)    │ │ (app)    │   │   │  │
│   │   │  └──────────┘ └──────────┘ └──────────┘   │   │  │
│   │   │         ↑ HPA scales pods on load          │   │  │
│   │   └─────────────────────────────────────────────┘   │  │
│   │                        │                             │  │
│   │              ┌─────────▼──────────┐                 │  │
│   │              │  Kubernetes Service │                 │  │
│   │              │  (LoadBalancer/ALB) │                 │  │
│   │              └─────────┬──────────┘                 │  │
│   └────────────────────────┼───────────────────────────-┘  │
│                             │                               │
│                    ┌────────▼────────┐                      │
│                    │   Internet /    │                      │
│                    │   End Users     │                      │
│                    └─────────────────┘                      │
└─────────────────────────────────────────────────────────────┘
```

---

## What Gets Deployed

| Resource | Details |
|---|---|
| **EKS Cluster** | Managed Kubernetes control plane on AWS |
| **Node Group** | EC2 worker nodes (t3.medium, auto-scaling 2–5 nodes) |
| **Deployment** | App with 3 replicas, rolling update strategy |
| **Service** | LoadBalancer type — exposes app via AWS ALB |
| **HPA** | Horizontal Pod Autoscaler — scales 2–10 pods on CPU > 50% |
| **ConfigMap** | Environment config injected into pods |
| **Namespace** | Isolated namespace per environment (dev/prod) |

---

## Project Structure

```
k8s-eks-deployment/
├── cluster/
│   └── eksctl-config.yaml       # EKS cluster definition (eksctl)
│
├── manifests/
│   ├── namespace.yaml           # Environment namespace
│   ├── deployment.yaml          # App deployment — replicas, image, resources
│   ├── service.yaml             # LoadBalancer service
│   ├── hpa.yaml                 # Horizontal Pod Autoscaler
│   └── configmap.yaml           # Environment variables
│
├── scripts/
│   ├── deploy.sh                # Full deploy script
│   ├── rollback.sh              # Roll back to previous version
│   └── debug.sh                 # Common kubectl debug commands
│
└── README.md
```

---

## Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/) configured
- [eksctl](https://eksctl.io/) installed
- [AWS CLI](https://aws.amazon.com/cli/) with valid credentials
- Docker image pushed to ECR or Docker Hub

---

## Quick Start

**1. Clone the repository**
```bash
git clone https://github.com/kgdinni/k8s-eks-deployment.git
cd k8s-eks-deployment
```

**2. Create the EKS cluster**
```bash
eksctl create cluster -f cluster/eksctl-config.yaml
```

**3. Verify cluster access**
```bash
kubectl get nodes
kubectl get namespaces
```

**4. Deploy the application**
```bash
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/configmap.yaml
kubectl apply -f manifests/deployment.yaml
kubectl apply -f manifests/service.yaml
kubectl apply -f manifests/hpa.yaml
```

**5. Check deployment status**
```bash
kubectl get pods -n app
kubectl get svc -n app
kubectl get hpa -n app
```

**6. Get the load balancer URL**
```bash
kubectl get svc app-service -n app -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

---

## Key Manifests

**Deployment — rolling update with resource limits:**
```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    spec:
      containers:
        - name: app
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

**HPA — auto-scale on CPU:**
```yaml
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

---

## Common kubectl Commands

```bash
# Check pod status and events
kubectl describe pod <pod-name> -n app

# Stream logs from a pod
kubectl logs -f <pod-name> -n app

# Exec into a running pod
kubectl exec -it <pod-name> -n app -- /bin/sh

# Check HPA scaling activity
kubectl describe hpa -n app

# Force rolling restart
kubectl rollout restart deployment/app -n app

# Check rollout status
kubectl rollout status deployment/app -n app

# Roll back to previous version
kubectl rollout undo deployment/app -n app
```

---

## Cleanup

```bash
kubectl delete -f manifests/
eksctl delete cluster -f cluster/eksctl-config.yaml
```

---

## Related Projects

| Project | Stack | Link |
|---|---|---|
| AWS Infrastructure with Terraform | Terraform · EC2 · VPC · IAM | [View repo](https://github.com/kgdinni/terraform-aws-infra) |
| CI/CD Pipeline Automation | Jenkins · Docker · GitHub Actions | [View repo](https://github.com/kgdinni/cicd-jenkins-docker) |

---

## Author

**Girish Dinni**  
DevOps & Cloud Engineer · Bengaluru, India  
[LinkedIn](https://www.linkedin.com/in/girish-dinni) · [GitHub](https://github.com/kgdinni)  
kgdinni@gmail.com
