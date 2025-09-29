# Kubernetes Node Profiling Tools

These tools deploy the Linux `perf` utility to profile and trace performance on specific Kubernetes nodes.

---
## **Prerequisites for Both Tools**

These tools only run on nodes with a specific label. You must label the node you want to profile before using them.

1.  **Label the node:**
    ```bash
    kubectl label node <your-node-name> enable-perf=true
    ```
2.  **Check Kernel Version:** You must edit the downloaded YAML file to ensure the `KERNEL_VERSION` variable matches your node's kernel (`uname -r`).

---
### **`perf-record.yaml` (CPU Profiling)**

The `perf-record` tool is a Kubernetes DaemonSet that continuously profiles CPU usage on a node. This tool is important for discovering which programs and functions are consuming the most CPU time. 🏎️

#### ⚠️ Warning
This tool runs as **privileged** and adds performance overhead. Use it only for debugging and remove it when finished.

#### How to use it?
Apply it to a labeled node by running the following command.
```
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/perf/perf-record.yaml
```

#### How to get the result?
The profiling data is not in the pod logs. It is saved directly on the node's filesystem. SSH into your node and find the `perf.data` files in a new timestamped directory inside:
/var/log/perf_record/

---
### **`perf-trace.yaml` (System Call Tracing)**

The `perf-trace` tool is a Kubernetes DaemonSet that traces system calls made by programs on a node. This tool is important for debugging low-level application errors related to file access, permissions, or networking. 🐞

#### ⚠️ Warning
This tool runs as **privileged** and adds performance overhead. Use it only for debugging and remove it when finished.

#### How to use it?
Apply it to a labeled node by running the following command. You can edit the YAML to change the `TARGET_PGREP` environment variable to trace a specific process (e.g., "containerd") or leave it empty to trace the whole system.
```
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/perf/perf-trace.yaml
```

#### How to get the result?
The trace data is not in the pod logs. It is saved directly on the node's filesystem. SSH into your node and find the log files in a new timestamped directory inside:
```
/var/log/perf_trace/
```

#### How to remove it?
```
kubectl delete -f https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/perf/perf-trace.yaml
```