# github-common-actions
Reusable composite actions used from other Weaviate workflows to avoid code duplication

## weaviate-version-in-image

### Description
Retrieves the Weaviate version from a given Weaviate Docker image.

### Inputs
- `image_tag` (required): The Weaviate Docker image to retrieve the version from. Default: `latest`.
- `registry` (optional): The Docker registry and namespace to pull the image from. Default: `docker.io/semitechnologies/weaviate`.

### Outputs
- `weaviate_version`: The Weaviate version from the image tag.

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


## get-latest-weaviate-version

### Description
Retrieves the latest Weaviate version available from GitHub releases.

### Inputs
None

### Outputs
- `latest_weaviate_version`: The latest Weaviate version available from GitHub releases.

### Usage
```yaml
name: Test Get Latest Weaviate Version Action

on:
  push:

jobs:
  version-latest:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      
      - name: Test get-latest-weaviate-version action
        id: latest-version
        uses: weaviate/github-common-actions/.github/actions/get-latest-weaviate-version@main
      
      - name: Output latest Weaviate version
        run: echo "Latest Weaviate version: ${{ steps.latest-version.outputs.latest_weaviate_version }}"
```


## get-previous-version

### Description
Retrieves a previous version of Weaviate based on specified criteria, allowing for patch or minor version jumps.

### Inputs
- `weaviate_version` (required): The current Weaviate version to start from
- `jump_type` (required): Type of version jump to perform
  - `patch`: Get the previous patch version within the same minor version
  - `minor`: Get a version from a previous minor version
- `jumps` (optional): Number of minor versions to jump back when `jump_type` is 'minor'. Default: '1'

### Outputs
- `previous_version`: The previous Weaviate version based on the specified criteria

### Usage
```yaml
name: Get Previous Weaviate Version
on: [push]

jobs:
  get-version:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Get Previous Patch Version
        uses: weaviate/github-common-actions/.github/actions/get-previous-version@main
        id: get-version
        with:
          weaviate_version: '1.24.21'
          jump_type: 'patch'

      - name: Get Previous Minor Version
        uses: weaviate/github-common-actions/.github/actions/get-previous-version@main
        id: get-minor-version
        with:
          weaviate_version: '1.25.21'
          jump_type: 'minor'
          jumps: '1'

      - name: Output Versions
        run: |
          echo "Previous patch version: ${{ steps.get-version.outputs.previous_version }}"
          echo "Previous minor version: ${{ steps.get-minor-version.outputs.previous_version }}"
```

### Examples
- For version `1.24.21` with `jump_type: 'patch'` → returns `1.24.20`
- For version `1.25.21` with `jump_type: 'minor'` and `jumps: '1'` → returns latest `1.24.x` version
- For version `1.26.17` with `jump_type: 'minor'` and `jumps: '2'` → returns latest `1.24.x` version

### Runs
This action runs using `composite` with the following steps:
1. **Set up Python**: Sets up Python 3.11 environment
2. **Install Dependencies**: Installs required Python packages (requests, semver)
3. **Get Previous Version**: Fetches available versions from GitHub releases and returns the appropriate previous version based on the specified criteria

