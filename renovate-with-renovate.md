# Renovate w/ renovate

[//]: # (Abstract)
[//]: # (Links)
[//]: # (Intro)

## Introduction

Nowadays, an application quickly contains several dozen libraries, also known as dependencies. These regularly publish new versions, sometimes several times a week. Dependencies should be updated regularly in order to benefit promptly from the advantages of a new version. 

Renovate updates dependencies in the code without needing to do it manually and thus makes dependencies planable. Renovate runs on the repository and looks for references to dependencies (both public and private). If there are newer versions available, Renovate can create pull requests to update versions automatically.

Renovate is a self-hosted tool that runs in CI/CD pipelines and works with GitHub, GitLab, and many more. In this article we focus on the setup in GitHub and GitLab after comparing Renovate Bot with Depandabot. We will also give some examples for defining custom dependency updates.

## RENOVATE vs. DEPANDABOT

In addition to Renovate, Dependabot is also a useful and popular tool that can automate the updating and patching of software dependencies. The following section compares the tools and describes when it makes sense to use Renovate.

In the table bellow some of the main features of Renovate and Dependabot are listed:

| Feature                                   | Renovate                                                                                                                       | Dependabot                                                                                                                                                                   |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dependency Dashboard                      | Yes                                                                                                                            | No                                                                                                                                                                           |
| Officially supported platforms            | GitHub, GitLab, Bitbucket, Azure, Gitea                                       | GitHub only                                                                                                                                                                  |
| Show changelogs                           | Yes                                                                                                                            | Yes                                                                                                                                                                          |
| Built-in to GitHub                        | No, requires app or self-hosting                                                                                               | Yes                                                                                                                                                                          |
| Scheduling                                | By default, Renovate runs as often as it is allowed to | `daily`, `weekly`, `monthly`                                                                                                                                            |

                                                                    
When to prefer Renovate?

Large projects with complex dependencies or monorepos.
Cross-platform support requirements, or when using multiple package managers.
Teams that want more control over scheduling, grouping, and merging of dependency updates.
Organizations that need self-hosted options or more control over security and compliance.
Teams that prefer advanced automation and detailed changelogs.

## Integrated Automated Dependency Updates

* GoLang Dependencies
* Dockerfile
* Helm-Charts - Public
* OCI Dependencies [HELM](https://docs.renovatebot.com/modules/manager/helmv3)
* Flux Dependencies 
* ArgoCD Dependencies 

## SETUP RENOVATE

In the following, it is shown how to set up Renovate and how to start a so-called dry-run, to preview the behavior of Renovate in logs, without making any changes to the repository files.

Two files are of central importance for the setup of Renovate: renovate.json and config.json.

renovate.json is a repository-specific configuration file. It defines how Renovate behaves for a particular repository.

Bellow is a basic setup of the renovate.json file:



```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended"
  ]
}
```

config.json is a global configuration file for Renovate, typically used when running Renovate self-hosted. It defines the default behavior for all repositories Renovate manages.

An easy way for getting started with renovate is to create a configuration file and to perform a dry run on the source code / repository. In the next section example config.json files for Renovate dry-run are given.

## RENOVATE DRY-RUN
With the dry-run option Renovate runs only locally without creating Merge Requests. If the LOG_LEVEL is set to debug, the detected dependencies are displayed in the console.

### CompanyHub Example

This is a basic config.json file to setup Renovate dry-run for a CompanyHub Repository:

```json
{
    "platform": "gitlab",
    "endpoint": "https://companyhub.de/api/v4",
    "token": "<YOUR_TOKEN>",
    "repositories": ["<YOUR_REPO>"],
    "dryRun": "full"
  }
  ```

After creating the file the Renovate Bot can be started with docker with the following command:

```json
docker run --rm -e RENOVATE_CONFIG_FILE=config.json \
-e LOG_LEVEL=debug \
-v "$(pwd)/config.json:/usr/src/app/config.json" \
renovate/renovate
 ```

### GitHub Example

Renovate can also be setup in GitHub. Here is a basic config.json file for dry-run, the "service-golang" is just an example:

```json
{
    "repositories": ["gbeletti/service-golang"],
    "token": "<YOUR_TOKEN>",
    "dryRun" : "full"
 
}
 ```

It can also be started with docker with the following command:

```json
docker run --rm -v "$(pwd)/config.json:/opt/renovate/config.json" \
-e RENOVATE_PLATFORM=github \
-e LOG_LEVEL=debug \
-e RENOVATE_CONFIG_FILE=/opt/renovate/config.json \
renovate/renovate
 ```

## Integration of Renovate in GitLab

In the following section, example merge requests and issues created by Renovate in GitLab are shown.

In the screenshot bellow the initial merge request to configure Renovate in a repository is shown. After merging the bot runs in the defined schedule to discover dependencies.

![Configure_RenovateMR](https://github.com/user-attachments/assets/251f9b12-fbe6-47fc-8789-3c842324377c)


### Example Merge Request for dependency created by Renovate:

This screenshot shows a merge request created by Renovate for a detected dependency. After it has been merged the version will be updated.

![example_mr](https://github.com/user-attachments/assets/d5eca45b-0b48-4a0e-a212-91cafd4c3822)


### Dependency Dashboard:

The Dependency Dashboard lists all currently open MRs. It also contains a list of all detected dependencies and is also used to communicate any problems found. For example, if Renovate cannot resolve certain dependencies or if a misconfiguration is detected, these are reported here. The dashboard can also be used to interact with the bot.

![Dependency_Dashboard](https://github.com/user-attachments/assets/a482e889-a287-4e9e-9a0e-fe2eac0acff5)


### Post Upgrade Tasks

With Post Upgrade Tasks Renovate can perform further tasks when creating PRs (any shell commands possible), for example notification of the team via mail or execute linting.

An alternative to Post Upgrade Tasks are Post Update Options, which comes with 12 preconfigured actions (npm/yarn, go, Ruby bundler, Helm), e.g. “go mod tidy” or “helm dependency update”.


### RegEx Manager

With the help of the RegEx Manager dependencies in custom file formats that are not supported by the supplied managers can be found. For example shell scripts (curl ...), or RUN in a Dockerfile or GitHub Actions parameters or custom JSON/YAML file formats.


# GitLab Integration

There are various ways in which Renovate can access a repository or several repositories. If Renovate should only run in one repository, a GitLab access token with api and write_repository access is sufficient. If Renovate is to take care of several repositories, it is advisable to create a Renovate user in GitLab and add this user as a member (maintainer or developer) to the relevant projects.

GitLab example

This example shows a basic config.json file for a GitLab repo. As dryRun is not set, the Bot will create Merge Requests in the GitLab repo for the detected dependencies. Renovate will also create the Issue "Dependency Dashboard" in GitLab with the findings.

```json
{
    "platform": "gitlab",
    "endpoint": "https://gitlab.com/api/v4",
    "token": "<TOKEN>",
    "repositories": ["<YOUR_GITLAB_REPOSITORY>"]
  }
```

As already mentioned Renovate can be started with docker:

```bash
docker run --rm -e RENOVATE_CONFIG_FILE=config.json -v "$(pwd)/config.json:/usr/src/app/config.json" renovate/renovate
```

# Renovate in CI/CD Pipeline (GitLab)

If Renovate is to take care of several repositories an extra repo for the Renovate pipeline can be created.

In the following example Renovate runs in a Docker Container. In the CI/CD settings in GitLab the scheduler can be configured to set the intervall pattern.

```yaml
#.gitlab-ci.yml
 
stages:
  - dependency_updates
 
renovate:
  image: renovate/renovate:slim
  stage: dependency_updates
  variables:
    RENOVATE_PLATFORM: gitlab
    RENOVATE_ENDPOINT: $CI_API_V4_URL
    RENOVATE_AUTODISCOVER: "true"
    RENOVATE_BINARY_SOURCE: install
    LOG_LEVEL: debug
  tags:
    - docker
  only:
    - schedules
  script:
    - renovate $RENOVATE_EXTRA_FLAGS
```

The following environment variabels can bet set, among others:

RENOVATE_AUTODISCOVER: This flag specifies that Renovate should automatically search all repositories to which it has access for dependencies.
RENOVATE_BINARY_SOURCE: Renovate uses this to install third-party tools that it needs to be able to perform updates, e.g. npm, yarn, ...
After creating the pipeline the Renovate user can be added as maintainer or developer in a repository to activate Renovate. If the environment variable Autodiscover is set to true in the Renovate job, Renovate will automatically find and analyze the repository the next time it runs. In the renovate.json the configuration for the repository can be added.

# GitHub Integration
The easiest way to integrate Renovate into GitHub:

* Visit the Renovate GitHub App page.
* Click Install and select the repositories or organizations where you want to enable Renovate.
* Renovate will create the renovate.json in your repository or use the default settings.
* Add a renovate.json to your repository to customize Renovate's behavior

  ![image](https://github.com/user-attachments/assets/f31ab5e3-0db7-48e7-86c4-5ac1de5c4dd4)

# Define Custom Dependency Updates
For specialized dependencies update rules (which may not fit standard update workflows) the renovate concepts customManagers and customDatasources can be used.

In the following (shortend) example we want to get updates from GitHub and GitLab releases for an ansible playbook which can be used for installing binaries. The comment after the version is used as a marker for renovate for inserting an version update in the given structure/variable.

```yaml
vars:
  - name: tools
    file: |
      ---
      kind_version: 0.25.0 # datasource=github-tags depName=kubernetes-sigs/kind
      skopeo_version: 1.14.4 # datasource=github-tags depName=lework/skopeo-binary
      helm_version: 3.16.2 # datasource=github-tags depName=helm/helm
      kubectl_version: v1.30.2 # datasource=github-tags depName=kubernetes/kubectl
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

The following configuration shows how a regex based customManager for the custom ansible format can be defined. The custom Datasource is used for getting updates from hashicorp releases (which will be published on their api).

```yaml
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

# Conclusion
Renovate offers a broad spectrum of functionality. Its package manager support, customization for complex project needs, and seamless integration with external vulnerability databases empower developers to implement a proactive security posture. Additionally, Renovate’s granular pull request control streamlines code review and collaboration. Renovate can be used quickly with little configuration to regularly update dependencies. This is especially important to eliminate security vulnerabilities. It furthermore helps to keep the application up to date at all times. This contributes to the error-free and reliable operation of the application. Overwriting transitive dependencies is often technically difficult or impossible. Updating dependencies is the task of the respective maintainer. But there are dedicated tools for detecting unsafe dependencies, e.g.: OSS Review Toolkit, Trivy, OSV Scanner and Dependabot Alerts.

