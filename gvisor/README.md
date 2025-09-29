# enable-gvisor-flags.yaml (gVisor Pod Flag Overrides)

The `enable-gvisor-flags` tool is a Kubernetes DaemonSet that modifies the gVisor (`runsc`) configuration on GKE Sandbox nodes to allow runtime flags to be set on a per-pod basis. This tool is important for advanced debugging and customization of sandboxed pods by enabling specific gVisor features like `strace` or debug logging. 🔬

---
### ⚠️ Warning
This tool runs as **privileged** to modify a system configuration file. The flags you enable via pod annotations can alter the security, stability, and performance characteristics of your sandboxed workloads. Use these flags with a clear understanding of their function.

---
### **Prerequisites**

This DaemonSet is designed to run automatically on nodes that are part of a **GKE Sandbox node pool**, which uses the gVisor runtime. It will not run on standard node pools. This feature is supported on GKE versions `1.18.6-gke.3504` and later.

---
### **How to use it?**

1.  Save the script to a file named `enable-gvisor-flags.yaml`.
2.  Apply the DaemonSet to your cluster. It will automatically find and configure all gVisor-enabled nodes.
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/gvisor/enable-gvisor-flags.yaml
    ```

---
### **How to get the result?**

The result of this tool is the **ability to set gVisor flags using pod annotations**. After the DaemonSet has run, you can enable flags on any pod scheduled to a GKE Sandbox node.

1.  **Add annotations to your pod's metadata.** Here is an example that enables debug logging and `strace` for a specific pod:
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-sandboxed-pod
      annotations:
        dev.gvisor.flag.debug-log: "/tmp/sandbox-%ID/"
        dev.gvisor.flag.debug: "true"
        dev.gvisor.flag.strace: "true"
    spec:
      runtimeClassName: gvisor
      containers:
      - name: my-container
        image: nginx
    ```

2.  **To verify the configuration was applied to the node,** you can SSH into a gVisor node and check the contents of the `runsc` config file. It should contain the new line.
    ```bash
    cat /run/containerd/runsc/config.toml
    ```

---
### **How to remove it?**

You can remove the DaemonSet to prevent it from configuring any new gVisor nodes.

```bash
kubectl delete -f https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/gvisor/enable-gvisor-flags.yaml