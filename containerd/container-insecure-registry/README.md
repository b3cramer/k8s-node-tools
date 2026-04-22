# Configure Insecure Registries for Containerd on GKE

This guide outlines how to configure Google Kubernetes Engine (GKE) nodes to pull container images from an insecure (HTTP) registry.

Historically, this was achieved using a privileged `DaemonSet` to modify the node's filesystem. However, the **recommended method** is now to use GKE's native `containerd` customization feature, which is more secure, officially supported, and compatible with both Standard and Autopilot clusters.

---

## Recommended Method: Native GKE Configuration (GKE 1.34.1-gke.2980000 )

The preferred way to configure insecure registries is using the `ContainerdConfig` custom resource. This method is persistent across node upgrades and does not require privileged pods.

### 1. Prepare the Configuration
Create a file named `containerd-config-native.yaml` (or use the one provided in this directory). Replace `REGISTRY_ADDRESS` with your registry's actual address (e.g., `my-registry.local:5000`).

```yaml
registryHosts:
- server: "REGISTRY_ADDRESS"
  hosts:
  - host: "http://REGISTRY_ADDRESS"
    capabilities:
    - "HOST_CAPABILITY_PULL"
    - "HOST_CAPABILITY_RESOLVE"
```

### 2. Apply to a Node Pool
Use the `gcloud` command to apply this configuration to an existing node pool or when creating a new one.

**Update an existing node pool:**
```bash
gcloud container node-pools update NODE_POOL_NAME \
    --cluster=CLUSTER_NAME \
    --location=LOCATION \
    --containerd-config-from-file=containerd-config-native.yaml
```

**Create a new node pool:**
```bash
gcloud container node-pools create NEW_NODE_POOL_NAME \
    --cluster=CLUSTER_NAME \
    --location=LOCATION \
    --containerd-config-from-file=containerd-config-native.yaml
```

*Note: GKE will automatically apply the configuration to the nodes. Existing nodes will be recreated to apply the new configuration.*

---

## Reference: Generated Configuration

When using the native method, GKE automatically generates the appropriate `containerd` configuration on the node. For example, the configuration above results in a file at `/etc/containerd/hosts.d/REGISTRY_ADDRESS/hosts.toml` with the following content:

```toml
server = 'REGISTRY_ADDRESS'

[host]
  [host.'http://REGISTRY_ADDRESS']
    capabilities = ['pull', 'resolve']
```

---

## Legacy Method: Privileged DaemonSet

This method uses a `DaemonSet` that modifies the `containerd` configuration on each node. This is typically used for older GKE versions or specific edge cases.

### Instructions

1.  **Modify the DaemonSet YAML file.** Before applying the manifest, you must edit `insecure-registry-config.yaml` to specify the address of your insecure registry. Locate the `env` section for the `startup-script` container and change the `value` for the `ADDRESS` variable from `REGISTRY_ADDRESS` to your registry's actual address.

    **Original:**
    ```yaml
          env:
          - name: ADDRESS
            value: "REGISTRY_ADDRESS"
    ```
    **Example after editing:**
    ```yaml
          env:
          - name: ADDRESS
            value: "my-registry.local:5000"
    ```
2.  **Deploy the DaemonSet.** Apply the modified YAML file to your cluster.

    ```bash
    kubectl apply -f insecure-registry-config.yaml
    ```
    The script within the pod will automatically update the `containerd` configuration and restart the service.

---

## How the Legacy Method Works

The `DaemonSet` runs a privileged pod on each node. This pod executes a startup script that:
1.  Identifies the `containerd` configuration file (`/etc/containerd/config.toml`).
2.  Detects which configuration model the node is using (legacy V1 or modern V2).
3.  Injects the necessary configuration to define your registry as an insecure endpoint (`http://...`).
4.  Reloads the `systemd` daemon and restarts the `containerd` service.

---

## Important Considerations

* **Security Risk:** Configuring an insecure registry means that image pulls are unencrypted (HTTP) and unauthenticated. This should only be used in trusted, isolated network environments.
* **GKE Autopilot:** The **Native Configuration method** is supported on GKE Autopilot (version 1.29 and later). The **Legacy DaemonSet method** is **not compatible** with Autopilot clusters, as they restrict privileged host access.