# /CROSSPLANE - THE (PLATFORM ENGINEERING) ESSENTIAL
⚡️Provision and manage cloud infrastructure and services using kubectl⚡️

<img src="https://media.licdn.com/dms/image/C4E22AQEd16vto8HKOA/feedshare-shrink_800/0/1671131117163?e=2147483647&v=beta&t=Ac815Z-pKEYJu6Hkve5HCGSi9timgGpEUS4rpSco624" width="600"/>


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

<details><summary>TERRAFORM WORKSPACE DEFINITION</summary>

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

<img src="https://i.redd.it/5l2a72cmdss91.png" width="600"/>

### /KUBERNETES PROVIDER - TEKTON/ANSIBLE

ADD DESCRIPTION HERE

<details><summary>KUBERNETES PROVIDER INSTALLATION</summary>

```bash
kubectl apply -f - <<EOF
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-kubernetes
spec:
  package: xpkg.upbound.io/crossplane-contrib/provider-kubernetes:v0.14.0
EOF
```

</details>

<details><summary>PROVIDER CONFIG (KUBECONFIG)</summary>

```bash
# CREATE KUBECONFIG SECRET FROM LOCAL FILE
kubectl -n crossplane-system create secret generic kubeconfig-dev43 --from-file=/home/sthings/.kube/dev-cicd
```

```bash
kubectl apply -f - <<EOF
apiVersion: kubernetes.crossplane.io/v1alpha1
kind: ProviderConfig
metadata:
  name: dev-cicd
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: kubeconfig-dev-cicd
      key: dev-cicd
EOF
```

</details>

<details><summary>KUBERNETES OBJECT DEFINITION</summary>

```bash
kubectl apply -f - <<EOF
apiVersion: kubernetes.crossplane.io/v1alpha2
kind: Object
metadata:
  name: tekton-baseos
spec:
  deletionPolicy: Delete
  forProvider:
    manifest:
      apiVersion: tekton.dev/v1
      kind: PipelineRun
      metadata:
        name: baseos-dev17-9
        namespace: tektoncd
      spec:
        params:
        - name: ansibleWorkingImage
          value: eu.gcr.io/stuttgart-things/sthings-ansible:10.0.1
        - name: createInventory
          value: "true"
        - name: gitRepoUrl
          value: https://github.com/stuttgart-things/stuttgart-things.git
        - name: gitRevision
          value: main
        - name: gitWorkspaceSubdirectory
          value: /ansible/workdir/
        - name: vaultSecretName
          value: vault
        - name: installExtraRoles
          value: "true"
        - name: ansibleExtraRoles
          value:
          - https://github.com/stuttgart-things/install-requirements.git,2024.05.11
          - https://github.com/stuttgart-things/manage-filesystem.git,2024.05.15
          - https://github.com/stuttgart-things/install-configure-vault.git
          - https://github.com/stuttgart-things/create-send-webhook.git,2024-06-06
        - name: ansiblePlaybooks
          value:
          - ansible/playbooks/prepare-env.yaml
          - ansible/playbooks/base-os.yaml
        - name: ansibleVarsFile
          value:
          - manage_filesystem+-true
          - update_packages+-true
          - install_requirements+-true
          - install_motd+-true
          - username+-sthings
          - lvm_home_sizing+-'15%'
          - lvm_root_sizing+-'35%'
          - lvm_var_sizing+-'50%'
          - send_to_msteams+-true
          - reboot_all+-false
        - name: ansibleVarsInventory
          value:
          - all+["dev17.labul.sva.de"]
        - name: ansibleExtraCollections
          value:
          - https://github.com/stuttgart-things/stuttgart-things/releases/download/0.5.0/sthings-base_os-0.5.0.tar.gz
          - community.crypto:2.10.0
        - name: installExtraCollections
          value: "true"
        pipelineRef:
          params:
          - name: url
            value: https://github.com/stuttgart-things/stuttgart-things.git
          - name: revision
            value: main
          - name: pathInRepo
            value: stageTime/pipelines/execute-ansible-playbooks.yaml
          resolver: git
        workspaces:
        - name: shared-workspace
          volumeClaimTemplate:
            spec:
              accessModes:
              - ReadWriteOnce
              resources:
                requests:
                  storage: 20Mi
              storageClassName: openebs-hostpath
  providerConfigRef:
    name: dev-cicd
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


LINKS:
https://www.codecentric.de/wissens-hub/blog/crossplane-configuration-packages-github
https://aaroneaton.com/walkthroughs/crossplane-package-testing-with-kuttl/
https://info.codecentric.de/hubfs/Softwerker/Softwerker%20Vol.%2024/Softwerker_Vol_24.pdf?utm_campaign=Softwerker%20Vol.%2024&utm_medium=email&_hsenc=p2ANqtz-_qlCO67oCtBxnu4FVF_cOMjQPuJGTy01-N4Th07b11ZGMCbkp6tC89hyP8B3oe3B2JM_2bLKryzPmi29XvtwNiVLkJ1g&_hsmi=308424267&utm_content=308424267&utm_source=hs_email&hsCtaTracking=340cf8bf-40b5-4ff0-8f3b-412292ef74a9%7C960fb87b-dff5-409c-8745-f801169ac171
https://www.codecentric.de/wissens-hub/blog/testing-crossplane-compositions-kuttl
