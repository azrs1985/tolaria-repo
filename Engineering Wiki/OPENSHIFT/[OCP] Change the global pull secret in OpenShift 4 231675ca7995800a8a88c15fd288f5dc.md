# [OCP] Change  the global pull secret in OpenShift 4

Owner: Nam Tran
Last edited time: August 28, 2025 1:54 PM

1. Download the global cluster pull secret to your local file system.

```bash
$ oc extract secret/pull-secret -n openshift-config --to=.
```

1. In a text editor, edit the `.dockerconfigjson` file that was downloaded.

1. For updating the existing secret, use the following command:

```bash
$ oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=/path/to/downloaded/.dockerconfigjson

```