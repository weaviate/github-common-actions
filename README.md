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

## capture-logs

### Description
Captures and manages logs from either Kubernetes pods or Docker containers. For Kubernetes, it uses stern to capture logs from pods matching specific labels. For Docker, it uses docker compose logs to capture logs from all services defined in a docker-compose file.

### Inputs
- `stern_version` (optional): The version of stern to install when using Kubernetes mode. Default: '1.30.0'
- `action` (optional): The action to perform. Options: 'start' or 'stop'. Default: 'start'
- `log_file_name` (optional): Name of the file where logs will be captured. Default: 'weaviate_pods.log'
- `namespace` (optional): Kubernetes namespace to capture logs from (only used when type=kubernetes). Default: 'weaviate'
- `selector` (optional): Kubernetes label selector for pods (only used when type=kubernetes). Default: 'app=weaviate'
- `type` (optional): Type of logging to perform. Options: 'kubernetes' or 'docker'. Default: 'kubernetes'
- `docker_compose_file` (optional): Path to docker-compose.yml file (only used when type=docker). Default: 'docker-compose.yml'

### Usage

#### Kubernetes Mode
```yaml
name: Capture Kubernetes Logs
on: [push]

jobs:
  capture-logs:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      # Start capturing logs
      - name: Start capturing Kubernetes logs
        uses: weaviate/github-common-actions/.github/actions/capture-logs@main
        with:
          stern_version: '1.30.0'
          action: 'start'
          log_file_name: 'weaviate.log'
          namespace: 'weaviate'
          selector: 'app=weaviate'
          type: 'kubernetes'

      # Your test steps here...

      # Stop capturing logs
      - name: Stop capturing logs
        uses: weaviate/github-common-actions/.github/actions/capture-logs@main
        with:
          action: 'stop'
          type: 'kubernetes'
        if: always()
```

#### Docker Mode
```yaml
name: Capture Docker Logs
on: [push]

jobs:
  capture-logs:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      # Start capturing logs
      - name: Start capturing Docker logs
        uses: weaviate/github-common-actions/.github/actions/capture-logs@main
        with:
          action: 'start'
          log_file_name: 'docker.log'
          type: 'docker'
          docker_compose_file: 'docker-compose.yml'

      # Your test steps here...

      # Stop capturing logs
      - name: Stop capturing logs
        uses: weaviate/github-common-actions/.github/actions/capture-logs@main
        with:
          action: 'stop'
          type: 'docker'
        if: always()
```

### Behavior
- For Kubernetes mode:
  1. Installs stern
  2. Starts capturing logs from pods matching the namespace and selector
  3. Writes logs to the specified log file
  4. Can be stopped using the stop action

- For Docker mode:
  1. Verifies the docker-compose file exists
  2. Starts capturing logs from all services in the docker-compose file
  3. Writes logs to the specified log file
  4. Can be stopped using the stop action

### Notes
- The action runs in the background and continues capturing logs until explicitly stopped
- Logs are written to `/tmp/{log_file_name}`
- For Kubernetes mode, stern is used to capture logs from multiple pods simultaneously
- For Docker mode, docker compose logs is used to capture logs from all services
- Always use the stop action in a cleanup step (with `if: always()`) to ensure logs are properly stopped
- The docker-compose file must be accessible in the workspace when using Docker mode

