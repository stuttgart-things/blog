# Don't repeat yourself with GitLab Pipelines - Use components!

When working with GitLab pipelines, one of the core principles to ensure efficiency and maintainability is "Don't Repeat Yourself" (DRY). Repetitive code not only increases maintenance overhead but also introduces the risk of inconsistencies across different parts of a project. GitLab's move from CI/CD templates to CI/CD components is a perfect example of applying DRY principles.

## ONCE UPON A TIME: GITLAB TEMPLATES
GitLab CI/CD templates are predefined YAML files that provide reusable and shareable configurations for setting up CI/CD pipelines in GitLab. These templates were designed to simplify the process of defining pipelines by offering pre-configured stages, jobs, and scripts for common tasks, eliminating the need to write everything from scratch.

## 2024: USE GITLAB COMPONENTS
CI/CD components are the next generation of CI/CD templates. CI/CD components are reusable, single-purpose building blocks that abstract away pipeline configuration units. With the introduction of the CI/CD Catalog, GitLab is no longer accepting contributions of new CI/CD templates to the codebase. Instead CI/CD components for the catalog should be created. This transition enhances the modularity and maintainability of shared CI/CD resources, and avoids the complexities of contributing new CI/CD templates. 

CI/CD components are similar to the other kinds of configuration added with the include keyword, but have several advantages:
* Components can be listed in the CI/CD Catalog.
* Components can be released and used with a specific version.
* Multiple components can be defined in the same project and versioned together.
* This leads to:
  * Reusability and abstraction
  * Flexibility with input
  * High-quality standards through testing

## TEMPLATE VS. COMPONENT

|     | Component (custom)         | Component (GitLab) | To Be Continous          | Template |
| --- | -------------------------- | ------------------ | ------------ | -------- |
| Description     | Include custom files from another private project on the same GitLab instance to the pipeline| Add a modular [GitLab CI/CD component](https://docs.gitlab.com/ee/ci/components/index.html) to the pipeline | Set of GitLab CI templates developed and maintained by DevOps and technology experts to build state-of-the-art CI/CD pipelines in minutes | Predefined YAML files that provide reusable and shareable configurations for setting up CI/CD pipelines|
| Include Keyword | ```include:project``` or ```include:file``` | ```include:component``` | ```include:component``` to use it as a CI/CD component; ```include:project``` to use it as a regular template | ```include:template``` |
| Source          | custom project or file path | [GitLab Components](https://gitlab.com/components) or [Codehub Components](https://codehub.sva.de/components/) | [TBC Generator](https://to-be-continuous.gitlab.io/kicker/) or [TBC Repo](https://codehub.sva.de/to-be-continuous) | [CI-Templates](https://gitlab.com/gitlab-org/gitlab/-/tree/master/lib/gitlab/ci/templates) |
| Configuration   | [inputs](https://docs.gitlab.com/ee/ci/components/index.html#use-a-component) | [inputs](https://docs.gitlab.com/ee/ci/components/index.html#use-a-component) | with [inputs](https://docs.gitlab.com/ee/ci/components/index.html#use-a-component) if using the include:component technique \* or with [variables](https://docs.gitlab.com/ee/ci/variables/) if using include:project or include:remote | [variables](https://docs.gitlab.com/ee/ci/variables/)                                                                                                                                                                                                                                    |


## REUSE A (CUSTOM) PIPELINE COMPONENT

### example - build ko image

PIPELINE COMPONENT DEFINITION

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

COMPONENT CALL

```yaml
include:
  - project: Lab/stuttgart-things/stuttgart-things
    file: build/gitlab/build-ko-image.yaml
    ref: master
```


### example - include the component located in the current project from the current SHA

```yaml
include:
  - component: $CI_SERVER_FQDN/$CI_PROJECT_PATH/my-component@$CI_COMMIT_SHA
    inputs:
      stage: build

stages: [build, test, release]
```


## USE GITLAB COMPONENT

### example - use the full go pipeline
```yaml
include:
  - component: codehub.sva.de/components/go/full-pipeline@97f5a6f4811246faa07892e75a17c4c9f7f9c2e3
    inputs:
      go_image: 'golang:latest'

stages: [build]
```

### example - only use the build part of the go pipleine
```yaml
include:
  - component: codehub.sva.de/components/go/build@97f5a6f4811246faa07892e75a17c4c9f7f9c2e3
    inputs:
      go_image: 'golang:latest'

stages: [build]
```

## USE TO BE CONTIUOUS 

### Use as a CI/CD component

```yaml
include:
  - component: codehub.sva.de/to-be-continuous/golang/gitlab-ci-golang@4.11.0
    inputs:
      image: "registry.hub.docker.com/library/golang:buster"
```

### Use as a CI/CD template

```yaml
include:
  - project: 'to-be-continuous/golang'
    ref: '4.11.0'
    file: '/templates/gitlab-ci-golang.yml'

variables:
  GO_IMAGE: "registry.hub.docker.com/library/golang:buster"

```


## USE GITLAB TEMPLATES

```yaml
include:
  - template: Jobs/SAST.gitlab-ci.yml

variables:
  SCAN_KUBERNETES_MANIFESTS: "true"

```



