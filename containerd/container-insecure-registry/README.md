# Configure Insecure Registries for Containerd on GKE

This guide outlines how to configure Google Kubernetes Engine (GKE) nodes to pull container images from an insecure (HTTP) registry. This is achieved by deploying a `DaemonSet` that modifies the `containerd` configuration on each node to trust the specified registry. This is typically used for accessing local or private container registries that are not configured with TLS.

---

## Instructions

1.  **Modify the DaemonSet YAML file.** Before applying the manifest, you must edit it to specify the address of your insecure registry. Locate the `env` section for the `startup-script` container and change the `value` for the `ADDRESS` variable from `REGISTRY_ADDRESS` to your registry's actual address (e.g., `my-registry.local:5000`).

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
2.  **Deploy the DaemonSet.** Apply your modified YAML file to your cluster. The `DaemonSet` will create a pod on each GKE node that uses the `containerd` runtime.

    ```bash
    kubectl apply -f your-daemonset-filename.yaml
    ```
    The script within the pod will then automatically update the `containerd` configuration and restart the service to apply the changes.

---

## How It Works

The `DaemonSet` runs a privileged pod on each node selected by the `nodeSelector` (`cloud.google.com/gke-container-runtime: "containerd"`). This pod executes a startup script that:
1.  Identifies the `containerd` configuration file (`/etc/containerd/config.toml`).
2.  Detects which configuration model the node is using (legacy V1 or modern V2).
3.  Injects the necessary configuration to define your registry as an insecure endpoint (`http://...`).
4.  Reloads the `systemd` daemon and restarts the `containerd` service to make the changes take effect.

---

## Important Considerations

* **Security Risk:** Configuring an insecure registry means that image pulls are unencrypted (HTTP) and unauthenticated, which can be a security risk. This should only be used in trusted, isolated network environments.
* **GKE Autopilot:** This method uses a privileged `DaemonSet` that modifies the host node's file system. This is **not compatible with GKE Autopilot clusters**, which restrict this level of host access. This solution is intended for GKE Standard clusters only.