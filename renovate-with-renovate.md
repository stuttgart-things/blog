# Renovate w/ renovate

[//]: # (Abstract)
[//]: # (Links)
[//]: # (Intro)

## Introduction

Renovate updates dependencies in the code without needing to do it manually. Renovate runs on the repo and looks for references to dependencies (both public and private). If there are newer versions available, Renovate can create pull requests to update versions automatically.

[//]: # (ADD A SCREENSHOT FROM A PR)

## Integrated Automated Dependency Updates

[//]: # (TABLE?)

* GoLang Dependencies
* Dockerfile
* Helm-Charts - Public
* OCI Dependencies [HELM](https://docs.renovatebot.com/modules/manager/helmv3)
* Flux Dependencies (Preview Envs?)
* ArgoCD Dependencies (Preview Envs?)

## RENOVATE vs. DEPANDABOT
* Large projects with complex dependencies or monorepos.
* Cross-platform support requirements, or when using multiple package managers.
* Teams that want more control over scheduling, grouping, and merging of dependency updates.
* Organizations that need self-hosted options or more control over security and compliance.
* Teams that prefer advanced automation and detailed changelogs.

## RENOVATE DRY-RUN

```bash
cat <<EOF > config.json
{
    "repositories": ["stuttgart-things/clusterbook"],
    "dryRun" : "full"
}
EOF

docker run --rm -v "$(pwd)/config.json:/opt/renovate/config.json"`\
-e RENOVATE_PLATFORM=github \
-e RENOVATE_TOKEN=${GITHUB_TOKEN} \
-e LOG_LEVEL=debug \
-e RENOVATE_CONFIG_FILE=/opt/renovate/config.json \
renovate/renovate
```

https://github.com/renovatebot/renovate/blob/main/docs/usage/examples/self-hosting.md

## SETUP RENOVATE

1. Add Renovate to Your Repository
First, enable Renovate for your repository. If you're using GitHub, GitLab, or Bitbucket, this usually involves adding the Renovate app to your repository through the platform’s marketplace or Renovate’s official website.

2. Create a renovate.json configuration file in the root of your repository to tell Renovate how to handle updates. Here’s a basic setup for a Docker-based project:

```json
{
  "extends": [
    "config:base"
  ],
  "dockerfile": {
    "fileMatch": ["Dockerfile", "Dockerfile.*"],
    "enabled": true
  },
  "packageRules": [
    {
      "managers": ["dockerfile"],
      "matchDatasources": ["docker"],
      "groupName": "docker dependencies"
    }
  ]
}
```

3. Run Renovate Locally (Optional)

```bash
docker run --rm -v "$(pwd):/usr/src/app" renovate/renovate
```


## Custom Dependency Updates

[//]: # (Add sthings ansible example)

```bash
cat <<EOF > renovate.json
{
    "$schema": "https://docs.renovatebot.com/renovate-schema.json",
    "extends": [
        "config:recommended"
    ],
    "customManagers": [
        {
            "customType": "regex",
            "fileMatch": [
                "\\.yaml$"
            ],
            "matchStrings": [
                "\\s*(?<binName>.+)_version:\\s(?<currentValue>v?\\d+\\.\\d+\\.\\d+)\\s*#\\s*(datasource=(?<datasource>[^\\s]*))\\s*(depName=(?<depName>[^\\s]*))?\\s*"
            ],
            "versioningTemplate": "{{#if versioning}}{{{versioning}}}{{else}}semver{{/if}}",
            "extractVersionTemplate": "^v?(?<version>.*)$",
            "datasourceTemplate": "{{#if datasource}}{{{datasource}}}{{else}}github-releases{{/if}}",
            "depNameTemplate": "{{#if depName}}{{{depName}}}{{else}}{{{binName}}}{{/if}}"
        },
        {
            "customType": "regex",
            "fileMatch": ["^Dockerfile$"],
            "matchStrings": [
                "\\s*(?<binName>.+)_version:\\s(?<currentValue>v?\\d+\\.\\d+\\.\\d+)\\s*#\\s*(datasource=(?<datasource>[^\\s]*))\\s*(depName=(?<depName>[^\\s]*))?\\s*"
            ],
            "versioningTemplate": "{{#if versioning}}{{{versioning}}}{{else}}semver{{/if}}",
            "extractVersionTemplate": "^v?(?<version>.*)$",
            "datasourceTemplate": "{{#if datasource}}{{{datasource}}}{{else}}github-releases{{/if}}",
            "depNameTemplate": "{{#if depName}}{{{depName}}}{{else}}{{{binName}}}{{/if}}"
      }
    ],
    "customDatasources": {
      "hashicorp": {
        "defaultRegistryUrlTemplate": "https://api.releases.hashicorp.com/v1/releases/{{packageName}}?license_class=oss",
        "transformTemplates": [
          "{ \"releases\": $map($, function($v) { { \"version\": $v.version, \"releaseTimestamp\": $v.timestamp_created, \"changelogUrl\": $v.url_changelog, \"sourceUrl\": $v.url_source_repository } }), \"homepage\": $[0].url_project_website, \"sourceUrl\": $[0].url_source_repository }"
        ]
      }
    }
}
EOF
```

```
vars:
  - name: tools
    file: |
      ---
      kind_version: 0.25.0 # datasource=github-tags depName=kubernetes-sigs/kind
      skopeo_version: 1.14.4 # datasource=github-tags depName=lework/skopeo-binary
      helm_version: 3.16.2 # datasource=github-tags depName=helm/helm
      kubectl_version: v1.30.2 # datasource=github-tags depName=kubernetes/kubectl
      k9s_version: v0.32.5 # datasource=github-tags depName=kubernetes/kubectl
      velero_version: 1.15.0 # datasource=github-tags depName=vmware-tanzu/velero
      kubectl_slice_version: 1.4.0 # datasource=github-tags depName=patrickdappollonio/kubectl-slice
      helmfile_version: 0.169.1 # datasource=github-tags depName=helmfile/helmfile
      argocd_version: 2.13.0 # datasource=github-tags depName=argoproj/argo-cd
      flux_version: 2.4.0 # datasource=github-tags depName=fluxcd/flux2
      glab_version: 1.48.0 # datasource=gitlab-tags depName=gitlab-org/cli
      cilium_version: 0.16.19 # datasource=gitlab-tags depName=cilium/cilium-cli
      dagger_version: 0.13.3 # datasource=gitlab-tags depName=dagger/dagger

      bin:
        flux:
          bin_name: flux
          bin_version: "{{ flux_version }}"
          check_bin_version_before_installing: true
          source_url: "https://github.com/fluxcd/flux2/releases/download/v{{ flux_version }}/flux_{{ flux_version }}_linux_amd64.tar.gz"
          bin_to_copy: flux
          to_remove: ""
          bin_dir: "/usr/bin/flux"
          version_cmd: "version"
          target_version: "{{ flux_version }}"
```

## GitHub Integration


![image](https://github.com/user-attachments/assets/952cacb2-8f05-4a37-8322-89fb5e6f8ded)



## GitLab Integration

```json
{
    "platform": "gitlab",
    "endpoint": "https://companyhub.de/api/v4",
    "token": ${GITAB_TOKEN} ,
    "repositories": ["Lab/stuttgart-things/homerun/homerun-gitlab-pitcher"] ,
    "dryRun": true,
    "hostRules": [
      {
        "hostType": "docker",
        "matchHost": ${HOST},
        "username": ${USER},
        "password": ${PASSWORD} 
      }
    ],
    "logLevel": "info",  // Set to debug for detailed logs
    // "packageRules": [
    //   {
    //     "matchPackageNames": ["*"],
    //     "enabled": true,
    //     "groupName": "All Updates" // Optional: To group all updates together
    //   }
    // ]
  }
```

Run Renovate

```bash
docker run --rm -e RENOVATE_CONFIG_FILE=config.json -v "$(pwd)/config.json:/usr/src/app/config.json" renovate/renovate
```

Renovate will create an Issue called "Dependency Dashboard" in GitLab with the findings.


## Local / Testing

```json
{
    "platform": "gitlab",
    "endpoint": "https://companyhub.sva.de/api/v4",
    "token": ${GITLAB_TOKEN},
    "repositories": ["Lab/stuttgart-things/homerun/homerun-gitlab-pitcher"] ,
    "dryRun": true,
    "hostRules": [
      {
        "hostType": "docker",
        "matchHost": ${HOST},
        "username": ${USER},
        "password": ${PASSWORD} 
      }
    ],
    "logLevel": "debug",  // Set to debug for detailed logs
    // "packageRules": [
    //   {
    //     "matchPackageNames": ["*"],
    //     "enabled": true,
    //     "groupName": "All Updates" // Optional: To group all updates together
    //   }
    // ]
  }
```

Run Renovate Locally

```bash
docker run --rm -e RENOVATE_CONFIG_FILE=config.json -e LOG_LEVEL=debug -v "$(pwd)/config.json:/usr/src/app/config.json" renovate/renovate
```
## CompanyHub Configuration

Renovate Runner Onboarding:
* add Renovate Service Account to repo or group
* wait for next pipeline schedule to get onboarding pr
* accept the onboarding pr in the repos
* wait for next pipeline schedule to get dep prs

## Conclusion

Renovate can be used quickly with little configuration to regularly update dependencies. It helps to keep the application up to date at all times. This contributes to the error-free and reliable operation of the application.

[//]: # (outro)
