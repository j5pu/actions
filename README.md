# actions

## pypi
Inputs:
* *password* (default: true): PYPI_API_TOKEN/TWINE_PASSWORD
* *version* (default: 3.9): python version

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
          password: ${{ secrets.PYPI_API_TOKEN }}
          version: 3.10
```

## tag
Inputs:
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

env:
  GITHUB_TOKEN: ${{ secrets.TOKEN }}

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
