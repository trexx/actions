# actions
GitHub Action Workflows for Org

Repositories reference these workflows and actions by `@main`. Third-party actions are pinned here, never in the calling repositories.

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

## Pinned third-party actions

Repositories use third-party actions only through these wrappers, so every third-party pin lives in this repository and one Renovate PR here updates all of them. Each wrapper passes the upstream action's inputs (with the same defaults) and outputs through unchanged; use it exactly like the upstream action:

```yaml
- uses: 'trexx/actions/.github/actions/checkout@main'
  with:
    submodules: 'recursive'
```

| Wrapper | Upstream | Pinned at |
|---|---|---|
| `cache` | [`actions/cache`](https://github.com/actions/cache) | v6.1.0 |
| `cache-nix` | [`nix-community/cache-nix-action`](https://github.com/nix-community/cache-nix-action) | v7 |
| `checkout` | [`actions/checkout`](https://github.com/actions/checkout) | v7.0.1 |
| `create-pull-request` | [`peter-evans/create-pull-request`](https://github.com/peter-evans/create-pull-request) | v8.1.1 |
| `docker-login` | [`docker/login-action`](https://github.com/docker/login-action) | v4.7.0 |
| `download-artifact` | [`actions/download-artifact`](https://github.com/actions/download-artifact) | v8.0.2 |
| `gh-release` | [`softprops/action-gh-release`](https://github.com/softprops/action-gh-release) | v3.0.3 |
| `golangci-lint` | [`golangci/golangci-lint-action`](https://github.com/golangci/golangci-lint-action) | v9.3.0 |
| `gradle-wrapper-validation` | [`gradle/actions/wrapper-validation`](https://github.com/gradle/actions) | v6.4.1 |
| `nix-installer` | [`DeterminateSystems/nix-installer-action`](https://github.com/DeterminateSystems/nix-installer-action) | main |
| `rust-cache` | [`Swatinem/rust-cache`](https://github.com/Swatinem/rust-cache) | v2.9.3 |
| `rust-toolchain` | [`dtolnay/rust-toolchain`](https://github.com/dtolnay/rust-toolchain) | master |
| `setup-go` | [`actions/setup-go`](https://github.com/actions/setup-go) | v7.0.0 |
| `setup-node` | [`actions/setup-node`](https://github.com/actions/setup-node) | v7 |
| `upload-artifact` | [`actions/upload-artifact`](https://github.com/actions/upload-artifact) | v7.0.2 |

To wrap another action, copy an existing wrapper and replace its inputs, outputs and pinned `uses:` with the upstream action's (from its `action.yml`).

## Renovate preset

`default.json` holds the settings every repository shares:

- Renovate's `config:recommended`: grouping of monorepo and commonly paired packages, crowd-sourced replacements and workarounds, changelog links and merge confidence badges
- the Dependency Dashboard
- plain PR titles ("Update foo to v1.2"), no semantic commit prefixes
- automerge of digest, minor and patch updates once checks pass. Renovate merges them itself (`platformAutomerge: false`), because no repository requires status checks and GitHub's native auto-merge could otherwise merge a PR whose checks failed. A PR with no status checks at all is never automerged
- weekly lock file maintenance, automerged
- `trexx/**` actions and workflows stay on `main` instead of being pinned

A repository that wants hand-reviewed updates adds its own `packageRules`, e.g. `"matchPackageNames": ["*"], "automerge": false` followed by rules for what may automerge.

```json
{
  "extends": ["github>trexx/actions"]
}
```

`docker.json` adds the settings shared by container image repositories: major image updates and digest pinning.

```json
{
  "extends": ["github>trexx/actions", "github>trexx/actions:docker"]
}
```

Repository-specific settings (custom managers, package rules, groups) stay in each repository's `.github/renovate.json`.
