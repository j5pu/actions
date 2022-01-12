# actions

![main](https://github.com/j5pu/actions/actions/workflows/pypi.yaml/badge.svg)


## [id](id)

GitHub ID.

Inputs:
* *user* (default: `github.actor`)

### Examples:

```yaml
name: id

on:
  workflow_dispatch:

jobs:
  id:
    runs-on: ubuntu-latest
    steps:
      - id: ID
        uses: j5pu/actions/id@main
      - run: echo ${{ steps.ID.outputs.ID }}

```

## [pypi](pypi)

Build and publish 📦 to PyPI.

Inputs:
* *keep* (default: 3): releases to keep 
* *pypi_password*: password to remove releases (PYPI_CLEANUP_PASSWORD)
* *pypi_token*: token to upload releases (PYPI_API_TOKEN)
* *pypi_user* (default: `github.actor`): python version
* *version* (default: 3.9): python version

Uses: [pypirm](pypirm)

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
      - '**'
  
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

## [pypirm](pypirm)

Delete versions 📦 from PyPI.

Inputs:
* *keep* (default: 3): releases to keep 
* *pypi_password*: password to remove releases (PYPI_CLEANUP_PASSWORD)
* *pypi_user* (default: `github.actor`): python version

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

## [tag](tag)

Tag and push repository if 'svu current' != 'svu next'.

Inputs:
* *keep* (default: 3): releases to keep 
* *release* (default: true): creates a release and delete older releases 
* *repository* (default: `github.repository`): repository owner/name
* *strip* (default: true): strip prefix

Repository will be checkout with fetch-depth: 0 and te following will be done if `svu next` != `svu current`:
* *tag* with `svu next` (based on strip option),
* *push tag*,
* create a *release* (unless, release: false)
* and *delete older releases*  (unless, release: false).

### Examples:

Use te following to not trigger on tags (only pushing commits):

```yaml
on:
  push:
    branches:
      - '**'
```

### Another action to be triggered after tag
An action in a workflow run 
[can’t trigger a new workflow](https://github.community/t/github-actions-workflow-not-triggering-with-tag-push/17053/2).

```bash
secrets.sh
```

```yaml
name: tag

on:
  push:
    branches:
      - '**'
  workflow_dispatch:

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

on:
  push:
    branches:
      - '**'
  workflow_dispatch:

jobs:
  tag:
    runs-on: ubuntu-latest
    steps:
      - uses: j5pu/actions/tag@main
```

#### With release: false, separate workflow for release:
```yaml
name: tag

on:
  push:
    branches:
      - '**'
  workflow_dispatch:

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
      - '**'
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

## [tap](tap)

Brew Tap Release.

A `LICENSE` file is required.

Inputs:
* *commit_email* (default: 
`${{ github.event.schedule.user.id }}+${{ github.workflow }}@users.noreply.github.com`) 
* *commit_owner* (default: `github.workflow`)
* *debug* (default: false)
* *depends_on*
* *formula_folder* (default: Formula) 
* *github_token* (default: `env.GITHUB_TOKEN`) 
* *homebrew_owner* (default: `github.actor`) 
* *homebrew_tap* (default: dev)
* *install* (default: `bin.install "#{name}"`)
* *skip_commit* (default: false)
* *test* (default: `system "test", "-x", "#{name}"`)
* *update_readme_table* (default: true)

The following will be done on the tap repository:
* *Formula: create*: create empty formula file.
* *Formula: release*: release formula file with inputs using
[Justintime50/homebrew-releaser](https://github.com/Justintime50/homebrew-releaser).
* *Formula: workflow*: creates a workflow for testing.
* *Formula: tag*: tags the formula repository to trigger the testing workflow, using [tag](tag).

### Examples:

#### Separate workflow for tag and tap
```bash
secrets.sh
```

```yaml
name: tag

on:
  push:
    branches:
      - '**'
  workflow_dispatch:

env:
  GITHUB_TOKEN: ${{ secrets.TOKEN }}

jobs:
  tag:
    runs-on: ubuntu-latest
    steps:
      - id: tag
        uses: j5pu/actions/tag@main
```

```yaml
name: tap

on:
  push:
    tags:
      - '**'

jobs:
  tap:
    runs-on: ubuntu-latest
    name: tap
    steps:
      - name: "Release to Homebrew tap"
        uses: j5pu/actions/tap@main
        with:
          github_token: ${{ secrets.TOKEN }}
          depends_on: |
            "bats-core"
            "bats-core/bats-core/bats-assert"
            "bats-core/bats-core/bats-file"
            "bats-core/bats-core/bats-support"
          install: 'bin.install "#{name}.bash"'
          test: |
            system "sh", "-c", "#{HOMEBREW_PREFIX}/bin/bats.bash && command -v assert_file_exist"
```

#### Single workflow
```bash
secrets.sh
```

Assumes executable with the same name of repository `${{ github.event.repository.name }}`.

```yaml
name: main

on:
  push:
    branches:
      - '**'
  workflow_dispatch:

env:
  GITHUB_TOKEN: ${{ secrets.TOKEN }}

jobs:
  main:
    runs-on: ubuntu-latest
    name: "Tag & Tap"
    steps:
      - name: "Tag"
        id: TAG
        uses: j5pu/actions/tag@main
        
      - name: "Tap"
        uses: j5pu/actions/tap@main
        if: steps.TAG.outputs.CHANGED == 'true'
```
