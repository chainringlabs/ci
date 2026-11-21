# ci

Shared reusable GitHub Actions workflows (`workflow_call`) for chainringlabs
repositories. Each repository keeps a short caller file; the logic lives here.

This repository is public because public repositories cannot call a reusable
workflow that lives in a private one.

## Workflows

| Workflow | What it does | Permissions the calling job must grant |
|---|---|---|
| `ghcr-retention.yml` | Prunes the calling repository's GHCR package: keeps `latest` and the 5 most recent tagged versions, deletes untagged manifests. | `packages: write` |
| `docker-publish.yml` | Builds the calling repository's image (`linux/amd64`). On pull requests it only builds. Otherwise it pushes to `ghcr.io/<owner>/<repo>` (`latest` on the default branch, semver on `v*` tags, `sha-<short sha>`), signs with cosign keyless, and attaches an SPDX SBOM and build provenance. | `contents: read`, `packages: write`, `id-token: write`, `attestations: write`, `artifact-metadata: write` |

## Calling a workflow

Pin to a full commit SHA (the organization requires it) with the release tag
in a trailing comment, so Renovate can raise the pin:

```yaml
name: GHCR retention

on:
  workflow_dispatch:
  schedule:
    - cron: '0 6 * * 1' # Mondays 06:00 UTC

permissions: {}

jobs:
  prune:
    permissions:
      packages: write
    uses: chainringlabs/ci/.github/workflows/ghcr-retention.yml@<commit-sha> # v1.0.0
```

The check a caller reports is named `<caller job> / <reusable job>`, for
example `prune / Prune old GHCR versions`.

## Releasing

Merge to `main`, then tag the merge commit `vX.Y.Z`. Callers pin the tag's
commit SHA. A change that alters inputs, permissions or check names is a new
major version.
