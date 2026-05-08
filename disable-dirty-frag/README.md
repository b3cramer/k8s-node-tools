# Mitigation Advice for CVE-2026-43284 (Dirty Frag)

This document provides immediate mitigation steps for CVE-2026-43284. While waiting for patched node images to roll out, you can protect your clusters using the following methods.

CVE-2026-43284 is a Linux kernel privilege escalation vulnerability in ESP (Encapsulating Security Payload) in-place decryption. A second pending-assignment CVE in the same disclosure affects the RxRPC protocol (used by AFS). Local privilege escalation to root is confirmed; container escape is considered possible. Canonical rates this CVSS 3.1 7.8 (High).

## GKE Standard Nodes (COS and Ubuntu)

You can unload the `esp4`, `esp6`, and `rxrpc` kernel modules using the privileged DaemonSet (`disable-dirty-frag.yaml`) provided in this directory. **Unlike the algif-aead mitigation, this does not require a node reboot.** The DaemonSet re-applies the mitigation automatically on node restart.

Use the `cloud.google.com/gke-dirty-frag-disabled=true` node label to control which nodes receive the mitigation.

**Deployment Instructions:**
1. Label your target nodes to control the rollout:
```bash
kubectl label nodes <node-name> cloud.google.com/gke-dirty-frag-disabled=true
```
2. Apply the DaemonSet:
```bash
kubectl apply -f disable-dirty-frag.yaml
```

To label all nodes at once:
```bash
kubectl label nodes --all cloud.google.com/gke-dirty-frag-disabled=true
```

### ⚠️ Known Limitations
* **IPsec traffic:** Unloading `esp4` and `esp6` blocks IPsec ESP traffic. If your environment uses IPsec-based VPNs or encrypted tunnels, test thoroughly before applying.
* **AFS clients:** The `rxrpc` module is only required by AFS clients. It is safe to disable on most GKE nodes, but verify if your workloads use the Andrew File System.
* **Not persistent across node image replacement:** This mitigation unloads modules from the running kernel. If a node is replaced with a fresh image (e.g. after a node pool upgrade), the DaemonSet will re-apply the mitigation automatically on the replacement node, provided the node label is preserved.

## GKE Autopilot (and Alternative for Standard)

GKE Autopilot does not allow running privileged daemonsets, and therefore must be mitigated by applying a [custom seccomp profile to your workloads](../spo-seccomp-mitigation). This also works as an alternative mitigation method for GKE Standard clusters.

---
*Note: We do not recommend relying on containers as a strict security boundary. For stronger isolation, consider using GKE Sandbox, network policies and the guidance found at [https://docs.cloud.google.com/kubernetes-engine/docs/how-to/hardening-your-cluster](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/hardening-your-cluster).*
