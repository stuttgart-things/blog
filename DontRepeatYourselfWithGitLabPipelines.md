# Don't repeat yourself with GitLab Pipelines

## REUSE A (CUSTOM) PIPELINE COMPONENT

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


## USE GITLAB JOBS

```yaml
include:
  - template: Jobs/SAST-IaC.gitlab-ci.yml # template
```


### Include Pipeline Examples (tbc)

```yaml
include:
  - template: Jobs/SAST-IaC.gitlab-ci.yml # template
  - component: codehub.sva.de/to-be-continuous/golang/gitlab-ci-golang@4.10.0 # to be continous
    inputs:
      sbom-disabled: true
      build-mode: "application"
  - component: codehub.sva.de/components/go/full-pipeline@97f5a6f4811246faa07892e75a17c4c9f7f9c2e3 # component
  - project: Lab/stuttgart-things/stuttgart-things # self written
    file: build/gitlab/build-ko-image.yaml
    ref: master
```
