
---

## 📗 `playbook.md` — Cluster Upgrade Runbook (Beginner Friendly)

```markdown
# 🛠️ Cluster Upgrade Runbook (Control Plane + Nodes)

This playbook shows how to safely upgrade your Kubernetes cluster using `kubeadm`.

---

## ✅ Before You Start

- Make sure you have access to the control plane node.
- Backup etcd (the cluster brain).
- Confirm GitOps is working (Argo CD + Helm).
- Make sure monitoring is green (Prometheus + Grafana).
- Get approval from your team.

---

## 🔄 Step-by-Step

### Step 1: Backup etcd

```bash
export ETCDCTL_API=3
etcdctl snapshot save /tmp/etcd-snap.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

aws s3 cp /tmp/etcd-snap.db s3://your-audit-bucket/etcd-snapshots/

## Step 2: Plan the upgrade
bash

kubeadm version
kubeadm upgrade plan

Set your target version:
bash

TARGET_VERSION=v1.26.7

Step 3: Dry-run the upgrade
bash

sudo kubeadm upgrade apply ${TARGET_VERSION} --dry-run

Step 4: Apply the upgrade
bash

sudo kubeadm upgrade apply ${TARGET_VERSION} -y

Step 5: Upgrade worker nodes
For each :

bash

kubectl cordon <node>
kubectl drain <node> --ignore-daemonsets --delete-local-data
sudo apt-get install -y kubeadm=${TARGET_VERSION}-00 kubelet=${TARGET_VERSION}-00 kubectl=${TARGET_VERSION}-00
sudo kubeadm upgrade node
sudo systemctl restart kubelet
kubectl uncordon <node>

Step 6: Validate everything
bash

kubectl get nodes
kubectl get pods --all-namespaces
kubectl -n kube-system rollout status deploy/coredns

Step 7: Save evidence
bash

kubectl get nodes -o yaml > /tmp/nodes-after-upgrade.yaml
kubectl get pods --all-namespaces -o yaml > /tmp/pods-after-upgrade.yaml
aws s3 cp /tmp/nodes-after-upgrade.yaml s3://your-audit-bucket/evidence/
aws s3 cp /tmp/pods-after-upgrade.yaml s3://your-audit-bucket/evidence/

Rollback Plan: Restore etcd Snapshot to New Control Plane

If the control plane fails:

    Restore etcd snapshot to a new control plane.

    Follow your incident runbook.
Assumptions

    You have a valid etcd snapshot stored in Amazon S3 (or locally).

    You’re provisioning a new control plane node manually or via automation.

    You have access to the original cluster’s certificates and kubeadm config.

🔄 Step-by-Step Recovery

1. Provision a new control plane node

    Launch a new EC2 instance (same OS and specs as original).

    Install Kubernetes components:
bash

sudo apt-get update
sudo apt-get install -y kubeadm kubelet kubectl etcd

2. Download the etcd snapshot
bash

aws s3 cp s3://your-audit-bucket/etcd-snapshots/etcd-snap-<timestamp>.db /tmp/etcd-snap.db

3. Stop existing etcd service
bash

sudo systemctl stop etcd

4. Restore etcd from snapshot
bash

export ETCDCTL_API=3
etcdctl snapshot restore /tmp/etcd-snap.db \
  --data-dir /var/lib/etcd-restored

5. Update etcd systemd unit file
bash

Point --data-dir to /var/lib/etcd-restored
Update /etc/systemd/system/etcd.service or /etc/kubernetes/manifests/etcd.yaml if using static pods

6. Start etcd with restored data
bash

sudo systemctl daemon-reexec
sudo systemctl start etcd

. Reinitialize kubeadm (if needed)

    Use the original kubeadm-config.yaml (or regenerate from backup)

    Run:
    bash

sudo kubeadm init --config kubeadm-config.yaml

8. Rejoin worker nodes

    On each worker node:
    bash

kubeadm reset
kubeadm join <new-control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>

9. Validate cluster health
bash

kubectl get nodes
kubectl get pods --all-namespaces
kubectl get componentstatuses

📎 Incident Runbook Tie-In

    Notify stakeholders (App, Ops, GRC)
    Log incident in ticketing system
    Export post-restore evidence to S3
    Document root cause and recovery steps




