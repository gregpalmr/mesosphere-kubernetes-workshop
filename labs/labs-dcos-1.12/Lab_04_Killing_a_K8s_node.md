## Automated Self Healing

Kubernetes with DC/OS includes automated self-healing of Kubernetes infrastructure. 

We can demo this by killing some of the Kubernetes framework components, as well as nodes. 

In the command line enter

```
dcos task
```

Output should resemble:
```
$ dcos task

NAME                                           HOST         USER   STATE  ID                                                                                               MESOS ID                                      REGION          ZONE
cluster1-kubectl-proxy                         10.0.3.71    root     R    cluster1-kubectl-proxy.e1ceff89-dc94-11e8-9737-6e7302759662                                      2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S0   aws/us-west-2  aws/us-west-2b
etcd-0-peer                                    10.0.1.147   root     R    kubernetes-cluster1__etcd-0-peer__1e90ef41-30da-456e-be4e-11fa9e78a545                           2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S4   aws/us-west-2  aws/us-west-2b
etcd-1-peer                                    10.0.0.36    root     R    kubernetes-cluster1__etcd-1-peer__b9615d85-230e-4e77-b68f-09d7e46f17b9                           2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S11  aws/us-west-2  aws/us-west-2b
etcd-2-peer                                    10.0.0.173   root     R    kubernetes-cluster1__etcd-2-peer__b032595e-0d2b-4a12-b5fa-b47e179a8e69                           2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S9   aws/us-west-2  aws/us-west-2b
kube-control-plane-0-instance                  10.0.1.143   root     R    kubernetes-cluster1__kube-control-plane-0-instance__c9d98201-7158-403e-b7b4-2b365b336cac         2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S5   aws/us-west-2  aws/us-west-2b
kube-control-plane-1-instance                  10.0.2.1     root     R    kubernetes-cluster1__kube-control-plane-1-instance__6d7f08e5-016a-46dc-b39d-455fe8ebf77a         2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S8   aws/us-west-2  aws/us-west-2b
kube-control-plane-2-instance                  10.0.2.18    root     R    kubernetes-cluster1__kube-control-plane-2-instance__450452f9-bdaa-473e-83bf-02c44ecb4948         2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S10  aws/us-west-2  aws/us-west-2b
kube-node-0-kubelet                            10.0.0.173   root     R    kubernetes-cluster1__kube-node-0-kubelet__0a55b725-f20d-4d0c-9e16-9cb6d0253b3d                   2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S9   aws/us-west-2  aws/us-west-2b
kubernetes                                     10.0.3.71    root     R    kubernetes.2c8904d7-dc7d-11e8-9737-6e7302759662                                                  2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S0   aws/us-west-2  aws/us-west-2b
kubernetes-cluster1                            10.0.2.230   root     R    kubernetes-cluster1.2d6b5dba-dc9c-11e8-9737-6e7302759662                                         2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S7   aws/us-west-2  aws/us-west-2b
marathon-lb                                    10.0.7.193   root     R    marathon-lb.40c98838-dc94-11e8-9737-6e7302759662                                                 2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S1   aws/us-west-2  aws/us-west-2b
mesosphere-kubernetes-engine-0-cosmos-adapter  10.0.3.151  nobody    R    kubernetes__mesosphere-kubernetes-engine-0-cosmos-adapter__8ee8b36c-019b-45bc-85c6-ef4c0f5eda62  2086f3c7-adcd-4ed9-a8b7-4be436fb1bf0-S6   aws/us-west-2  aws/us-west-2b

```

### Kill etcd

Lets kill an instance of the etcd database to observe auto-healing capabilities:

First we need to identify the etcd-0 PID value. In the example below the etcd PID value is 3:

```
$ dcos task exec -it etcd-0-peer ps ax
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:00 /opt/mesosphere/active/mesos/libexec/mesos/mesos-cont
    3 ?        Sl     2:04 ./etcd-v3.3.3-linux-amd64/etcd --name=infra0 --cert-f
 7008 pts/0    Ss+    0:00 /opt/mesosphere/active/mesos/libexec/mesos/mesos-cont
 7009 pts/0    R+     0:00 ps ax


 If your workern node doesn't have a `ps` command, you may use this command:

 $ dcos task exec -it etcd-0-peer /usr/bin/pidof etcd

 ```
 
Navigate to the DC/OS UI > Services > Kubernetes tab and open next to the terminal so you can see the components in the DC/OS UI. Use the search bar to search for etcd


Kill the etcd manually and watch the UI auto-heal the etcd instance:
```
dcos task exec -it etcd-0-peer kill -9 3
```

You may need to refresh brower. Watch as Kubernetes etcd-0 instance is killed and respawned

### Kill a Kubelet
Next, lets kill a Kubernetes node to observe auto-healing capabilities:

First we need to identify the kube-node-0 PID value. Enter etcd PID value associated with the cmd: `sh -c ./bootstrap --resolve=false 2>&1  chmod +x kube`: In the example below the etcd PID value is 3:

```
$ dcos task exec -it kube-node-0-kubelet ps ax
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:00 /opt/mesosphere/active/mesos/libexec/mesos/mesos-cont
    3 ?        S      0:00 sh -c ./bootstrap --resolve=false 2>&1  chmod +x kube
   10 ?        S      0:00 /bin/bash ./kubelet-wrapper.sh
```

Navigate to the DC/OS UI > Services > Kubernetes tab and open next to the terminal so you can see the components in the DC/OS UI. Use the search bar to search for kube-node-0

Kill the etcd manually and watch the UI auto-heal the etcd instance:

```
dcos task exec -it kube-node-0-kubelet kill -9 3
```

You may need to refresh brower. Watch as Kubernetes Kubelet instance is killed and respawned
