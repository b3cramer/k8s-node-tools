# kubelet-log-config.yaml (Container Log Rotation)

The `kubelet-log-config` tool is a Kubernetes DaemonSet that configures container log rotation settings on a node by modifying the kubelet's configuration file. This tool is important for managing disk space on your nodes by controlling the size and number of log files kept for each container. 🪵

---
### ⚠️ Warning
This tool runs as **privileged** and **restarts the kubelet service** on the node. A kubelet restart can disrupt running pods. It is highly recommended to apply this configuration to a new, empty node pool and then safely migrate your workloads to the newly configured nodes.

---
### **Prerequisites**

This tool only runs on nodes with a specific label. You must label the node(s) you want to configure before applying the DaemonSet.

1.  **Label the node:**
    ```bash
    kubectl label node <your-node-name> kubelet-log-config=true
    ```

---
### **How to use it?**

1.  Save the script to a file named `kubelet-log-config.yaml`.
2.  Before applying, you can edit the file to change the values for `CONTAINER_LOG_MAX_SIZE` (e.g., "20Mi") and `CONTAINER_LOG_MAX_FILES` (e.g., "10").
3.  Apply the DaemonSet to your cluster:
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/kubelet-log-config/kubelet-log-config.yaml
    ```

---
### **How to get the result?**

You can verify that the script ran successfully in two ways:

1.  **Check the pod logs:** The pod runs in the `kube-system` namespace and will output "Success!" upon completion.
    ```bash
    kubectl logs -n kube-system -l name=kubelet-log-config
    ```
2.  **Inspect the node's configuration:** SSH into the labeled node and view the kubelet config file to see the new values.
    ```bash
    cat /home/kubernetes/kubelet-config.yaml
    ```

---
### **How to remove it?**

You can remove the DaemonSet to prevent it from configuring any new nodes.

```bash
kubectl delete -f https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/kubelet-log-config/kubelet-log-config.yaml