# CROSSPLANE TERRAFORM PROVIDER: IAC ON STEROIDS

EXTRACT:
* crossplane enables orchestrating applications and infrastructure no matter where they run
* kubernetes declarative API is used to define the desired state of infrastructure -> crossplane takes care of the rest
* resources are manageable across multiple clouds or clusters without creating code

<ADD INTRO HERE>

## PREFACE: CROSSPLANE TERMINOLOGY

PROVIDER: A provider extends Crossplane by installing controllers for new kinds of managed resources
XRD (Composite Resource Definition): define new types of composite resources and claims
CLAIM XRC”, or just “a claim” is also an API type defined using Crossplane. Each type of claim corresponds to a type of composite resource, and the pair have nearly identical schemas.

## PREPARATION: CROSSPLANE DEPLOYMENT

##  DEPLOYMENT

<details><summary><b>CLI INSTALLATION</b></summary>

```bash
curl -sL "https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh" | sh
sudo mv crossplane /usr/local/bin
```

</details>

<details><summary><b>DEPLOYMENT W/ HELM</b></summary>

[provider-helm](https://github.com/crossplane-contrib/provider-helm/tree/master)

```bash
kubectl create namespace crossplane-system
helm repo add crossplane-stable https://charts.crossplane.io/stable && helm repo update

helm upgrade --install crossplane --wait \
--namespace crossplane-system \
crossplane-stable/crossplane --version 1.14.5

kubectl api-resources | grep upbound
```

</details>

## PREPARATION: TERRAFORM PROVIDER DEPLOYMENT

<details><summary><b>TERRAFORM PROVIDER DEPLOYMENT</b></summary>

```yaml
# CONTROLLER REF
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-terraform
spec:
  package: xpkg.upbound.io/upbound/provider-terraform:v0.13.0
  controllerConfigRef:
    name: cert-bundle
```

</details>

## PREPARATION: TERRAFORM PROVIDER CONFIGURATION


## HELLO-WORLD: INLINE-WORKSPACE



## HELLO-WORLD: REMOTE MODULE-CALL
