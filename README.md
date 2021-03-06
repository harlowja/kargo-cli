Kargo wrapper
=============

This tool helps to deploy a kubernetes cluster with ansible.

**Note**: The following choices are done automatically for redundancy.
According to the number of nodes on your cluster:

-   The 2 firsts nodes will have master components installed
-   The 3 firsts nodes will be members of the etcd cluster

You should have at least 3 nodes but you can spawn only one instance for
tests purposes.

Example on GCE:
[![asciicast](https://asciinema.org/a/5in6riip9w93r4zzunlv2vib8.png)](https://asciinema.org/a/5in6riip9w93r4zzunlv2vib8?speed=4)

Requirements
============

-   **Ansible v2.x**
-   The current user must have its ssh **public key** installed on the
    remote servers.
-   The remote user (option --user) must be in the sudoers with no
    password

Installation
============

### Python pip

    sudo pip2 install kargo


### Docker image
Alternatively you can use the docker image `k8s-kargocli` as follows:

    docker run -it -v /home/smana/kargoconf:/etc/kargo quay.io/smana/k8s-kargocli:latest /bin/bash

The mounted directory contains kargo's configuration as well as keys

Config file
-----------

A config file can be updated (yaml). (default: */etc/kargo/kargo.yml* ) </br>
This file contains default values for some parameters that don't change
frequently </br>
**Note** these values are **overwritten** by the command line.


    # Common options
    # ---------------
    # Path where the kargo ansible playbooks will be installed
    # Defaults to current user's home directory if not set
    # kargo_path: "/tmp"
    # Default inventory path
    kargo_git_repo: "https://github.com/kubespray/kargo.git"
    # Logging options
    loglevel: "info"
    #
    # Google Compute Engine options
    # ---------
    machine_type: "n1-standard-1"
    image: "debian-8-kubespray"
    service_account_email: "kubespray-ci-1@appspot.gserviceaccount.com"
    pem_file: "/home/smana/kargo.pem"
    project_id: "kubespray-ci-1"
    zone: "us-east1-c"
    ...

Basic usage
-----------

### Generate inventory for a baremetal cluster

If the servers are already available you can use the argument **prepare**
The command below will just clone the git repository and creates the
inventory.
The hostvars must be separated by a **comma without spaces**

    kargo preprare --nodes node1[ansible_ssh_host=10.99.21.1] node2[ansible_ssh_host=10.99.21.2] node3[ansible_ssh_host=10.99.21.3]

### Run instances and generate the inventory on Clouds

**AWS**

In order to create vms on AWS you can either edit the config file */etc/kargo/kargo.yml* or set the options with the argument **aws**
if the config file is filled with the proper information you just need to run the following command

    kargo aws --instances 3

Another example which download kargo's repo in a defined directory and set the cluster name

    kargo aws --instances 3 -p /tmp/mykargo --cluster-name foobar


**GCE**

In order to create vms on GCE you can either edit the config file */etc/kargo/kargo.yml* or set the options with the argument **gce**
if the config file is filled with the proper information you just need to run the following command

    kargo gce --instances 3

Another example if you already have a kargo repository in your home dir

    kargo gce --instances 3 --noclone --cluster-name foobar

**Add a node to an existing cluster**
It's possible to add nodes to a running cluster, </br>
these newly added nodes will act as node only (no etcd, no master components)

Add a node

    kargo [aws|gce] --add --instances 1

Then deploy the cluster with the same options as the running cluster.


### Deploy cluster

The last step is to run the cluster deployment.

**Note**:
-   default network plugin : flannel (vxlan) default
-   default kargo\_path : "/home/\<current\_user\>/kargo"
-   inventory path : "\<kargo\_path\>/inventory/inventory.cfg".
-   The option `--inventory` allows to use an existing inventory (file or dynamic)
-   On coreos (--coreos) the directory **/opt/bin** must be writable
- You can use all Ansible's variables with
`--ansible-opts '-e foo=bar -e titi=toto -vvv'` (the value must be enclosed by simple quotes)

some examples:

Deploy with the default options on baremetal

    kargo deploy

Deploy on AWS using a specific kargo directory and set the api password

    kargo deploy --aws --passwd secret -p /tmp/mykargo -n weave

Deploy a kubernetes cluster on CoreOS servers located on GCE

    kargo deploy -u core -p /kargo-dc1 --gce --coreos --cluster-name mykube --kube-network 10.42.0.0/16
