## Lab 8 Upgrading the Kubernetes Cluster

NOTE: Before upgrading, we strongly suggest you backup your Kuberntes etcd meta data. You can find the full documentation on upgrading the cluster [here]( https://docs.mesosphere.com/services/kubernetes/2.0.0-1.12.1/operations/upgrade)

### Updating the package version

View/List available package versions:
```
$ dcos package describe kubernetes --package-versions
[
  "2.0.0-1.12.1",
  "1.2.0-1.10.5",
  "1.1.1-1.10.4",
  "1.1.0-1.10.3",
  "1.0.3-1.9.7",
  "1.0.2-1.9.6",
  "1.0.1-1.9.4",
  "1.0.0-1.9.3"
]
```

## Update DC/OS Kubernetes CLI
Before starting the update process, it is mandatory to update the CLI to the new version, in our case 2.0.0-1.12.2:

```
$ dcos package install kubernetes --cli --package-version=2.0.0-1.12.2
```

Below is how the user starts the package version update:

```
$ dcos kubernetes manager update --package-version=2.0.0-1.12.2

About to start an update from version <CURRENT_VERSION> to <NEW_VERSION>

Updating these components means the Kubernetes cluster may experience some
downtime or, in the worst-case scenario, cease to function properly.
Before updating proceed cautiously and always backup your data.

This operation is long-running and has to run to completion.
Are you sure you want to continue? [yes/no] yes

2018/03/01 15:40:14 starting update process...
2018/03/01 15:40:15 waiting for update to finish...
2018/03/01 15:41:56 update complete!
```
