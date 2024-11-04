# DAGGER

## PREPARATION - INSTALL REQUIREMENTS

```bash
# DOCKER
# DAGGER
```

## GETTING STARTED - CALL FUNCTION ON SHELL (FROM DAGGERVERSE)

```bash
# OUTPUT TEXT
dagger call -m github.com/shykes/daggerverse/hello@v0.1.2 hello --giant=false --name=pat
```

```bash
# SCAN IMAGE REF W/ AQUA TRIVY
dagger call -m github.com/jpadams/daggerverse/trivy@v0.3.0 scan-image --image-ref alpine/git:latest
```


## FUNCTIONS

### CREATE A NEW FUNCTION

```bash
dagger init --sdk=go --source=./dagger
```

### CREATE FUNCTION WITH RETURN

the following function is using a golang base image and builds a binary

```go
												// dagger call build --progress plain --src ./ export --path build (returned dir(path))
func (m *Clusterbook) Build(ctx context.Context, src *dagger.Directory) *dagger.Directory {

	// get `golang` image
	golang := dag.Container().From("golang:latest")

	// mount cloned repository into `golang` image
	golang = golang.WithDirectory("/src", src).WithWorkdir("/src")

	// define the application build command
	path := "build/"
	golang = golang.WithExec([]string{"env", "GOOS=linux", "GOARCH=amd64", "go", "build", "-o", path, "./main.go"})
}
```

### CALL FUNCTION

```bash
dagger functions
dagger call build --progress plain --src ./ export --path build
```

### CREATE FUNCTION WHICH IS BASED ON A IMPORT WITHOUT A RETURN

```go
func (m *Clusterbook) Scan(ctx context.Context, image string) {

	golang := dag.Container().From(image)

	scanResult, _ := dag.Trivy().ScanContainer(ctx, golang, dagger.TrivyScanContainerOpts{
		TrivyImageTag: trivyImageTag,
		Severity:      "HIGH,CRITICAL",
		Format:        "table",
		ExitCode:      0,
	})

	fmt.Println(scanResult)
}
```

```
dagger install github.com/jpadams/daggerverse/trivy
dagger call scan --progress plain --image nginx:latest
```
