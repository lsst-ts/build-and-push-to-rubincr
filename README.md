# lsst-sqre/build-and-push-to-rubincr

A composite GitHub Actions action that builds a Docker image, tags it based on the current Git branch/tag, and pushes it to rubin-cr.lsst.org.

## Usage

```yaml
name: CI

"on":
  merge_group: {}
  pull_request: {}
  push:
    tags:
      - "*"

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    # (optional) only build on tags or ticket branches
    if: >
      startsWith(github.ref, 'refs/tags/')
      || startsWith(github.head_ref, 'tickets/')

    steps:
      - uses: actions/checkout@v4

      - uses: lsst-ts/build-and-push-to-rubincr@v1
        id: build
        with:
          image: ${{ github.repository }} # e.g. lsst-sqre/safirdemo
          username: ${{ secrets.RUBINCR_USERNAME }}
          password: ${{ secrets.RUBINCR_PASSWORD }}

      - run: echo Pushed rubin-cr.lsst.org/${{ github.repository }}:${{ steps.build.outputs.tag }}
```

By default, rubin-cr.lsst.org packages are named after the GitHub repository.
To automatically set that, the above example uses the context variable `${{ github.repository }}` as the image name.

## Action reference

### Inputs

- `image` (string, required) the name of the image to build and push. The image does not include the registry (`rubin-cr.lsst.org/`) or the tag.
  For example, the image input for `rubin-cr.lsst.org/repo:tag` image is `repo`.

- `username` (string, required) the username to authenticate with for pushing to rubin-cr.lsst.org. Default is `${{ secrets.RUBINCR_USERNAME }}`.

- `password` (string, required) the password to authenticate with for pushing to rubin-cr.lsst.org. Default is `${{ secrets.RUBINCR_PASSWORD }}`.

- `dockerfile` (string, optional) the path to the Dockerfile to build. Default is `Dockerfile`.

- `context` (string, optional) the [Docker build context](https://docs.docker.com/build/building/context/). Default is `.`.

- `push` (boolean, optional) a flag to enable pushing to rubin-cr.lsst.org. Default is `true`.
  If `false`, the action skips the push to rubin-cr.lsst.org, but still builds the image with [`docker build`](https://docs.docker.com/engine/reference/commandline/build/).

- `cache-from` (string, optional) a comma-separated list of Docker buildx cache sources.
  Default is `type=gha` to use the GitHub Actions cache.
  Set as `type=local,src=/tmp/.buildx-cache` to use a local cache.

- `cache-to` (string, optional) a comma-separated list of Docker buildx cache destinations.
  Default is `type=gha,mode=max` to use the GitHub Actions cache.
  Set as `type=local,dest=/tmp/.buildx-cache` to use a local cache.

### Outputs

- `fully_qualified_image_digest` (string) A complete, unique, and immutable identifier for the built image,
  e.g. `rubin-cr.lsst.org/repo@sha256:4dcaf15076e027f272dc8aba14b1bab77fec44f8aac94c94f1b01ceee8d099d4`.
  This string may be used to reference the built image in `docker pull`, `docker run`, etc.
- `tag` (string) the tag of the image that was pushed to rubin-cr.lsst.org.

## Developer guide

This repository provides a **composite** GitHub Action, a type of action that packages multiple regular actions into a single step.
We do this to make the GitHub Actions of all our software projects more consistent and easier to maintain.
[You can learn more about composite actions in the GitHub documentation.](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

Create new releases using the GitHub Releases UI and assign a tag with a [semantic version](https://semver.org), including a `v` prefix. Choose the semantic version based on compatibility for users of this workflow. If backwards compatibility is broken, bump the major version.

When a release is made, a new major version tag (i.e. `v1`, `v2`) is also made or moved using [nowactions/update-majorver](https://github.com/marketplace/actions/update-major-version).
We generally expect that most users will track these major version tags.
