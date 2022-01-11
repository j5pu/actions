# actions

## pypi

Build and publish 📦 to PyPI.

Inputs:
* *keep* (default: 3): releases to keep 
* *pypi_password*: password to remove releases (PYPI_CLEANUP_PASSWORD)
* *pypi_token*: token to upload releases (PYPI_API_TOKEN)
* *pypi_user* (default: github.actor): python version
* *version* (default: 3.9): python version

Uses: *pypirm*

### Examples:

#### With version: 3.10
```shell
secrets.sh
```

```yaml
name: publish

on:
  push:
    tags:
      - '*'
    
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: j5pu/actions/pypi@main
        with:
          pypi_password: ${{ secrets.PYPI_CLEANUP_PASSWORD }}
          pypi_token: ${{ secrets.PYPI_API_TOKEN }}
          version: 3.10
```

## pypirm

Delete versions 📦 from PyPI.

Inputs:
* *keep* (default: 3): releases to keep 
* *pypi_password*: password to remove releases (PYPI_CLEANUP_PASSWORD)
* *pypi_user* (default: github.actor): python version

### Examples:

#### With version: 3.10
```shell
secrets.sh
```

```yaml
name: remove

on:
  push:
    tags:
      - '*'
  workflow_dispatch:
    
jobs:
  remove:
    runs-on: ubuntu-latest
    steps:
      - uses: j5pu/actions/pypirm@main
        with:
          pypi_password: ${{ secrets.PYPI_CLEANUP_PASSWORD }}
```

## tag

Tag and push repository if 'svu current' != 'svu next'.

Inputs:
* *keep* (default: 3): releases to keep 
* *release* (default: true): creates a release and delete older releases 
* *strip* (default: true): strip prefix

Repository will be checkout with fetch-depth: 0 and te following will be done if `svu next` != `svu current`:
* *tag* with `svu next` (based on strip option),
* *push tag*,
* create a *release* (unless, release: false)
* and *delete older releases*  (unless, release: false).

### Examples:

### Another action to be triggered after tag
An action in a workflow run 
[can’t trigger a new workflow](https://github.community/t/github-actions-workflow-not-triggering-with-tag-push/17053/2).

```bash
secrets.sh
```

```yaml
name: tag

on: [push, workflow_dispatch]

env:
  GITHUB_TOKEN: ${{ secrets.TOKEN }}

jobs:
  tag:
    runs-on: ubuntu-latest
    steps:
      - uses: j5pu/actions/tag@main
```

#### With release: true
```yaml
name: tag

on: [push, workflow_dispatch]

jobs:
  tag:
    runs-on: ubuntu-latest
    steps:
      - uses: j5pu/actions/tag@main
```

#### With release: false, separate workflow for release:
```yaml
name: tag

on: [push, workflow_dispatch]

jobs:
  tag:
    runs-on: ubuntu-latest
    steps:
      - uses: j5pu/actions/tag@main
        with:
          release: false
```

```yaml
name: publish

on:
  push:
    tags:
      - '*'
  workflow_dispatch:

jobs:
  publish:
    runs-on: macOS-latest
    steps:
      - uses: actions/checkout@v2
      - uses: softprops/action-gh-release@v1
      - uses: garden-io/update-homebrew-action@v1
```

#### Outputs
```yaml
name: tag

on: [push, workflow_dispatch]

jobs:
  tag:
    runs-on: ubuntu-latest
    steps:
      - id: tag
        uses: j5pu/actions/tag@main
      - run: |
          echo ${{ steps.tag.outputs.changed }} 
          echo ${{ steps.tag.outputs.current }} 
          echo ${{ steps.tag.outputs.next }} 
          echo ${{ steps.tag.outputs.prefix }}
        shell: bash
```
