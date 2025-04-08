# Bump Version and Create Git Tag

[parent](../README.md)

A GitHub Action that automates version bumping and Git tag creation for Rust
projects using semantic versioning.

## Description

This action is designed to streamline release management for Rust projects using
Cargo workspaces. It leverages `cargo-workspaces` to bump versions across
multiple packages in a synchronized manner, create appropriate Git tags, and
make a version commit.

## Features

- Semantic versioning support (major, minor, patch)
- Branch protection (only bump versions on specified branches)
- Customizable commit messages
- Configurable Git user identity
- Support for Cargo workspaces with options to control tagging behavior
- Outputs the new version and tag for use in subsequent workflow steps

## Usage

Add this action to your GitHub workflow file:

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Bump version and create tag
        uses: p6m-dev/actions-rust/cut-tag@v1
        id: bump
        with:
          bump-level: patch # Can be major, minor, or patch

      - name: Use the new version
        run: echo "New version is ${{ steps.bump.outputs.version }}"
```

## Inputs

| Input                | Description                                                   | Required | Default                                   |
| -------------------- | ------------------------------------------------------------- | -------- | ----------------------------------------- |
| `bump-level`         | Version bump level (major, minor, patch)                      | Yes      | `patch`                                   |
| `allow-branch`       | Branch to allow version bumping on                            | No       | `main`                                    |
| `commit-message`     | Commit message template for version bump (use %v for version) | No       | `Release %v [skip ci]`                    |
| `commit-user-name`   | Git user name for the commit                                  | No       | `github-actions`                          |
| `commit-user-email`  | Git user email for the commit                                 | No       | `github-actions@users.noreply.github.com` |
| `no-individual-tags` | Do not create individual tags for each workspace              | No       | `true`                                    |
| `force-all`          | Force version for all packages                                | No       | `true`                                    |

## Outputs

| Output    | Description                                      |
| --------- | ------------------------------------------------ |
| `version` | The new version that was created (e.g., `1.2.3`) |
| `tag`     | The Git tag that was created (e.g., `v1.2.3`)    |

## Examples

### Basic Usage

```yaml
- name: Bump patch version
  uses: p6m-dev/actions-rust/cut-tag@v1
  with:
    bump-level: patch
```

### Minor Version Bump with Custom Commit Message

```yaml
- name: Bump minor version
  uses: p6m-dev/actions-rust/cut-tag@v1
  with:
    bump-level: minor
    commit-message: "chore: bump version to %v"
```

### Custom Git User and Branch Control

```yaml
- name: Bump version with custom user
  uses: p6m-dev/actions-rust/cut-tag@v1
  with:
    bump-level: patch
    allow-branch: release
    commit-user-name: "Release Bot"
    commit-user-email: "release-bot@example.com"
```

### Workspace Configuration

```yaml
- name: Bump version with individual workspace tags
  uses: p6m-dev/actions-rust/cut-tag@v1
  with:
    bump-level: patch
    no-individual-tags: "false"
    force-all: "false"
```

### Cutting Tags based on Workflow Dispatch

```yaml
name: Create Tag

on:
  workflow_dispatch:
    inputs:
      level:
        description: "Version Bump Level"
        type: choice
        required: true
        default: patch
        options:
          - major
          - minor
          - patch
      branch:
        description: "Branch to create tag on"
        type: string
        required: false
        default: main

permissions:
  contents: write

jobs:
  create-tag:
    name: Create Version Tag
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          ref: ${{ github.event.inputs.branch }}

      - name: Setup Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          profile: minimal

      - name: Bump Version and Create Tag
        id: cut-tag
        uses: p6m-dev/actions-rust/cut-tag@v1
        with:
          bump-level: ${{ github.event.inputs.level }}
          allow-branch: ${{ github.event.inputs.branch }}
          commit-message: "chore(release): bump version to %v [skip ci]"

      - name: Push Changes and Tags
        run: |
          git push --follow-tags

      - name: Display New Version
        run: echo "Successfully created tag v${{ steps.bump-version.outputs.version }}"
```

## Requirements

- Your project must use Cargo workspaces
- Git must be configured with appropriate permissions to push to the repository

## How It Works

1. Installs the `cargo-workspaces` tool
2. Configures Git with the specified user information
3. Runs `cargo workspaces version` with the specified bump level and options
4. Captures the new version for output to subsequent workflow steps
