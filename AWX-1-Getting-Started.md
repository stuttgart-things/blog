# Getting started with AWX
<img src="https://private-user-images.githubusercontent.com/166600787/433764776-cc322bef-5985-4eb4-a6c6-5618d9b2a3e4.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDYzMzcyODIsIm5iZiI6MTc0NjMzNjk4MiwicGF0aCI6Ii8xNjY2MDA3ODcvNDMzNzY0Nzc2LWNjMzIyYmVmLTU5ODUtNGViNC1hNmM2LTU2MThkOWIyYTNlNC5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNTA0JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDUwNFQwNTM2MjJaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT04NjM0MWVlZDUyZmFlMDI3MGE4YzYwMzNmOGI4ZDE2NDg0ZjNiNzdkZTViZjIxNjdlY2M5MzZhNWY4NGVhYjk0JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9._ylOV4eFgSS6A7fVSiHXBN5S9uIluzi9tO4gbAkkcTI" align="center" width="300">

This blog post provides a comprehensive guide to setting up AWX in a local development environment using KIND (Kubernetes IN Docker).

## What is AWX?

AWX is an open-source project that provides a web-based user interface, REST API, and task engine for Ansible. It is the upstream project from which the Red Hat Ansible Automation Platform (AAP) is derived. AWX allows users and organizations to manage Ansible playbooks, inventories, and credentials in a centralized and user-friendly manner. It provides a web-based interface that simplifies the infrastructure automation process and projects, enhancing team collaboration through a structured approach.

<details><summary>Terminology</summary>

Understanding the key terminology in AWX is crucial for effectively using the platform:

|         |                                                                 |
|---------|-----------------------------------------------------------------|
| Organizations    | An Organization is a logical collection of Users, Teams, Projects and Inventories. And it is the highest level in the AWX hierarchy. It also provides a structured way to manage access and permissions within AWX. |
| Playbooks | Playbooks are the YAML files that define a series of tasks to be executed by Ansible. They are used to automate complex workflows and configurations. |
| Inventories | Inventories are lists of hosts or groups of hosts that Ansible manages. They can be defined in static files or dynamically generated from various sources. |
| Credentials| Credentials in AWX are used to authenticate and authorize access to remote systems. AWX seemless integration with various cloud, vault or secret service providers allows the users/teams/organizations to securely store and manage credentials for different environments and use cases. |
| Projects | Projects in AWX are collections of Ansible playbooks and related files. They can be sourced from various version control systems like Git, allowing for versioned and collaborative development of automation scripts. |
| Jobs | Jobs are instances of AWX launching an Ansible playbook against an inventory of hosts. AWX allows the user to schedule, monitor, and manage jobs, providing detailed logs and status information. |
| Job Templates | A job template is a definition and set of parameters for running an Ansible job. They include information about the playbook to run, the inventory to use, and any extra variables or options required for the execution. Job templates are useful to execute the same job many times. |
| Workflows | Workflows are sequences of job templates that can be executed in a specific order, where the templates may or may not share inventory, playbooks, or permissions. They allow you to chain multiple automation tasks together, enabling more complex and conditional automation scenarios. |
| Execution Environments | Execution Environments in AWX are containerized environments that provide the necessary dependencies and runtime for executing Ansible playbooks. They ensure consistency and isolation of the execution context, allowing users to define and manage the specific versions of Ansible and other required tools. Execution Environments can be customized and shared across different teams and projects, facilitating a standardized and reproducible automation process. |

</details>

## Why unsing AWX in KIND?

By leveraging KIND (Kubernetes IN Docker), developers can create a local, isolated environment to experiment with AWX easily and safely. KIND offers simplicity, local development, isolation, consistency with production, resource efficiency, and strong community support.

<details><summary>Install requirements</summary>

The requirements file you need to install:

```yaml
cat <<EOF > requirements.yaml
---
collections:
  - name: community.crypto
    version: 2.26.0
  - name: community.general
    version: 10.5.0
  - name: ansible.posix
    version: 2.0.0
  - name: kubernetes.core
    version: 5.1.0
  - name: community.docker
    version: 4.5.2
  - name: community.vmware
    version: 5.5.0
  - name: awx.awx
    version: 24.6.1
  - name: community.hashi_vault
    version: 6.2.0
  - name: ansible.netcommon
    version: 7.2.0
  - name: https://github.com/stuttgart-things/ansible/releases/download/sthings-container-25.2.474.tar.gz/sthings-container-25.2.474.tar.gz
  - name: https://github.com/stuttgart-things/ansible/releases/download/sthings-baseos-25.2.472.tar.gz/sthings-baseos-25.2.472.tar.gz
  - name: https://github.com/stuttgart-things/ansible/releases/download/sthings-awx-25.2.473.tar.gz/sthings-awx-25.2.473.tar.gz
  - name: https://github.com/stuttgart-things/ansible/releases/download/sthings-rke-25.1.568.tar.gz/sthings-rke-25.1.568.tar.gz
EOF
```

<br>

To install the specified collections and roles, use the following command:

```bash
ansible-galaxy collection install -r ./requirements.yaml -f
```

This command will download and install all the necessary collections and roles defined in the requirements.yaml file, ensuring that your Ansible environment is ready for the subsequent tasks to install KIND and AWX.

</details>

## Create KIND and deploy AWX

### Inventory file

You need an inventory file which defines the hosts and/or groups of hosts that Ansible will manage.

<details><summary>Example Inventory</summary>

The [group1] section lists the IP addresses or hostnames of the remote machines, along with the Ansible user and password for authentication (definable after each host entry or in the defaults section). Alternatively you can only specify one host. The [defaults] section sets global variables, such as disabling host key checking and specifying the Ansible user and password.

```yaml
cat <<EOF > inventory
# Host group
[group1]
<xxx.xxx.x.xx> ansible_user=<user> ansible_password=<password>
<xxx.xxx.x.xx>

# Only one host
<xxx.xxx.x.xx> # Enter IP of the remote machine you want to use

# Variables
[defaults]
host_key_checking = False
ansible_user = <name> # Enter Ansible User
ansible_password = <password> # Enter Ansible User Password
EOF
```

</details>

### Install KIND and AWX with Ansible

Now that you have your inventory file ready, you can proceed with the playbook to install KIND and then AWX. These plays include various tasks, many of them broken down into separate playbooks for better manageability and clarity.

The first play installs KIND and kubernetes resources ( ingress-nginx, cert-manager and cilium ).

```bash
ansible-playbook sthings.container.kind -i inventory -vv
```
<details><summary>Install options:</summary>

Add these arguments to the execution command<br>

```yaml
-e kind_cluster_name=<name> \ # Enter cluster name
-e path_to_kubeconfig=/home/<user>/.kube/<kubeconfig> \ # Enter username and file-name
-e kind_version=0.27.0 \ # change if needed
-e kubectl_version=1.32.3 \ # change if needed
-e count_worker_nodes=3 \ # change if needed
-e count_controlplane_nodes=1 # change if needed
```

</details><br>

When KIND is installed you can execute the next play to install AWX and deploy its resources.

```bash
ansible-playbook sthings.awx.deploy -i inventory -vv
```
<details><summary>Install options:</summary>

Add these arguments to the execution command<br>

```yaml
-e kind_cluster_name=<name> \ # Enter same cluster name of above
-e control_plane_ip=xx.xxx.xxx.xx # Enter ip from last task output from first play
-e ansible_user=<user> # Enter user
```

</details><br>

These commands execute the plays against the host(s) defined in your inventory file (-i <inventory>), with verbose output enabled (-vv). This will provide detailed logs and status information, helping you monitor the execution process and troubleshoot any issues that arise.

<details><summary>Don't forget to export the KUBECONFIG</summary>

```bash
export KUBECONFIG=/home/<user>/.kube/<config>
```
</details>

<details><summary>MANUAL INSTALL</summary>

#### Install Kind

Download the Kind binary from the Kind releases page, make the binary executable and move it to a directory in your $PATH.

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

Create a YAML file with the desired cluster configuration.

<details><summary>Example cluster configuration</summary>

```yaml
cat <<EOF > /tmp/kind-config.yaml
---
kind: Cluster
name: dev
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: True # Disabled because cilium gets deployed, change if needed
#  kubeProxyMode: none
nodes:
  - role: control-plane
    image: kindest/node:v1.33.0.0
    kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: ingress-ready=true
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
  - role: worker
    image: kindest/node:v1.33.0
    extraMounts:
      - hostPath: /mnt/data-node1 # Host directory to mount
        containerPath: /data # Mount path inside the KinD node
  - role: worker
    image: kindest/node:v1.33.0
    extraMounts:
      - hostPath: /mnt/data-node2 # Host directory to mount
        containerPath: /data # Mount path inside the KinD node
  - role: worker
    image: kindest/node:v1.33.0
    extraMounts:
      - hostPath: /mnt/data-node3 # Host directory to mount
        containerPath: /data # Mount path inside the KinD node
EOF
```
</details><br>

Execute this command to create the KIND cluster.

```bash
kind create cluster --config /tmp/kind-config.yaml
```

<details><summary>Don't forget to export the KUBECONFIG</summary>

```bash
export KUBECONFIG=/home/<user>/.kube/<config>
```
</details><br>

The cluster nodes will remain in state **NotReady** until Cilium is deployed. This behavior is expected.

#### KIND provisioning

To deploy cilium, ingress-nginx and cert-manager use these following commands.

##### cilium

<details><summary>Cilium-values</summary>

```yaml
cat <<EOF > /tmp/cilium-values.yaml
---
kubeProxyReplacement: true
routingMode: "native"
ipv4NativeRoutingCIDR: "10.244.0.0/16"
k8sServiceHost: "dev-control-plane"
k8sServicePort: 6443

l2announcements:
  enabled: true
  leaseDuration: "3s"
  leaseRenewDeadline: "1s"
  leaseRetryPeriod: "500ms"

devices: ["eth0", "net0"]

externalIPs:
  enabled: true

autoDirectNodeRoutes: true

operator:
  replicas: 2 # Change if needed
EOF
```

</details><br>

```bash
helm repo add cilium https://helm.cilium.io/

helm install cilium cilium/cilium --version 1.17.2 --namespace kube-system --set image.pullPolicy=IfNotPresent --set ipam.mode=kubernetes # -f /tmp/cilium-values.yaml
```

##### ingress-nginx

<details><summary>Ingress-nginx-values</summary>

```yaml
cat <<EOF > /tmp/ingress-nginx-values.yaml 
---
controller:
  nodeSelector:
    ingress-ready: "true"
    node-role.kubernetes.io/control-plane: ""  # Ensures it runs on the control plane
  tolerations:
    - key: "node-role.kubernetes.io/control-plane"
      operator: "Exists"
      effect: "NoSchedule"

  service:
    type: ClusterIP  # Required only if you want external access
  admissionWebhooks:
    enabled: false  # Avoids potential Kind issues
  hostPort:
    enabled: true  # Enables direct binding to host ports
EOF
```

</details><br>

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm install ingress-nginx ingress-nginx/ingress-nginx --version 4.12.0 --create-namespace --namespace ingress-nginx --disable-openapi-validation # -f /tmp/ingress-nginx-values.yaml 
```

##### cert-manager

```bash
helm repo add jetstack https://charts.jetstack.io

kubectl apply --validate=false -f https://github.com/cert-manager/cert-manager/releases/download/v1.17.2/cert-manager.yaml
# helm install cert-manager jetstack/cert-manager --version v1.17.2 --create-namespace --namespace cert-manager # -f cert-manager-values.yaml

```

Next you need to deploy a clusterissuer:

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned
spec:
  selfSigned: {}
EOF
```

##### awx

```yaml
cat <<EOF > /tmp/awx-values.yaml
AWX:
  enabled: true
  spec:
    service_type: ClusterIP
    ingress_type: ingress 
    hostname: <hostname> # Enter Cluster ip
    ingress_class_name: nginx
EOF
```

```bash
helm repo add awx-operator https://ansible-community.github.io/awx-operator-helm/

helm install awx-operator awx-operator/awx-operator --version 3.1.0 --create-namespace --namespace awx -f /tmp/awx-values.yaml
```

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
  namespace: awx
spec:
  service_type: ClusterIP
  ingress_type: ingress
  hostname: <hostname> # Enter hostname
EOF
```

</details>

To access AWX in your browser you can port-forward:

```bash
kubectl port-forward svc/awx-service -n awx 8080:80
```

Now you can reach AWX in you browser with: http://localhost:8080

If you installed KIND on a VM you can access AWX with the cluster ip.

By following these steps, you can set up a local development environment with KIND and AWX, allowing you to test and learn Ansible automation in a safe and isolated environment. This setup is ideal for developers who want to experiment with AWX without the need for extensive infrastructure, making it a cost-effective and convenient solution.

In Blog Part 2 we will explain how to setup AWX Resources and how to create the first awx-jobs and workflows.
