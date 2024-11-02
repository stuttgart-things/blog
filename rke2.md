# ANSIBLE AUTOMATION RANCHER

## REQUIREMENTS

```
cat <<EOF > requirements.yml
---
collections:
  - name: community.crypto
    version: 2.20.0
  - name: community.general
    version: 9.1.0
  - name: ansible.posix
    version: 1.5.4
  - name: kubernetes.core
    version: 5.0.0
  - name: https://github.com/stuttgart-things/stuttgart-things/releases/download/1.28.10/sthings-deploy_rke-1.28.10.tar.gz
EOF

ansible-galaxy collection install -r requirements.yml
```

## RANCHER-HA SERVER

```bash
# CREATE INVENTORY
cat <<EOF > rke2
[initial_master_node]
10.100.136.151
[additional_master_nodes]
10.100.136.152
10.100.136.153
EOF
```

``` bash
# PLAYBOOK CALL
CLUSTER_NAME=rke2
ansible-playbook sthings.deploy_rke.rke2 \
-i rke2 -vv \
-e rke2_fetched_kubeconfig_path=~/.kube/${CLUSTER_NAME} \
-e cluster_setup=multinode \
-e 1.28.10 \
-e rke2_release_kind=rke2r1
```

## RANCHER-DEPLOYMENT:

### METALLB

```bash
cat <<EOF > ~/projects/rke2/metallb.yaml # absolute path needed for deployment_vars
---
ip_range: 10.31.103.18-10.31.103.18
EOF
```

```bash
ansible-playbook sthings.deploy_rke.deploy_to_k8s \
-e deployment_vars=~/projects/rke2/metallb.yaml \
-e path_to_kubeconfig=~/.kube/rke2 \
-e profile=metallb \
-e state=present \
-vv
```

### INGRESS-NGINX

```bash
ansible-playbook sthings.deploy_rke.deploy_to_k8s \
-e path_to_kubeconfig=~/.kube/rke2 \
-e profile=ingress-nginx \
-e state=present \
-vv
```

### CERT-MANAGER

```bash
cat <<EOF > ~/projects/rke2/cert-manager.yaml # absolute path needed for deployment_vars
---
approle_id: <SECRET>
approle_secret: <SECRET>
ca_bundle: <SECRET>
name_cluster_issuer: cluster-issuer-approle
pki_path: pki/sign/sthings-vsphere.labul.sva.de
vault_server: https://vault-vsphere.labul.sva.de:8200
vault_secret: vault-approle
EOF
```

```bash
ansible-playbook sthings.deploy_rke.deploy_to_k8s \
-e path_to_kubeconfig=~/.kube/rke2 \
-e profile=cert-manager \
-e state=present \
-e deployment_vars=~/projects/rke2/cert-manager.yaml \
-vv
```

### RANCHER

```bash
cat <<EOF > ~/projects/rke2/cert-manager.yaml # absolute path needed for deployment_vars
---
hostname: rancher-things
domain: demo-rancher.sthings-vsphere.labul.sva.de
ca_certs: <SECRET>
password: "{{ lookup('community.general.random_string', length=16) }}"
EOF
```

```bash
ansible-playbook sthings.deploy_rke.deploy_to_k8s \
-e path_to_kubeconfig=~/.kube/rke2 \
-e profile=rancher \
-e state=present \
-e deployment_vars=~/projects/rke2/cert.yaml \
-vv
```

### API-TOKEN

```bash
ansible-playbook sthings.deploy_rke.api_token \
-e path_to_kubeconfig=~/.kube/rke2 \
-vv \
-e profile=rancher \
-e state=present \
-e deployment_vars=~/projects/rke2/cert.yaml
```

### DOWNSTEAM-CLUSTER

```bash
10.31.104.124 rancher_cluster_cmd="--controlplane --etcd --worker"

[all:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```
