# Lab 1 Launch a Kubernetes  Cluster

The instructor will give you access to IP address and credentials that you will need to SSH into.

### Step 1. Set Up DC/OS Command Line

### Step 1.a

If you have a Macbook or Linux laptop and you don't have any restrictions on accessing servers on the internet, you can use the instructions in Step 1.b on your laptop. If you don't then you should login to your cluster's "bootstrap" server and use it as a command line client.

Download the id_rsa key from the workshop cluster github location at:

SSH Key
https://raw.githubusercontent.com/gregpalmr/mesosphere-kubernetes-workshop/master/clusters/Workshop-Clusters-2018-11-01/keys/id_rsa

SSH to the bootstrap server:
```
ssh -i ./id_rsa centos@<your bootstrap server ip address>
```

If using Windows and Putty Telnet, use the putty key at:  

https://raw.githubusercontent.com/gregpalmr/mesosphere-kubernetes-workshop/master/clusters/Workshop-Clusters-2018-11-01/keys/puttykey.ppk



### Step 1.b

Set up the DC/OS command line by clicking on the top left and choosing "install CLI"

![CLI](https://i.imgur.com/p4kqIj6.png)

Click in the dialogue box too copy the command

![Copy Command](https://i.imgur.com/3rQ2Unj.png)

Paste that command into your Terminal and press enter

For CoreOS use the following commands to install the CLI binary:

```
sudo mkdir -p /opt/bin && 
curl https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.12/dcos -o dcos && 
chmod +x dcos &&
sudo mv dcos /opt/bin

Then run the `dcos cluster setup` command for your cluster like this:

NOTE: For this workshop, you must use an HTTPS URL for setting up the cluster with the DC/OS CLI.

dcos cluster setup https://<master node ip addr>
```

Once the CLI is installed, confirm that it is installed correctly and connected to your cluster by running following command

```
dcos node

```

The output should be a list of nodes in the cluster

```

   HOSTNAME        IP                         ID                     TYPE                 REGION          ZONE       
  10.0.0.101   10.0.0.101  94141db5-28df-4194-a1f2-4378214838a7-S0   agent            aws/us-west-2  aws/us-west-2a  
  10.0.2.100   10.0.2.100  94141db5-28df-4194-a1f2-4378214838a7-S4   agent            aws/us-west-2  aws/us-west-2a
```

### Step 2. Tour DC/OS Catalog

Your instructor will give you a tour of DC/OS UI and catalog. 

### Step 3. Launch a Kubernetes Cluster 

To launch a Kubernetes cluster, you must first deploy the Mesosphere Kubernetes Control Plane Manager. 

### Step 3.a 

Install the Kubernetes Control Plane Manager with the following command:

dcos package install kubernetes --yes

```
By Deploying, you agree to the Terms and Conditions https://mesosphere.com/catalog-terms-conditions/#certified-services
Mesosphere Kubernetes Engine
Installing Marathon app for package [kubernetes] version [2.0.0-1.12.1]
Installing CLI subcommand for package [kubernetes] version [2.0.0-1.12.1]
New command available: dcos kubernetes
The Mesosphere Kubernetes Engine service is being installed.
```

You can check to see if the control manager is installed completely by running the following command:

dcos kubernetes manager plan status deploy

```
deploy (serial strategy) (COMPLETE)
└─ mesosphere-kubernetes-engine (serial strategy) (COMPLETE)
   └─ mesosphere-kubernetes-engine-0:[cosmos-adapter] (COMPLETE)
```

When all steps are "COMPLETE", confirm that the "dcos kubernetes" CLI was installed.

```
dcos kubernetes --help
```

### Step 3.b 

Once the Kubernetes control plan manager is running, you can use it to launch a Kubernetes cluster.  Since you are using the Enterprise version of DC/OS, you can use the DC/OS certificate authoritity to create an SSL key to be used with a DC/OS service account user.

Run the following commands to create the SSL keys, the service account and the secret.

```
dcos package install --cli dcos-enterprise-cli --yes

dcos security org service-accounts keypair private-key.pem public-key.pem
dcos security org service-accounts create -p public-key.pem -d 'Kubernetes cluster 1 service account' kubernetes-cluster1
dcos security secrets create-sa-secret private-key.pem kubernetes-cluster1 kubernetes-cluster1/sa
```

You should also grant the correct permissions to allow the new service account to launch and view Kubernetes clusters. Run the following ACL commands:

```
dcos security org users grant kubernetes-cluster1  dcos:mesos:master:framework:role:kubernetes-cluster1-role create
dcos security org users grant kubernetes-cluster1  dcos:mesos:master:task:user:root create
dcos security org users grant kubernetes-cluster1  dcos:mesos:agent:task:user:root create

dcos security org users grant kubernetes-cluster1  dcos:mesos:master:reservation:role:kubernetes-cluster1-role create
dcos security org users grant kubernetes-cluster1  dcos:mesos:master:reservation:principal:kubernetes-cluster1 delete
dcos security org users grant kubernetes-cluster1  dcos:mesos:master:volume:role:kubernetes-cluster1-role create
dcos security org users grant kubernetes-cluster1  dcos:mesos:master:volume:principal:kubernetes-cluster1 delete

dcos security org users grant kubernetes-cluster1  dcos:secrets:default:/kubernetes-cluster1/* full
dcos security org users grant kubernetes-cluster1  dcos:secrets:list:default:/kubernetes-cluster1 read

dcos security org users grant kubernetes-cluster1  dcos:adminrouter:ops:ca:rw full
dcos security org users grant kubernetes-cluster1  dcos:adminrouter:ops:ca:ro full

dcos security org users grant kubernetes-cluster1  dcos:mesos:master:framework:role:slave_public/kubernetes-cluster1-role create
dcos security org users grant kubernetes-cluster1  dcos:mesos:master:framework:role:slave_public/kubernetes-cluster1-role read
dcos security org users grant kubernetes-cluster1  dcos:mesos:master:reservation:role:slave_public/kubernetes-cluster1-role create
dcos security org users grant kubernetes-cluster1  dcos:mesos:master:volume:role:slave_public/kubernetes-cluster1-role create
dcos security org users grant kubernetes-cluster1  dcos:mesos:master:framework:role:slave_public read
dcos security org users grant kubernetes-cluster1  dcos:mesos:agent:framework:role:slave_public read
```

Use the service account and secret when you launch a Kubernetes cluster.  Run the following command to setup a package installer options file that references the service account and secret.

```
cat > cluster1-options.json << EOF
{
  "service": {
    "name": "kubernetes-cluster1",
    "service_account": "kubernetes-cluster1",
    "service_account_secret": "kubernetes-cluster1/sa"
  }
}
EOF
```

Then run the DC/OS Kubernetes CLI command to launch the Kubernetes cluster.

dcos kubernetes cluster create --options=cluster1-options.json --yes

```
By Deploying, you agree to the Terms and Conditions https://mesosphere.com/catalog-terms-conditions/#certified-services
Kubernetes on DC/OS.

	Documentation: https://docs.mesosphere.com/service-docs/kubernetes-cluster
	Issues: https://github.com/mesosphere/dcos-kubernetes-quickstart/issues

Creating Kubernetes cluster kubernetes-cluster1
DC/OS Kubernetes is being installed!
Kubernetes cluster '[kubernetes-cluster1]' is being created
```

You can see the installation runbook automation and status of installation of each component with this command

```
dcos kubernetes manager plan status deploy --name=kubernetes-cluster1
```
First it will show some Kubernetes components completed, and some started or pending like this:

```
$ dcos kubernetes manager plan status deploy --name=kubernetes-cluster1

deploy (serial strategy) (IN_PROGRESS)
├─ etcd (serial strategy) (COMPLETE)
│  └─ etcd-0:[peer] (COMPLETE)
├─ control-plane (dependency strategy) (STARTED)
│  └─ kube-control-plane-0:[instance] (STARTED)
├─ mandatory-addons (serial strategy) (PENDING)
│  └─ mandatory-addons-0:[instance] (PENDING)
├─ node (dependency strategy) (PENDING)
│  └─ kube-node-0:[kubelet] (PENDING)
└─ public-node (dependency strategy) (COMPLETE)
```

When it is completely installed, the plan status should look like this:

```
$ dcos kubernetes manager plan status deploy --name=kubernetes-cluster1

deploy (serial strategy) (COMPLETE)
├─ etcd (serial strategy) (COMPLETE)
│  └─ etcd-0:[peer] (COMPLETE)
├─ control-plane (dependency strategy) (COMPLETE)
│  └─ kube-control-plane-0:[instance] (COMPLETE)
├─ mandatory-addons (serial strategy) (COMPLETE)
│  └─ mandatory-addons-0:[instance] (COMPLETE)
├─ node (dependency strategy) (COMPLETE)
│  └─ kube-node-0:[kubelet] (COMPLETE)
└─ public-node (dependency strategy) (COMPLETE)

```

### Step 4. Install Kubernetes kubectl Command Line

Install the Kubernetes command line by following instructions [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

**For Macs** with brew installed the command is

```
brew install kubectl
```

**For CoreOS** the commands are:
```
curl -O https://storage.googleapis.com/kubernetes-release/release/v1.12.1/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mkdir -p /opt/bin
sudo mv kubectl /opt/bin/kubectl
```

**For Red Red or CentOS** the commands are:
```
curl -O https://storage.googleapis.com/kubernetes-release/release/v1.12.1/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mkdir -p /usr/local/bin
sudo mv kubectl /usr/local/bin/kubectl
```

**For Ubuntu** the commands are:

```
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo touch /etc/apt/sources.list.d/kubernetes.list 
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

Confirm that kubectl is installed and in path /usr/local/bin (it will say it is not connected to dcos cluster yet which is ok)

```
kubectl version
```


-----------


