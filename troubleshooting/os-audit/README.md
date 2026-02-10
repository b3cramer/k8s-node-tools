The os-audit tools provide example code for
[managing Linux auditd logs on GKE nodes](https://cloud.google.com/kubernetes-engine/docs/how-to/linux-auditd-logging),
which documents how to enable and disable verbose operating system audit logs on
Google Kubernetes Engine nodes running Container-Optimized OS.

#### Available Tools

-   `cos-auditd-logging.yaml`: A DaemonSet that enables and starts the
    `cloud-audit-setup` and `audit-rules` services on the host.

    Deployment:

    ```sh
    export CLUSTER_NAME=<CLUSTER_NAME> # e.g. 'cluster-1'
    export CLUSTER_LOCATION=<CLUSTER_LOCATION> # e.g. 'us-central1-c'

    envsubst '$CLUSTER_NAME,$CLUSTER_LOCATION' < cos-auditd-logging.yaml | kubectl apply -f -
    ```

-   `cos-auditd-logging-disable.yaml`: A DaemonSet that disables these services
    to revert changes and restore the default node logging state.

    **Note**: Before deploying this DaemonSet, delete the `cos-auditd-logging`
    DaemonSet and wait for all associated Pods to be deleted. Once the
    `cleanup-auditd` DaemonSet (`cos-auditd-logging-disable.yaml`) has
    successfully rolled out to all nodes, it can be deleted.

    Deployment:

    ```sh
    kubectl apply -f cos-auditd-logging-disable.yaml
    ```
