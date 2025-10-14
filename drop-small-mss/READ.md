# drop-small-mss.yaml (Network Security Hardening)

The `drop-small-mss` tool is a Kubernetes DaemonSet that adds a firewall rule to every cluster node to drop incoming TCP packets with an unusually small Maximum Segment Size (MSS). This tool is important for hardening network security and protecting nodes from certain types of denial-of-service (DoS) attacks that exploit small packet sizes to exhaust server resources. 🛡️

---
### ⚠️ Warning
This tool runs as a **privileged** and **system-node-critical** pod to modify the host's core networking rules. While this rule protects against a known attack vector, it could potentially interfere with legitimate but non-standard network clients or tunnels that require a small MSS to function correctly.

---
### **Prerequisites**

There are no special prerequisites. This DaemonSet is designed to run on **all nodes** in your cluster, including control-plane nodes.

---
### **How to use it?**

1.  Save the script to a file named `drop-small-mss.yaml`.
2.  Apply the DaemonSet to your cluster.
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/drop-small-mss/drop-small-mss.yaml
    ```

---
### **How to get the result?**

The result of this tool is a new `iptables` rule on every node. You can verify that the rule was successfully applied:

1.  **SSH into any node** in your cluster.
2.  **List the firewall rules** in the `mangle` table. You should see the new rule with the "drop-small-mss" comment at the top of the `PREROUTING` chain.
    ```bash
    iptables -t mangle -L PREROUTING -v
    ```

---
### **How to remove it?**

You can delete the DaemonSet to prevent the rule from being applied to any new nodes that join the cluster.

```bash
kubectl delete -f https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/drop-small-mss/drop-small-mss.yaml