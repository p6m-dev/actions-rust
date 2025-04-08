# Cargo Registry Login

A GitHub Action for configuring Cargo registry authentication using environment variables.

## Overview

This action configures Cargo registry authentication by setting the appropriate environment variables based on provided registry tokens. It supports authentication for crates.io and any custom registries.

## Usage

Add this action to your workflow to authenticate with Cargo registries:

```yaml
- name: Login to Cargo registries
  uses: p6m-dev/actions/rust/cargo-registry-login@v1
  with:
    cargo-tokens: |
      crates-io=your-crates-io-token
      my-private-registry=your-private-registry-token
```

## Inputs

| Name           | Description                                        | Required |
| -------------- | -------------------------------------------------- | -------- |
| `cargo-tokens` | Registry tokens in key=value format (one per line) | No       |

### Format for `cargo-tokens`

The input follows this format:

```
registry-name=token
```

Where:

- `registry-name`: The name of the registry (use `crates-io` for crates.io)
- `token`: The authentication token for the registry

Multiple registries can be configured by adding one registry per line.

## How it works

For each registry token provided:

1. For crates.io, it sets:

   ```
   CARGO_REGISTRY_CREDENTIAL_PROVIDER=cargo:token
   CARGO_REGISTRY_TOKEN=Bearer <token>
   ```

2. For other registries, it sets:

   ```
   CARGO_REGISTRIES_<REGISTRY>_CREDENTIAL_PROVIDER=cargo:token
   CARGO_REGISTRIES_<REGISTRY>_TOKEN=Bearer <token>
   ```

   Note: Registry names are transformed to uppercase with hyphens converted to underscores.

## Example

### Basic authentication with crates.io

```yaml
- name: Login to crates.io
  uses: your-org/cargo-registry-login@v1
  with:
    cargo-tokens: crates-io=your-crates-io-token
```

### Authentication with multiple registries

```yaml
- name: Login to Cargo registries
  uses: your-org/cargo-registry-login@v1
  with:
    cargo-tokens: |
      crates-io=your-crates-io-token
      github-registry=your-github-token
      custom-registry=your-custom-token
```

## Secrets management

It's recommended to store tokens as GitHub secrets:

```yaml
- name: Login to Cargo registries
  uses: your-org/cargo-registry-login@v1
  with:
    cargo-tokens: |
      crates-io=${{ secrets.CRATES_IO_TOKEN }}
      github-registry=${{ secrets.GITHUB_REGISTRY_TOKEN }}
```
