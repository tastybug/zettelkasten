# Kubernetes the Hard Way

Talking to Grok to get a few insights to play with.

## Recurring Administrative Tasks

```
Regular Administrative Tasks for Your Mini-Cluster

Cluster Health Monitoring:
    Check node status (kubectl get nodes) and ensure all are Ready.
    Verify core components (API server, controller manager, scheduler, etcd) are running.
Resource Management:
    Monitor resource usage (CPU, memory) on nodes and pods (kubectl top nodes, kubectl top pods).
    Scale workloads or nodes if needed (e.g., adjust pod replicas or add nodes).
Workload Management:
    Deploy, update, or delete applications (e.g., kubectl apply -f, kubectl delete).
    Check pod status (kubectl get pods) and troubleshoot failures (e.g., kubectl logs, kubectl describe pod).
Networking Checks:
    Ensure pod networking (CNI plugin) is working (e.g., test pod-to-pod communication).
    Validate service exposure (e.g., ClusterIP, NodePort) and DNS resolution (CoreDNS).
Certificate and Security Maintenance:
    Monitor certificate expiration (e.g., CA, node certs) and renew them before they expire.
    Review RBAC policies (kubectl get rolebindings,clusterrolebindings) for least-privilege access.
Logging and Troubleshooting:
    Collect and review logs from pods and system components (kubectl logs, or check systemd logs for kubelet).
    Investigate and resolve issues (e.g., pod crashes, network errors).
Backup and Recovery:
    Back up etcd (the cluster’s brain) regularly to protect cluster state.
    Test restoring from backups to ensure disaster recovery works.
Upgrades and Patching:
    Apply security patches to nodes (e.g., OS updates).
    Plan and test Kubernetes version upgrades (e.g., kubeadm-style upgrades if adapted).
User and Access Management:
    Manage kubeconfig files or credentials for yourself and any other users/tools.
    Update RBAC rules as needed for new workloads or team members.
Storage Administration:
    Ensure persistent volumes (if used) are functioning (kubectl get pv,pvc).
    Check storage backend health (e.g., local disks, networked storage).
```
