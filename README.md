# actions
GitHub Action Workflows for Org

## Workflows

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
