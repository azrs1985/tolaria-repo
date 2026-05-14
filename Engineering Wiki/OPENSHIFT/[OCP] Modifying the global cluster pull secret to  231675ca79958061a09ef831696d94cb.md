# [OCP] Modifying the global cluster pull secret to disable remote health reporting

Owner: Nam Tran
Last edited time: August 28, 2025 1:54 PM

You can modify your existing global cluster pull secret to disable remote health reporting. This disables both Telemetry and the Insights Operator.

**Prerequisites**

- You have access to the cluster as a user with the `cluster-admin` role.

**Procedure**

1. Download the global cluster pull secret to your local file system.
    
    ```bash
    $ oc extract secret/pull-secret -n openshift-config --to=.
    ```
    
2. In a text editor, edit the `.dockerconfigjson` file that was downloaded.
3. Remove the `cloud.openshift.com` JSON entry, for example:
    
    ```bash
    "cloud.openshift.com":{"auth":"<hash>","email":"<email_address>"}
    ```
    
4. Save the file.

You can now update your cluster to use this modified pull secret.