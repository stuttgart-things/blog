# Don't repeat yourself with GitLab Pipelines

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
