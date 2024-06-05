# /CROSSPLANE - PLATFORM ENGINEERING ESSENTIAL NUMBER1
⚡️Provision and manage cloud infrastructure and services using kubectl⚡️

## /INTRO
- created by Upbound (released in December 2018)
- incubating CNCF project by the (2020)

## /CONCEPTS
- kubernetes basics
- operators + cr(d)s
- gitops
- cloud

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


### /KUBERNETES PROVIDER - TEKTON/ANSIBLE

ADD DESCRIPTION HERE

<details><summary>KUBERNETES PROVIDER INSTALLATION</summary>

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


### /COMPOSITION

### /XRDS

### /USAGE

### /CLAIM
