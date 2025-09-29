# SMT (Hyper-Threading) Configuration

These tools modify the Simultaneous Multithreading (SMT), also known as hyper-threading, setting on GKE nodes.

### **Important Note**
The officially recommended method for configuring SMT on GKE is to use the `--threads-per-core` flag when creating or updating a node pool with the `gcloud` command-line tool. These DaemonSets should be considered an alternative method. You can find more information in the [official GKE documentation](https://cloud.google.com/kubernetes-engine/docs/how-to/configure-smt).

---
## **`enable-smt.yaml` (Enable Hyper-Threading)**

The `enable-smt` tool is a Kubernetes DaemonSet that enables Simultaneous Multithreading (SMT) on targeted GKE nodes. This is important for increasing the potential CPU throughput for highly parallel workloads by allowing each physical CPU core to execute multiple threads concurrently. 🧠

#### ⚠️ Warning
* This tool runs as **privileged** and **restarts the kubelet**, which can disrupt workloads on the node.
* Enabling SMT may expose the node to security vulnerabilities like **Microarchitectural Data Sampling (MDS)**.
* It is highly recommended to apply this configuration to a new, empty node pool and then safely migrate your workloads to the newly configured nodes.

#### **Prerequisites**
This DaemonSet only runs on nodes with a specific label. You must label the node(s) you want to configure:
```bash
kubectl label node <your-node-name> [cloud.google.com/gke-smt-disabled=false](https://cloud.google.com/gke-smt-disabled=false)
```
How to use it?
Save the script to a file named enable-smt.yaml.

Apply the DaemonSet to your cluster.

```bash
kubectl apply -f  https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/disable-smt/gke/enable-smt.yaml
```
How to get the result?
SSH into the labeled node and check the SMT control file. The output should be on.

```bash
cat /sys/devices/system/cpu/smt/control
```
You can also check the pod's logs in the kube-system namespace to see the script's output.

How to remove it?
```bash
kubectl delete -f  https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/disable-smt/gke/enable-smt.yaml
Note: Removing the DaemonSet does not disable SMT on already-configured nodes.
```
disable-smt.yaml (Disable Hyper-Threading)
The disable-smt tool is a Kubernetes DaemonSet that disables Simultaneous Multithreading (SMT) on targeted GKE nodes. This is important for mitigating certain CPU security vulnerabilities (like MDS) or for ensuring predictable performance for workloads that do not benefit from hyper-threading. 🔒

⚠️ Warning
This tool runs as privileged and restarts the kubelet, which can disrupt workloads on the node.

Disabling SMT may have a severe performance impact on applications that rely on high thread counts.

It is highly recommended to apply this configuration to a new, empty node pool and then safely migrate your workloads.

Prerequisites
This DaemonSet only runs on nodes with a specific label. You must label the node(s) you want to configure:

```bash
kubectl label node <your-node-name> [cloud.google.com/gke-smt-disabled=true](https://cloud.google.com/gke-smt-disabled=true)
```
How to use it?
Save the script to a file named disable-smt.yaml.

Apply the DaemonSet to your cluster.

```bash
kubectl apply -f  https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/disable-smt/gke/disable-smt.yaml
```
How to get the result?
SSH into the labeled node and check the SMT control file. The output should be off.

```bash
cat /sys/devices/system/cpu/smt/control
You can also check the pod's logs in the kube-system namespace to see the script's output.
```

How to remove it?

```bash
kubectl delete -f  https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-node-tools/master/disable-smt/gke/disable-smt.yaml
Note: Removing the DaemonSet does not re-enable SMT on already-configured nodes.
 ```