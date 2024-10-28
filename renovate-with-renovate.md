# Renovate w/ renovate

[//]: # (Extract)
[//]: # (Links)
[//]: # (Intro)

## Integrated Automated Dependency Updates  

* GoLang Dependencies
* Dockerfile
* Helm-Charts - Public
* OCI Dependencies [HELM](https://docs.renovatebot.com/modules/manager/helmv3)
* Flux Dependencies (Preview Envs?)
* ArgoCD Dependencies (Preview Envs?)

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

## GitLab Integration

## GitHub Integration

## Local / Testing

[//]: # (outro)

