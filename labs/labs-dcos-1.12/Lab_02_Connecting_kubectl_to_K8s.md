# LAB 2 - Connecting kubectl to DC/OS

### Step 1. Deploy the DC/OS Marathon-LB HAPROXY based load balancer

Run the command to install Marathon-LB:

```
dcos package install marathon-lb --yes
```

### Step 2. Launch a proxy service on DC/OS

Launch a proxy service on DC/OS to expose the Kubernetes API Server port.

### Step 2.a

Create a kubectl-proxy service specification file:
```
cat <<EOF > cluster1-kubectl-proxy.json
{
  "id": "/cluster1-kubectl-proxy",
  "instances": 1,
  "cpus": 0.001,
  "mem": 16,
  "cmd": "tail -F /dev/null",
  "container": {
    "type": "MESOS"
  },
  "portDefinitions": [
    {
      "protocol": "tcp",
      "port": 0
    }
  ],
  "labels": {
    "HAPROXY_0_MODE": "http",
    "HAPROXY_GROUP": "external",
    "HAPROXY_0_SSL_CERT": "/etc/ssl/cert.pem",
    "HAPROXY_0_PORT": "6443",
    "HAPROXY_0_BACKEND_SERVER_OPTIONS": "  timeout connect 10s\n  timeout client 86400s\n  timeout server 86400s\n  timeout tunnel 86400s\n  server kubernetescluster1 apiserver.kubernetes-cluster1.l4lb.thisdcos.directory:6443 ssl verify none\n"
  }
}
EOF
```

### Step 2.b 

Deploy the cluster1-kubectl-proxy service with the command:
```
dcos marathon app add cluster1-kubectl-proxy.json
```

Here is how this works:
* Marathon-LB identifies that the application kubeapi-proxy has the HAPROXY_GROUP label set to external (change this if you're using a different HAPROXY_GROUP for your Marathon-LB configuration).
* The instances, cpus, mem, cmd, and container fields basically create a dummy container that takes up minimal space and performs no operation.
* The single port indicates that this application has one "port" (this information is used by Marathon-LB)
* "HAPROXY_0_MODE": "http" indicates to Marathon-LB that the frontend and backend configuration for this particular service should be configured with http.
* "HAPROXY_0_PORT": "6443" tells Marathon-LB to expose the service on port 6443 (rather than the randomly-generated service port, which is ignored)
* "HAPROXY_0_SSL_CERT": "/etc/ssl/cert.pem" tells Marathon-LB to expose the service with the self-signed Marathon-LB certificate (which has no CN)
* The last label HAPROXY_0_BACKEND_SERVER_OPTIONS indicates that Marathon-LB should forward traffic to the endpoint apiserver.kubernetes-cluster1.l4lb.thisdcos.directory:6443 rather than to the dummy application, and that the connection should be made using TLS without verification.


### Step 3. Find public IP address of Public DC/OS Node

Find the public IP address of the Public DC/OS node that is running the Marathon-LB load balancer. Run the following command (make sure Marathon-LB is running first, with the command 'dcos task marathon-lb'):

```
MARATHON_PUB_IP=$(priv_ip=$(dcos task marathon-lb | grep -v HOST | awk '{print $2}') && dcos node ssh --option StrictHostKeyChecking=no --option LogLevel=quiet --master-proxy --private-ip=$priv_ip "curl -s ifconfig.co | sed 's/\r//g'"); echo && echo "MARATHON_PUB_IP:   $MARATHON_PUB_IP"
```

### Step 4. Connecting using Kubeconfig

### Step 4.a 

Configure kubectl to connect to the Kubernetes cluster running on  DC/OS using the following commands:
```
dcos kubernetes cluster kubeconfig \
    --insecure-skip-tls-verify \
    --context-name=kubernetes-cluster1 \
    --cluster-name=kubernetes-cluster1 \
    --apiserver-url=https://${MARATHON_PUB_IP}:6443
```

### Step 4.b

Confirm connection:

```
kubectl get nodes
```

### Step 5. Kubernetes Dashboard (Official UI of Kubernetes)

(NOTE: if you are using a bootstrap server to access your cluster then the local proxy will not give you access to the Dashboard.)

To access the dashboard run:

```
kubectl proxy
```

Point your browser to:

```
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

### Step 6. Switching Clusters using kubectl

To get your contexts, use the command:

kubectl config get-contexts

Output should look like below:

```
$ kubectl config get-contexts
CURRENT   NAME                  CLUSTER               AUTHINFO              NAMESPACE
*         kubernetes-cluster1   kubernetes-cluster1   kubernetes-cluster1
          kubernetes-cluster2   kubernetes-cluster2   kubernetes-cluster2
```

Switch contexts:

kubectl config use-context <CONTEXT_NAME>

Rename your contexts:

kubectl config rename-context <CURRENT_CONTEXT_NAME> <NEW_CONTEXT_NAME>

