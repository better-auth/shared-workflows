# Better Auth shared workflows

Reusable GitHub Actions workflows for Better Auth repositories.

## `.github/actions/setup-pnpm`

Sets up Node.js and pnpm, then installs dependencies with a frozen lockfile. The pnpm store cache remains disabled.

Requires pnpm 11 or newer in `packageManager`, a numeric Node.js version in `.nvmrc`, and a committed `pnpm-lock.yaml`.

```yaml
- uses: actions/checkout@<commit-sha>
  with:
    persist-credentials: false

- uses: better-auth/shared-workflows/.github/actions/setup-pnpm@<commit-sha>
```

Set `node-version` to override `.nvmrc`, such as in a matrix job:

```yaml
- uses: better-auth/shared-workflows/.github/actions/setup-pnpm@<commit-sha>
  with:
    node-version: ${{ matrix.node-version }}
```

## `release-bumpp-library.yml`

Publishes packages from a pnpm project when a bumpp release pull request is merged.

The calling repository must export its release branch and npm tag from `bump.config.ts`:

```ts
export const releaseConfig = {
  branch: "main", // Release PR target
  npmTag: "latest", // npm dist-tag for stable releases
} as const;
```

It must also use pnpm, provide a `build` script and `.nvmrc`, and configure npm trusted publishing in a `release` environment.

For a maintenance branch, use the branch name and its npm tag instead, such as `v1.3.x` and `release-1.3`.

Call the workflow from the package repository and pin it to a full commit SHA:

```yaml
name: Release

on:
  pull_request:
    types: [closed]

permissions: {}

jobs:
  release:
    permissions:
      contents: write
      id-token: write
    uses: better-auth/shared-workflows/.github/workflows/release-bumpp-library.yml@<commit-sha>
    with:
      version-file: packages/example-library/package.json
```

## `lint-github-actions.yml`

Validates GitHub Actions workflows with actionlint and ShellCheck.

```yaml
name: Lint GitHub Actions

on:
  pull_request:
  push:
    branches: [main]

permissions: {}

jobs:
  lint:
    permissions:
      contents: read
    uses: better-auth/shared-workflows/.github/workflows/lint-github-actions.yml@<commit-sha>
```

## `zizmor.yml`

Runs zizmor in read-only blocking mode without GitHub Code Scanning.

```yaml
name: Zizmor

on:
  pull_request:

permissions: {}

jobs:
  zizmor:
    permissions:
      actions: read
      contents: read
    uses: better-auth/shared-workflows/.github/workflows/zizmor.yml@<commit-sha>
```

## `zizmor-code-scanning.yml`

Uploads zizmor findings to GitHub Code Scanning. Repository rulesets determine whether findings block merges.

```yaml
name: Zizmor Code Scanning

on:
  pull_request:
  push:
    branches: [main]

permissions: {}

jobs:
  zizmor:
    if: github.event_name == 'push' || !github.event.pull_request.head.repo.fork
    permissions:
      actions: read
      contents: read
      security-events: write
    uses: better-auth/shared-workflows/.github/workflows/zizmor-code-scanning.yml@<commit-sha>
```
