# /CROSSPLANE - THE (PLATFORM ENGINEERING) ESSENTIAL
⚡️Provision and manage cloud infrastructure and services using kubectl⚡️

## /EXTRACT



## /INTRO
- created by Upbound (released in December 2018)
- incubating CNCF project by the (2020)

## /CONCEPTS
* kubernetes basics
* operators + cr(d)s
* gitops
* cloud

## /REQURIEMENTS AND SCOPE
Terraform	Basiswissen und im besten Fall vorhandener IAC-Code ist von Vorteil für die Nachvollziehbarkeit
Kubernetes	Erfahrung mit der Benutzung von Operatoren ist wünschenswert


## /CONTROL PLANE

- service that watches:
  - a declared state
  - actual state reflects that of the declared
#<img src="https://www.padok.fr/hubfs/reconciliation_loop_crossplane.webp" width="900"/>


## /USECASE MULTICLOUD PROVIDER COMPOSITION

### /GCP PROVIDER - GCP BUCKET

ADD DESCRIPTION HERE

<details><summary>GCP PROVIDER INSTALLATION</summary>

```bash
kubectl apply -f - <<EOF
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-gcp-storage
spec:
  package: xpkg.upbound.io/upbound/provider-gcp-storage:v1.2.0
EOF
```

```bash
# GET GCP CREDENTIALS
# https://cloud.google.com/iam/docs/keys-create-delete?hl=de#creating
kubectl create secret generic gcp-secret -n crossplane-system --from-file=creds=../gcp-credentials.json
EOF
```

```bash
cat ../gcp-credentials.json | grep project_id # USE THIS AS PROJECT ID

kubectl apply -f - <<EOF
apiVersion: gcp.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  projectID: stuttgart-things
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: gcp-secret
      key: creds
EOF
```

</details>

<details><summary>GCP BUCKET CREATION</summary>

```bash
RANDOM_NAME=$(echo "sthings-bucket-"$(head -n 4096 /dev/urandom | openssl sha1 | tail -c 10))

kubectl apply -f - <<EOF
apiVersion: storage.gcp.upbound.io/v1beta1
kind: Bucket
metadata:
  name: example
  labels:
  annotations:
    crossplane.io/external-name: ${RANDOM_NAME}
spec:
  forProvider:
    location: US
    storageClass: MULTI_REGIONAL
  providerConfigRef:
    name: default
  deletionPolicy: Delete
EOF

kubectl get managed
```

</details>

ADD RESULT OF STEP HERE

### /TERRAFORM PROVIDER - VM

ADD DESCRIPTION HERE

<details><summary>TERRAFORM PROVIDER INSTALLATION</summary>

```bash
kubectl apply -f - <<EOF
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-gcp-storage
spec:
  package: xpkg.upbound.io/upbound/provider-gcp-storage:v1.2.0
EOF
```

</details>

<details><summary>TERRAFORM PROVIDER CONFIGURATION (GCP STATE)</summary>

```bash
kubectl apply -f - <<EOF
apiVersion: tf.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: gcp-tuesday-test1
  namespace: crossplane-system
spec:
  credentials:
    - filename: gcp-credentials.json
      source: Secret
      secretRef:
        namespace: crossplane-system
        name: gcp-secret
        key: creds
  configuration: |
    terraform {
      backend "gcs" {
        bucket  = "sthings-bucket-f0b3f1ee7"
        prefix  = "terraform/state"
        credentials = "gcp-credentials.json"
      }
    }
```

</details>

<details><summary>TERRAFORM WORRKSPACE DEFINITION</summary>

```bash
apiVersion: tf.upbound.io/v1beta1
kind: Workspace
metadata:
  annotations:
    crossplane.io/composition-resource-name: vsphere-vm
    crossplane.io/external-name: vsphere-vm
  labels:
    crossplane.io/claim-name: tuesday-test1
    crossplane.io/claim-namespace: crossplane-system
    crossplane.io/composite: tuesday-test1-knqpq
  name: tuesday-test1-knqpq-s92xd
spec:
  deletionPolicy: Delete
  forProvider:
    entrypoint: ""
    module: git::https://github.com/stuttgart-things/vsphere-vm.git?ref=v1.7.5-2.7.0
    source: Remote
    varFiles:
    - format: HCL
      secretKeyRef:
        key: terraform.tfvars
        name: vsphere-tfvars
        namespace: crossplane-system
      source: SecretKey
    vars:
    - key: vm_count
      value: "1"
    - key: vsphere_vm_name
      value: tuesday-test1
    - key: vm_memory
      value: "4096"
    - key: vm_disk_size
      value: "64"
    - key: vm_num_cpus
      value: "8"
    - key: firmware
      value: bios
    - key: vsphere_vm_folder_path
      value: stuttgart-things/testing
    - key: vsphere_datacenter
      value: /LabUL
    - key: vsphere_datastore
      value: /LabUL/datastore/UL-ESX-SAS-01
    - key: vsphere_resource_pool
      value: /LabUL/host/Cluster-V6.5/Resources
    - key: vsphere_network
      value: /LabUL/network/LAB-10.31.103
    - key: vsphere_vm_template
      value: /LabUL/vm/stuttgart-things/vm-templates/sthings-u23
    - key: bootstrap
      value: '["echo STUTTGART-THINGS"]'
    - key: annotation
      value: VSPHERE-VM BUILD w/ CROSSPLANE FOR STUTTGART-THINGS
    - key: unverified_ssl
      value: "true"
  managementPolicies:
  - '*'
  providerConfigRef:
    name: gcp-tuesday-test1
  writeConnectionSecretToRef:
    name: tuesday-test1
    namespace: crossplane-system
```

</details>

ADD RESULT OF STEP HERE

### /KUBERNETES PROVIDER - TEKTON/ANSIBLE

ADD DESCRIPTION HERE

<details><summary>KUBERNETES PROVIDER INSTALLATION</summary>

```bash
kubectl apply -f - <<EOF

EOF
```

</details>

<details><summary>KUBERNETES PROVIDER CONFIGURATION</summary>

```bash
kubectl apply -f - <<EOF

EOF
```

</details>

<details><summary>KUBERNETES OBJECT DEFINITION</summary>

```bash
kubectl apply -f - <<EOF

EOF
```

</details>


### /COMPOSITION

### /XRD

### /USAGE

<details><summary>USAGE RESOURCE</summary>

```bash
kubectl apply -f - <<EOF
apiVersion: apiextensions.crossplane.io/v1alpha1
kind: Usage
metadata:
  name: vspherevm-uses-bucket
spec:
  of:
    apiVersion: storage.gcp.upbound.io/v1beta1
    kind: Bucket
    resourceSelector:
      matchLabels:
        crossplane.io/claim-namespace: crossplane-system
  by:
    apiVersion: tf.upbound.io/v1beta1
    kind: Workspace
    resourceSelector:
       matchLabels:
        crossplane.io/claim-namespace: crossplane-system

EOF
```

</details>


### /CLAIM


### /ADVANTAGES OF (USING) CROSSPLANE

* Developer-friendly: By building on Kubernetes, Crossplane leverages a developer-friendly API. Also, when you combine it with Argo CD or Flux, you can apply GitOps principles in their full glory—combining orchestration, observability, declarative IaC, containers, immutable infrastructure, and DevOps best practices using GitOps as a single source of truth.

Satisfies both Infra Ops and App Opps needs: Through its Compositions, Crossplane makes both Infra Ops teams and App Ops teams happy. As one article explains, while Infra Ops teams “understand how to provision cloud provider-specific components, App Ops teams know the application requirements and understand what is required from the Application Infrastructure perspective
