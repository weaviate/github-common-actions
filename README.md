# github-common-actions
Reusable composite actions used from other Weaviate workflows to avoid code duplication

## weaviate-version-in-image

### Description
Retrieves the Weaviate version from a given Weaviate Docker image.

### Inputs
- `image_tag` (required): The Weaviate Docker image to retrieve the version from. Default: `latest`.
- `registry` (optional): The Docker registry and namespace to pull the image from. Default: `docker.io/semitechnologies/weaviate`.

### Usage
```yaml
name: Retrieve Weaviate Version
on: [push]

jobs:
    retrieve-version:
        runs-on: ubuntu-latest
        steps:
            - name: Checkout repository
              uses: actions/checkout@v2

            - name: Retrieve Weaviate Version
              uses: weaviate/github-common-actions/.github/actions/weaviate-version-in-image@main
              id: weaviate-version
              with:
                image_tag: 'latest'
                registry: 'docker.io/semitechnologies/weaviate'

            - name: Output Weaviate Version
              run: echo "Weaviate Version: ${{ steps.weaviate-version.outputs.weaviate_version }}"
```

### Runs
This action runs using `composite` with the following steps:
1. **Set globals**: Sets global variables for action timeout and port.
2. **Start Weaviate service**: Starts the Weaviate Docker container.
3. **Wait for .well-known/ready API to be ready**: Waits until the Weaviate service is ready.
4. **Set the Weaviate version as an output**: Retrieves and sets the Weaviate version as an output.