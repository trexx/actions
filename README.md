# actions
GitHub Action Workflows for Org

## Workflows

### build.yml

Builds a Docker image with Buildx and pushes it to GHCR, tagged and labelled by `docker/metadata-action` (which also sets `org.opencontainers.image.source`, linking the package to the repository). Prunes old images afterwards unless `prune_images: false`.

```yaml
jobs:
  build:
    uses: 'trexx/actions/.github/workflows/build.yml@main'
    permissions:
      contents: 'read'
      packages: 'write'
    with:
      # all optional
      images: 'ghcr.io/${{ github.repository }}-capture'
      context: '{{defaultContext}}:server'
      target: 'capture'
      tags: 'type=match,pattern=capture-(.*),group=1'
      flavor: 'latest=true'
      labels: 'org.opencontainers.image.title=capture'
      platforms: 'linux/amd64,linux/arm64'
      push: ${{ github.event_name != 'pull_request' }}
      cache_scope: 'capture'
      prune_images: false
```

Inputs: `images`, `context`, `target`, `tags`, `flavor`, `labels`, `platforms`, `push`, `cache_scope`, `prune_images`.

### rust-check.yml

`cargo fmt --check` (once), `cargo clippy -D warnings` and `cargo test`, on one or more runners.

```yaml
jobs:
  check:
    uses: 'trexx/actions/.github/workflows/rust-check.yml@main'
    permissions:
      contents: 'read'
    with:
      # all optional
      os: '["ubuntu-26.04", "windows-2025"]'
      toolchain: 'stable'
      working_directory: 'proxy'
      cargo_args: '--locked'
      apt_packages: 'libudev-dev'
      fmt: true
      build_release: false
```

### build-rust.yml

On a `v*` tag: checks the tag matches `Cargo.toml`'s version, builds `--release --locked` and attaches `files` to the GitHub release. Needs `contents: write`.

```yaml
jobs:
  build:
    uses: 'trexx/actions/.github/workflows/build-rust.yml@main'
    permissions:
      contents: 'write'
    with:
      files: |
        target/release/app.exe
      runs_on: 'windows-latest' # default
```

### trivy.yml / trivy-fs.yml

Scan the published `ghcr.io/<repo>:latest` image, or the checked-out repository, and upload the results to the Security tab.

### rendered-manifests.yml

Pushes the repository as a signed Flux OCI artifact to `ghcr.io/<owner>/<repo>`.


### build-test-publish.yml

Builds a Docker image, runs an optional test command against it, and on the publish branch pushes it when its contents differ from `:latest`. Tags: `latest`, `<version>`, `<version>-<run number>`, where `<version>` defaults to the tag of the Dockerfile's final `FROM` image.

```yaml
on:
  push:
    branches: [ '**' ]
  schedule:
    - cron: '17 4 * * 1'
  workflow_dispatch:

jobs:
  build:
    uses: 'trexx/actions/.github/workflows/build-test-publish.yml@main'
    permissions:
      contents: 'read'
      packages: 'write'
    with:
      test_command: './test.sh "${TEST_IMAGE}"'
```

Inputs: `context`, `image`, `test_command`, `version_command`, `publish_branch`, `prune_images`, `keep_n_tagged`.

## Actions

### setup-android

JDK from `gradle/gradle-daemon-jvm.properties`, the Android SDK with the given packages, and Gradle (caching and wrapper validation). Run it after `actions/checkout`.

```yaml
- uses: 'trexx/actions/.github/actions/setup-android@main'
  with:
    packages: 'platforms;android-37.0 build-tools;36.0.0' # optional
    project_directory: 'android' # optional
```

### decode-keystore

Writes a base64-encoded keystore secret to `$RUNNER_TEMP/<filename>`. Outputs `path` and `present`; optionally exports the path in `env_var`.

```yaml
- id: 'keystore'
  uses: 'trexx/actions/.github/actions/decode-keystore@main'
  with:
    keystore_base64: '${{ secrets.KEYSTORE_BASE64 }}'
    required: 'false'
    env_var: 'KEYSTORE_FILE'
    missing_message: 'no KEYSTORE_BASE64 secret: falling back to the debug key'
```

### enable-kvm

Lets the runner user open `/dev/kvm`, for emulators and VMs on Linux runners.

```yaml
- uses: 'trexx/actions/.github/actions/enable-kvm@main'
```


### prune-ghcr

Deletes old images from the repository's GHCR package: untagged and broken images, orphaned signatures and attestations, and all but the newest `keep_n_tagged` tagged images. Tags in `exclude_tags` (default `latest`) are never deleted. Needs `packages: write`.

```yaml
- uses: 'trexx/actions/.github/actions/prune-ghcr@main'
  with:
    keep_n_tagged: 2
```

## Renovate preset

`default.json` holds the settings every repository shares: the Dependency Dashboard, and leaving `trexx/**` actions and workflows on `main` instead of pinning them.

```json
{
  "extends": ["github>trexx/actions"]
}
```
