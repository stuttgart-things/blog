# Don't repeat yourself with GitLab Pipelines

## REUSE A (CUSTOM) PIPELINE COMPONENT

## TEMPLATE VS. COMPONENT

CI/CD components are the next generation of CI/CD templates, enhancing pipeline creation and maintenance. CI/CD components are reusable, single-purpose building blocks that abstract away pipeline configuration units.

CI/CD components are similar to the other kinds of configuration added with the include keyword, but have several advantages:
* Components can be listed in the CI/CD Catalog.
* Components can be released and used with a specific version.
* Multiple components can be defined in the same project and versioned together.
* This leads to:
  * Reusability and abstraction
  * Flexibility with input
  * High-quality standards through testing

## OVERVIEW TABLE 
* include component vs include project


### PIPELINE COMPONENT DEFINITION

example - build ko image

```yaml
spec:
  inputs:
    stage:
      default: build
---
ko-build:
  image: ghcr.io/ko-build/ko:df484d2df9da64a3fa89f2cb3b6286b21be04dac
  stage: $[[ inputs.stage ]]
  before_script:
    - export KO_DOCKER_REPO=$KO_DOCKER_REPO
  script:
    - ko login $REGISTRY -u $REGISTRY_USER -p $REGISTRY_PASSWORD
    - ko build $KO_REPO --insecure-registry
  tags:
    - sthings
```
### COMPONENT CALL


```yaml
include:
  - project: Lab/stuttgart-things/stuttgart-things
    file: build/gitlab/build-ko-image.yaml
    ref: master
```


example - include the component located in the current project from the current SHA

```yaml
include:
  - component: $CI_SERVER_FQDN/$CI_PROJECT_PATH/my-component@$CI_COMMIT_SHA
    inputs:
      stage: build

stages: [build, test, release]
```


## USE GITLAB COMPONENT

* how to define inputs
* what about stages? 

```yaml
include:
  - component: codehub.sva.de/components/go/full-pipeline@97f5a6f4811246faa07892e75a17c4c9f7f9c2e3
```

## USE TO BE CONTIUOUS 

https://to-be-continuous.gitlab.io/kicker/
[to-be-continuous](https://gitlab.com/to-be-continuous)

### PIPELINE CALL

* HOW TO HANDLE STAGES
* HOW TO OVERWRITE VARIABLES
* WHERE DO I KNOW WHICH VARIABLES EXISTS?

```yaml
include:
  - component: codehub.sva.de/to-be-continuous/golang/gitlab-ci-golang@4.10.0 # to be continous
    inputs:
      sbom-disabled: true
      build-mode: "application"
```


## USE GITLAB TEMPLATES

```yaml
include:
  - template: Jobs/SAST.gitlab-ci.yml

variables:
  SCAN_KUBERNETES_MANIFESTS: "true"

```



