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

## get-latest-branches

### Description
Returns the latest N version branches (e.g. `stable/v1.38`) of a repository, ordered oldest → newest, with optional inclusion of the `main` branch. The primary output is a JSON array designed to feed a `matrix` strategy via `fromJSON()`.

### Inputs
- `repository` (optional): The GitHub repository (`owner/name`) to enumerate version branches from. Default: `weaviate/weaviate`.
- `count` (optional): Number of most-recent version branches to return. Default: `3`.
- `branch_prefix` (optional): Branch namespace to enumerate (e.g. `stable/` matches `stable/vX.Y` branches). Default: `stable/`.
- `include_main` (optional): If `'true'`, append the `main` branch as the newest entry. Default: `'false'`.

### Outputs
- `branches_json`: JSON array of branch names, newest last (e.g. `["stable/v1.37","stable/v1.38"]`). Use with `fromJSON()` for matrix strategies.
- `branches`: Space-separated branch names, newest last (e.g. `stable/v1.37 stable/v1.38`). Convenient for shell `for`-loops.
- `latest`: The single newest version branch (e.g. `stable/v1.38`). Independent of `include_main`, so it always points at the highest release branch.

### Usage

#### Matrix strategy (primary use case)
```yaml
name: Test Across Latest Weaviate Branches
on: [push]

jobs:
  resolve-branches:
    runs-on: ubuntu-latest
    outputs:
      branches_json: ${{ steps.branches.outputs.branches_json }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Get latest branches
        id: branches
        uses: weaviate/github-common-actions/.github/actions/get-latest-branches@main
        with:
          count: '3'
          include_main: 'true'

  test:
    needs: resolve-branches
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        branch: ${{ fromJSON(needs.resolve-branches.outputs.branches_json) }}
    steps:
      - name: Show branch
        run: echo "Testing against ${{ matrix.branch }}"
```

#### Shell loop
```yaml
      - name: Get latest branches
        id: branches
        uses: weaviate/github-common-actions/.github/actions/get-latest-branches@main
        with:
          count: '5'

      - name: Iterate
        run: |
          for branch in ${{ steps.branches.outputs.branches }}; do
            echo "Branch: ${branch}"
          done
          echo "Latest: ${{ steps.branches.outputs.latest }}"
```

### Examples
- `count: '3'` against `weaviate/weaviate` → `["stable/v1.36","stable/v1.37","stable/v1.38"]`
- `count: '3'` with `include_main: 'true'` → `["stable/v1.36","stable/v1.37","stable/v1.38","main"]`
- `count: '1'` → `["stable/v1.38"]`

### Behavior
- Uses `git ls-remote` against the public repository — no token required.
- Branches are version-sorted by git (`ls-remote --sort='v:refname'`), so `stable/v1.9` correctly precedes `stable/v1.10`, and the newest branches are kept. Using git's sort avoids depending on GNU `sort -V`, which is unavailable on macOS/BSD runners.
- `main` (when `include_main: 'true'`) is always appended last, since it is ahead of every release branch.
- The action fails (exit 1) if `count` is not a positive integer, or if no matching version branches are found — preventing a silently empty matrix.

## get-branch-docker-tag

### Description
Resolves a Weaviate git branch (`main` or `stable/vX.Y`) to the **latest available** multi-arch Docker tag published to Docker Hub (e.g. `1.38.0-rc.1-8269596`). Weaviate's CI publishes a multi-arch tag `<version>-<short-sha>` for built commits only, so the newest commit on a branch frequently has no image yet — this action walks back the branch history and returns the most recent commit that *does* have a published image, guaranteeing the returned tag is pullable.

Pairs with [`get-latest-branches`](#get-latest-branches): use that to list the branches, then this to resolve each branch to its newest image.

### Inputs
- `branch` (required): Git branch to resolve, e.g. `main` or `stable/v1.38` (as emitted by `get-latest-branches`). A leading `refs/heads/` is stripped.
- `repository` (optional): GitHub repository (`owner/name`) whose commits and `openapi-specs/schema.json` are read. Default: `weaviate/weaviate`.
- `registry` (optional): Docker Hub repository (`namespace/name`) whose tags are checked. Default: `semitechnologies/weaviate`.
- `gh_token` (optional): GitHub token for the commits API (avoids rate limits). Recommended. Default: `''`.
- `max_depth` (optional): How many commits back from the branch tip to search before failing. Must be `1`-`100` (the GitHub commits API caps a page at 100). Default: `30`.

### Outputs
- `docker_tag`: Latest available multi-arch semver tag, e.g. `1.38.0-rc.1-8269596`.
- `version`: The version part of the tag, e.g. `1.38.0-rc.1`.
- `sha`: 7-char short commit the image was built from, e.g. `8269596`.
- `commit`: Full 40-char commit SHA the image was built from.
- `is_fallback`: `'true'` if the branch tip had no published image and an older commit was used, otherwise `'false'`.
- `commits_behind`: How many commits behind the branch tip the resolved image is (`0` = the tip itself).

### Usage

#### Single branch
```yaml
name: Resolve Latest Image
on: [push]

jobs:
  resolve:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Resolve latest image for main
        id: tag
        uses: weaviate/github-common-actions/.github/actions/get-branch-docker-tag@main
        with:
          branch: main
          gh_token: ${{ secrets.GITHUB_TOKEN }}

      - name: Pull it
        run: |
          echo "Pulling semitechnologies/weaviate:${{ steps.tag.outputs.docker_tag }}"
          docker pull "semitechnologies/weaviate:${{ steps.tag.outputs.docker_tag }}"
```

#### Composed with `get-latest-branches` (matrix over the newest branches)
```yaml
jobs:
  branches:
    runs-on: ubuntu-latest
    outputs:
      branches_json: ${{ steps.b.outputs.branches_json }}
    steps:
      - uses: actions/checkout@v4
      - id: b
        uses: weaviate/github-common-actions/.github/actions/get-latest-branches@main
        with:
          count: '3'
          include_main: 'true'

  resolve:
    needs: branches
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        branch: ${{ fromJSON(needs.branches.outputs.branches_json) }}
    steps:
      - uses: actions/checkout@v4
      - id: tag
        uses: weaviate/github-common-actions/.github/actions/get-branch-docker-tag@main
        with:
          branch: ${{ matrix.branch }}
          gh_token: ${{ secrets.GITHUB_TOKEN }}
      - run: echo "${{ matrix.branch }} -> ${{ steps.tag.outputs.docker_tag }} (behind ${{ steps.tag.outputs.commits_behind }})"
```

### Behavior
- Walks the branch's commit history newest-first; for each commit it reads `<version>` from `openapi-specs/schema.json` at that commit and checks whether `semitechnologies/weaviate:<version>-<short-sha>` exists in the registry, returning the first one that does.
- Only commits on the branch's own version line are considered (for `stable/vX.Y`, the `X.Y` line; for `main`/other, the tip's major.minor); off-line ancestor commits are skipped so a fallback can't return e.g. a `1.35.x` image for `stable/v1.36`.
- Uses the registry's **exact-tag** endpoint (reliable) rather than tag listings (which time out for common prefixes), and anchors to the branch's own commits so the result is branch-precise (the bare semver tag alone collides across branches that share a version).
- Fails (exit 1) if `max_depth` is not an integer between 1 and 100, if `repository`/`registry` is empty, if the branch is empty or does not exist, or if no published image is found within `max_depth` commits.
- Fails closed: if Docker Hub returns an ambiguous status (e.g. 5xx/429/000) for a candidate tag after retries, the action errors rather than treating it as "missing" and silently falling back to an older image.
- No Docker auth is required (public repo); a `gh_token` is recommended only to avoid GitHub commits-API rate limits.

