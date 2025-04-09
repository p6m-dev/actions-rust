# Publish Crates

[parent](../README.md)

A GitHub Action to publish crates from a Rust workspace using [cargo-workspaces](https://github.com/pksunkara/cargo-workspaces).

This action handles the publishing process without performing version bumping,
git tagging, or cargo login. It's designed to be used as part of a broader CI/CD
pipeline where those operations are handled separately.

## Description

This action uses the `cargo-workspaces` tool to simplify the publishing of Rust
crates from a workspace. It provides a clean, configurable interface that allows
you to:

- Publish all or specific crates in your workspace
- Skip specified crates
- Run in dry-run mode
- Configure various publishing options
- Publish without creating git tags or commits

The action intentionally avoids version bumping and git operations to allow for
more flexible CI/CD pipelines where these steps can be managed separately.

## Usage

```yaml
- name: Publish Rust crates
  uses: p6m-dev/actions-rust/publish-crates@v1
  with:
    dry_run: "false"
    # Additional parameters as needed
```

## Inputs

| Name                | Description                                                   | Required | Default       |
| ------------------- | ------------------------------------------------------------- | -------- | ------------- |
| `dry_run`           | Perform a dry run without actually publishing                 | false    | `'false'`     |
| `crates_to_publish` | Optional specific crates to publish (comma-separated)         | false    |               |
| `ignore_crates`     | Optional crates to ignore during publishing (comma-separated) | false    |               |
| `registry`          | Registry to publish to                                        | false    | `'crates.io'` |
| `no_verify`         | Skip verification step                                        | false    | `'false'`     |
| `allow_dirty`       | Allow dirty working directory                                 | false    | `'false'`     |
| `from_git`          | Use current git tag as version                                | false    | `'false'`     |
| `exact`             | Specify exact version dependencies between workspace crates   | false    | `'true'`      |
| `all_features`      | Activate all available features                               | false    | `'false'`     |

## Outputs

This action does not have any outputs.

## Basic Usage

### Step 1: Set up your action directory

Create a file at `.github/actions/publish-rust-crates/action.yml` with the contents
of the action.

### Step 2: Set up credentials

Before using this action, you'll need to configure your cargo credentials. Here's
an example of how to do this in your workflow:

```yaml
- name: Configure cargo credentials
  run: |
    mkdir -p ~/.cargo
    echo "[registry]" > ~/.cargo/credentials
    echo "token = \"${{ secrets.CARGO_REGISTRY_TOKEN }}\"" >> ~/.cargo/credentials
```

### Step 3: Use the action in your workflow

```yaml
name: Publish Crates

on:
  workflow_dispatch:

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Configure cargo credentials
        run: |
          mkdir -p ~/.cargo
          echo "[registry]" > ~/.cargo/credentials
          echo "token = \"${{ secrets.CARGO_REGISTRY_TOKEN }}\"" >> ~/.cargo/credentials

      - name: Publish Rust crates
        uses: p6m-dev/actions-rust/publish-crates@v1
```

## Examples

### Publishing Specific Crates

```yaml
- name: Publish specific crates
  uses: p6m-dev/actions-rust/publish-crates@v1
  with:
    crates_to_publish: "crate1,crate2"
```

### Dry Run

```yaml
- name: Test publishing process
  uses: p6m-dev/actions-rust/publish-crates@v1
  with:
    dry_run: "true"
```

### Publishing to a Different Registry

```yaml
- name: Publish to custom registry
  uses: p6m-dev/actions-rust/publish-crates@v1
  with:
    registry: "my-custom-registry"
```

### Complex Example with Multiple Options

```yaml
- name: Publish with advanced options
  uses: p6m-dev/actions-rust/publish-crates@v1
  with:
    crates_to_publish: "crate1,crate2"
    ignore_crates: "internal-crate"
    allow_dirty: "true"
    exact: "true"
    all_features: "true"
```

## Notes

- Make sure to set up your cargo credentials before using this action
- This action does not handle version bumping or git operations
- To use this action, you need to have a valid Rust workspace set up
