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

Sure, I’ll rewrite the previous answer in Markdown so you can easily store it. Here it is:

---

## Certificate Management in Your Kubernetes Mini-Cluster

Great choice to keep it simple by not running applications yet—focusing on certificates is a perfect starting point, especially since "Kubernetes the Hard Way" gives you full control (and responsibility) over them. Point 5—certificate and security maintenance—is critical in a manual setup like yours, as Kubernetes relies heavily on certificates for secure communication. Let’s break this down: what certificates are involved, how to check their time-to-live (TTL), and an elegant way to manage them.

### What Certificates Are There?

In a "Kubernetes the Hard Way" setup, you’ve likely generated several certificates during the process. Here’s a rundown of the key ones in your cluster:

1. **CA Certificate (Certificate Authority)**:
   - **Purpose**: The root of trust for your cluster. It signs all other certificates (e.g., API server, node, client certs).
   - **File**: Often `ca.crt` (paired with `ca.key`).
   - **Location**: Stored where you generated it (e.g., your admin machine) and embedded in kubeconfig files.

2. **API Server Certificate**:
   - **Purpose**: Secures the Kubernetes API server (e.g., `https://server.kubernetes.local:6443`). Clients (like `kubectl`, kubelet) use it to verify the server.
   - **File**: Something like `kubernetes.crt` (with `kubernetes.key`).
   - **Location**: On the API server node (e.g., `/etc/kubernetes/pki/`).

3. **Node Certificates (Kubelet)**:
   - **Purpose**: Each node uses a client certificate to authenticate to the API server.
   - **File**: Per node, e.g., `node-0.crt` and `node-0.key` (from your earlier script).
   - **Location**: On each node, referenced in its kubeconfig (e.g., `/etc/kubernetes/kubelet.conf`).

4. **Admin/Client Certificate**:
   - **Purpose**: Used by you (the admin) or other clients to authenticate to the API server via `kubectl`.
   - **File**: Something like `admin.crt` and `admin.key`.
   - **Location**: In your `~/.kube/config` or a custom kubeconfig file.

5. **Other Potential Certificates**:
   - **Controller Manager/Scheduler**: May use client certs to talk to the API server (e.g., `controller-manager.crt`, `scheduler.crt`).
   - **Service Account Key**: For signing service account tokens (e.g., `sa.key`, `sa.pub`), though this might not apply if you skipped service accounts for now.

### How to See Their TTL (Time-to-Live)?

Certificates have an expiration date, and in Kubernetes, this is typically set when they’re generated (e.g., 1 year, 10 years for the CA). You can inspect their TTL using `openssl`.

#### Steps to Check TTL

1. **Locate the Certificate**:
   - Find the `.crt` file (e.g., `ca.crt`, `kubernetes.crt`, `node-0.crt`, `admin.crt`).
   - If it’s embedded in a kubeconfig, extract it (see below).

2. **Use OpenSSL to Inspect**:
   ```bash
   openssl x509 -in <certificate-file>.crt -noout -dates
   ```
   - Output example:
     ```
     notBefore=Mar 22 12:00:00 2025 GMT
     notAfter=Mar 22 12:00:00 2026 GMT
     ```
   - `notAfter` is the expiration date. Subtract today’s date (Mar 22, 2025, per your system) to get the remaining TTL (e.g., 1 year here).

3. **For Embedded Certs in Kubeconfig**:
   - Open the kubeconfig (e.g., `node-0.kubeconfig` or `~/.kube/config`) in a text editor.
   - Look for `certificate-authority-data`, `client-certificate-data`, or similar fields (Base64-encoded).
   - Decode and save it:
     ```bash
     echo "<base64-string>" | base64 -d > temp.crt
     openssl x509 -in temp.crt -noout -dates
     rm temp.crt
     ```

4. **Batch Check All Certs**:
   - If you’ve got multiple certs in a directory (e.g., `./pki/`), run a loop:
     ```bash
     for cert in *.crt; do
       echo "Checking $cert:"
       openssl x509 -in "$cert" -noout -dates
     done
     ```

### What’s an Elegant Way to Deal with This?

Managing certificates manually can get messy, especially as they expire at different times. Here’s an elegant, practical approach for your mini-cluster:

#### 1. Understand Your Cert Lifespan
- Check all TTLs now (as above) and note their expiration dates.
- Typically:
  - CA cert: Long-lived (e.g., 10 years).
  - API server, node, admin certs: Shorter (e.g., 1 year).
- In "Kubernetes the Hard Way," you set these durations when generating them (default might be 1 year unless you specified otherwise).

#### 2. Scripted Monitoring
- Create a simple script to check expiration dates periodically:
  ```bash
  #!/bin/bash
  # save as check-certs.sh
  certs=("ca.crt" "kubernetes.crt" "node-0.crt" "node-1.crt" "admin.crt")
  for cert in "${certs[@]}"; do
    if [ -f "$cert" ]; then
      echo "Certificate: $cert"
      openssl x509 -in "$cert" -noout -dates
      # Optional: Add logic to warn if < 30 days left
    fi
  done
  ```
- Run it weekly (e.g., via cron: `0 0 * * 0 ./check-certs.sh`) and review the output.

#### 3. Renewal Process
- **Before Expiry**: Regenerate expiring certs using the same CA:
  - For node certs (e.g., `node-0.crt`):
    ```bash
    openssl req -new -key node-0.key -out node-0.csr -config <original-config>
    openssl x509 -req -in node-0.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out node-0.crt -days 365
    ```
  - Update the kubeconfig or API server config with the new cert.
- **Restart Services**: After replacing certs, restart affected components (e.g., `systemctl restart kube-apiserver` or `kubelet`).

#### 4. Elegant Automation (Lightweight)
- Use a tool like `cert-manager` (overkill for now, but future-proof) or a custom script with renewal logic:
  - Check TTLs.
  - If < 30 days, auto-generate new certs and update configs.
  - Example pseudo-script:
    ```bash
    for cert in *.crt; do
      expiry=$(openssl x509 -in "$cert" -noout -enddate | cut -d= -f2)
      if [[ $(date -d "$expiry" +%s) -lt $(date -d "+30 days" +%s) ]]; then
        echo "Renewing $cert..."
        # Add renewal commands here
      fi
    done
    ```
- For now, keep it manual but scripted—automation can come as you scale.

#### 5. Centralize Cert Storage
- Store all certs and keys in one directory (e.g., `./pki/` on your admin machine) with backups. This makes monitoring and renewal easier.

### Your Next Steps

1. **Inventory**: List all your certs (CA, API server, nodes, admin) and their locations.
2. **Check TTLs**: Run the `openssl x509 -noout -dates` commands on each.
3. **Plan**: Decide how often to check (e.g., monthly) and script it if you like.

Since your cluster is fresh (set up around Mar 22, 2025), most certs likely have ~1 year left if you used defaults. Want to check a specific cert together or refine the script? Let me know!

---

Feel free to copy this into your notes! Let me know if you want future answers in Markdown too, or if you’re ready to move on with your cluster adventure. What’s next?
