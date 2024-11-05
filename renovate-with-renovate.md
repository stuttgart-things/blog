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

## GitLab Integration

```json
{
    "platform": "gitlab",
    "endpoint": "https://codehub.sva.de/api/v4",
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

## GitHub Integration

## Local / Testing

```json
{
    "platform": "gitlab",
    "endpoint": "https://codehub.sva.de/api/v4",
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


[//]: # (outro)
