# actions

[![Build Status](https://github.com/j5pu/actions/workflows/main/badge.svg)](https://github.com/j5pu/actions/actions/workflows/main.yaml)

## [description](description)

GitHub repository description.

Inputs:
* *repository* (default: `github.repository`)

### Examples:

```yaml
name: id

on:
  workflow_dispatch:

jobs:
  id:
    runs-on: ubuntu-latest
    steps:
      - id: DESCRIPRION
        uses: j5pu/actions/id@main
      - run: echo ${{ steps.DESCRIPRION.outputs.DESCRIPRION }}
```

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
[new tag does not trigger an event](https://github.community/t/new-tag-does-not-trigger-an-event/121511)

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
* *version* (default: "")

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
        with:
          version: ${{ steps.TAG.outputs.NEXT }}
```

## Example: matrix

```yaml
jobs:
  main:
    runs-on: ${{ matrix.os}}
    strategy: 
      matrix:
        os: [macos-latest, ubuntu-latest]
        node: [3.9, 3.10]
    steps:
      - uses: actions/setup-node@v1
        with:
          python-version: ${{ matrix.python }}
```

## Composite and Local Execution Attempt
```shell
#!/usr/bin/env bash

# https://docs.github.com/en/actions/learn-github-actions/contexts

set -eu
shopt -s inherit_errexit

# actions repository path ${{github.action_path}}
export ACTION_PATH
export COMMIT_EMAIL
export COMMIT_OWNER
export DEBUG
export DEPENDS_ON
export FORMULA_FOLDER
export GITHUB_TOKEN
export HOMEBREW_OWNER
export HOMEBREW_TAP
export INSTALL
export SKIP_COMMIT
export TEST
export UPDATE_README_TABLE

#######################################
# git checkout when running locally
# Arguments:
#  None
#######################################
checkout() {
  ! $ACTION || return 0
}

#######################################
# updates git config
# Globals:
#   ACTION
#   COMMIT_EMAIL
#   COMMIT_OWNER
# Arguments:
#  None
# Returns:
#   <unknown> ...
#######################################
config() {
  $ACTION || return 0
  git config --global user.email "${COMMIT_EMAIL}"
  git config --global user.name "${COMMIT_OWNER}"
}

#######################################
# adds variable/s to environment $GITHUB_ENV
# Globals:
#   GITHUB_ENV
# Arguments:
#   1
#######################################
envfile() {
  $ACTION || return 0
  for arg; do
    echo "${arg}=${!arg}" >> "${GITHUB_ENV}"
  done
}

#######################################
# adds argument to $PATH or $GITHUB_PATH
# Arguments:
#  None
#######################################
pathfile() {
  local directory
  [ ! "${1-}" ]
  ! command -v "${0##*/}" || return 0

  directory="$(dirname "$0")"
  if $ACTION; then
    echo "${directory}" >> "${GITHUB_PATH}"
  else
    PATH="${directory}:${PATH}"
  fi
}

#######################################
# work done on the source repository
# Arguments:
#  None
# GitHub Context:
#   github.action             name of the action currently running, or the id of a step
#   github.action_path        path where an action is located. This property is only supported in composite actions.
#                             You can use this path to access files located in the same repository as the action:
#                             i.e: action.yml
#   github.action_repository  for a step executing an action, this is the owner and repository name of the action.
#                             For example, actions/checkout
#   github.actor              username of the user that initiated the workflow run
#   github.api_url            URL of the GitHub REST API.
#   github.event
#                             if: github.event.base_ref=='refs/heads/master'
#   github.event_name         the name of the event that triggered the workflow run.
#                             if: github.event_name == 'release' && github.event.action == 'created'
#                             if: github.event_name == 'push' 
#   github.path               the path on the runner to the file to set system $PATH variables: echo /bin >>$GITHUB_PATH
#   github.ref_name           branch or tag name that triggered the workflow run
#   github.ref_type           branch or tag:
#                             if: ${{ github.ref_type == 'tag' && github.event.base_ref=='refs/heads/master' }}
#                             if: startsWith(github.ref, 'refs/tags/v')
#                             if: github.event_name == 'push' 
#
#   github.repository         owner/name
#   github.repository_owner   owner
#   github.repositoryUrl      git://github.com/owner/name.git
#   github.server_url         https://github.com
#   github.workflow           name of the workflow or full path of the workflow file in the repository.
#   github.workspace          default working directory on the runner for steps, and the default location of repository
#                             when using the checkout action $GITHUB_WORKSPACE (git top)
#######################################
repo() {
  $ACTION || cd "$(git rev-parse --show-toplevel)"
}

#######################################
# main function for GitHub Action tap
# Globals:
#   TOKEN
# Arguments:
#  None
#######################################
main() {
  local token

  ACTION=true
  [ "${GITHUB_RUN_ID-}" ] || ACTION=false

  while (( $# )); do
    case "$1" in
      --action_path=*) ACTION_PATH="${1#--action_path=}" ;;
      --commit_email=*) COMMIT_EMAIL="${1#--commit_email=}" ;;
      --commit_owner=*) COMMIT_OWNER="${1#--commit_owner=}" ;;
      --debug=*) DEBUG="${1#--debug=}" ;;
      --depends_on=*) DEPENDS_ON="${1#--depends_on=}" ;;
      --formula_folder=*) FORMULA_FOLDER="${1#--formula_folder=}" ;;
      --github_token=*) token="${1#--token=}"; GITHUB_TOKEN="${1:-${token}}" ;;
      --homebrew_owner=*) HOMEBREW_OWNER="${1#--homebrew_owner=}" ;;
      --homebrew_tap=*) HOMEBREW_TAP="${1#--homebrew_owner=}" ;;
      --install=*) INSTALL="${1#--install=}" ;;
      --skip_commit=*) SKIP_COMMIT="${1#--skip_commit=}" ;;
      --test=*) TEST="${1#--test=}" ;;
      --update_readme_table=*) UPDATE_README_TABLE="${1#--update_readme_table=}" ;;
    esac
    shift
  done

  [ "${GITHUB_TOKEN-}" ] || { >&2 echo GITHUB_ACTION: Empty; exit 1; }

  : "${COMMIT_EMAIL=root@example.com}"

  if $ACTION; then
    : "${COMMIT_OWNER=j5pu}"  # ${{ github.actor }}
  else
    : "${COMMIT_OWNER="${USER}"}"
  fi

  : "${DEBUG=false}"
  : "${DEPENDS_ON=false}"
  : "${FORMULA_FOLDER=Formula}"
  : "${HOMEBREW_OWNER=${COMMIT_OWNER}}"
  : "${HOMEBREW_TAP=dev}"
  : "${INSTALL=bin.install name.to_s}"
  : "${SKIP_COMMIT=false}"
  : "${TEST="system \"command\", \"-v\", name.to_s"}"
  : "${UPDATE_README_TABLE=true}"

  envfile COMMIT_EMAIL GITHUB_TOKEN

  for i in COMMIT_EMAIL COMMIT_OWNER DEBUG DEPENDS_ON FORMULA_FOLDER HOMEBREW_OWNER HOMEBREW_TAP INSTALL \
    SKIP_COMMIT TEST UPDATE_README_TABLE; do
    echo "${i}: ${!i}"
  done
}

main "$@"
```
