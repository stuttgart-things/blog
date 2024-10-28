# DAGGER

## PREPARATION - INSTALL REQUIREMENTS

```bash
# DOCKER
# DAGGER
```

## GETTING STARTED - CALL FUNCTION (FROM DAGGERVERSE)

```bash
# OUTPUT TEXT
dagger call -m github.com/shykes/daggerverse/hello@v0.1.2 hello --giant=false --name=pat
```

```bash
# SCAN IMAGE REF W/ AQUA TRIVY
dagger call -m github.com/jpadams/daggerverse/trivy@v0.3.0 scan-image --image-ref alpine/git:latest
```
