# kubernetes_demo_2percentage

# 🚀 EKS Platform Reliability Demo

This repo shows how to build a reliable Kubernetes platform on AWS using Terraform, Helm, Argo CD, and Prometheus. It includes:

- ✅ A working Amazon EKS cluster (control plane + worker nodes)
- 🔐 Secure IAM roles for service accounts (IRSA)
- 📦 GitOps deployment with Argo CD and Helm
- 📊 Observability with Prometheus alerts and Grafana dashboards
- 🧪 A runbook to upgrade the cluster safely

---

## 🧱 What’s inside

- `terraform/` — Infrastructure as Code (IaC) to build the cluster
- `runbook/Runbook.md` — Step-by-step guide to upgrade the cluster
- `argo/application.yaml` — Argo CD manifest to deploy your app
- `helm/my-app/values.yaml` — Helm config for your app (ECR + IRSA + NetworkPolicy)
- `monitoring/prometheus-rules.yaml` — Alerts for node/pod/db failures
- `monitoring/grafana-dashboard.json` — Dashboard for cluster health

---

## 🧑‍💻 How to run this

1. **Install tools**  
   - Terraform  
   - AWS CLI  
   - kubectl  
   - Helm  
   - Argo CD CLI (optional)

2. **Clone this repo**  
   ```bash
   git clone https://github.com/your-org/eks-platform-demo.git
   cd eks-platform-demo/terraform
   
   git clone https://github.com/kubernetes_demo_2percentage
   cd kubernetes_demo_2percentage/terraform


