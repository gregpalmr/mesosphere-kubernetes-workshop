### Lab 3 - Making Kubernetes Highly Available through the GUI

You can choose to make Kubernetes on DC/OS high availability (HA) at the time of deployment. However, you can also make non-HA clusters into HA by editing and saving the configuration. 

In the DC/OS UI under Services, click on your Kubernetes cluster (Not kube-proxy). 

In the top right, click Edit.

![](https://i.imgur.com/2dYmVLp.png)

Under Kubernetes in left hand menu, choose the high availability box

![](https://i.imgur.com/PkGHHlJ.png)

Save configuration and watch new components come online with the following command or in the GUI. Output should look like below:

```
$ dcos kubernetes manager plan status deploy --name=kubernetes-cluster1

deploy (serial strategy) (COMPLETE)
├─ etcd (serial strategy) (COMPLETE)
│  ├─ etcd-0:[peer] (COMPLETE)
│  ├─ etcd-1:[peer] (COMPLETE)
│  └─ etcd-2:[peer] (COMPLETE)
├─ control-plane (serial strategy) (IN_PROGRESS)
│  ├─ kube-control-plane-0:[instance] (COMPLETE)
│  ├─ kube-control-plane-1:[instance] (COMPLETE)
│  └─ kube-control-plane-2:[instance] (COMPLETE)
├─ mandatory-addons (serial strategy) (COMPLETE)
│  └─ mandatory-addons-0:[instance] (COMPLETE)
├─ node (parallel strategy) (COMPLETE)
│  └─ kube-node-0:[kubelet] (COMPLETE)
└─ public-node (serial strategy) (COMPLETE)

```

### Making Kubernetes Highly Available through the CLI

Grab the Kubernetes options.json file:

```
dcos kubernetes cluster  describe --cluster-name=kubernetes-cluster1 | grep -v '^Using' > options.json
```

Edit the options.json file to set "high_availability": true,

```
$ cat options.json
{
  "calico": {
    "calico_ipv4pool_cidr": "192.168.0.0/16",
    "cni_mtu": 1400,
    "felix_ipinipenabled": true,
    "felix_ipinipmtu": 1420,
    "ip_autodetection_method": "can-reach=9.0.0.0",
    "ipv4pool_ipip": "Always",
    "typha": {
      "enabled": false,
      "replicas": 3
    }
  },
  "etcd": {
    "cpus": 0.5,
    "data_disk": 3072,
    "disk_type": "ROOT",
    "mem": 1024,
    "wal_disk": 512
  },
  "kubernetes": {
    "authorization_mode": "AlwaysAllow",
    "control_plane_placement": "[[\"hostname\", \"UNIQUE\"]]",
    "control_plane_reserved_resources": {
      "cpus": 1.5,
      "disk": 10240,
      "mem": 4096
    },
    "high_availability": true,
    "private_node_count": 1,
    "private_node_placement": "",
    "private_reserved_resources": {
      "kube_cpus": 2,
      "kube_disk": 10240,
      "kube_mem": 2048,
      "system_cpus": 1,
      "system_mem": 1024
    },
    "public_node_count": 0,
    "public_node_placement": "",
    "public_reserved_resources": {
      "kube_cpus": 0.5,
      "kube_disk": 2048,
      "kube_mem": 512,
      "system_cpus": 1,
      "system_mem": 1024
    },
    "service_cidr": "10.100.0.0/16",
    "use_agent_docker_certs": false
  },
  "service": {
    "log_level": "INFO",
    "name": "kubernetes-cluster1",
    "service_account": "kubernetes-cluster1",
    "service_account_secret": "kubernetes-cluster1/sa",
    "sleep": 1000,
    "virtual_network_name": "dcos"
  }
}

```

Update the Kubernetes Framework and put it into high availability mode:

```
dcos kubernetes cluster update --options=options.json --cluster-name=kubernetes-cluster1 --yes
```

Output should resemble below:

```
The components of the cluster will be updated according to the changes in the
options file [options.json].

Updating these components means the Kubernetes cluster may experience some
downtime or, in the worst-case scenario, cease to function properly.
Before updating proceed cautiously and always backup your data.
This operation is long-running and has to run to completion.
Continue cluster update? [yes/no]: yes
2018/10/30 23:46:18 starting update process...
2018/10/30 23:46:19 waiting for update to finish...
2018/10/30 23:46:39 update complete!
```

Validate update against the plan:

```
$ dcos kubernetes manager plan status deploy --name=kubernetes-cluster1

deploy (serial strategy) (COMPLETE)
├─ etcd (serial strategy) (COMPLETE)
│  ├─ etcd-0:[peer] (COMPLETE)
│  ├─ etcd-1:[peer] (COMPLETE)
│  └─ etcd-2:[peer] (COMPLETE)
├─ control-plane (serial strategy) (COMPLETE)
│  ├─ kube-control-plane-0:[instance] (COMPLETE)
│  ├─ kube-control-plane-1:[instance] (COMPLETE)
│  └─ kube-control-plane-2:[instance] (COMPLETE)
├─ mandatory-addons (serial strategy) (COMPLETE)
│  └─ mandatory-addons-0:[instance] (COMPLETE)
├─ node (parallel strategy) (COMPLETE)
│  └─ kube-node-0:[kubelet] (COMPLETE)
└─ public-node (serial strategy) (COMPLETE)

```
Notice that in high availability mode, DC/OS has started 3 etcd peers and 3 kube-control-plane processes. If any of these processes die, DC/OS will monitor their health and automatically restart them.

