# Publish Rust Containers

A GitHub Action for building and publishing Docker containers from Rust applications.

## Description

This action automates the process of building multi-architecture Docker containers
(amd64 and arm64) for Rust applications. It handles:

- Building Rust code for multiple architectures using `cross`
- Automatically detecting your package version using `cargo-workspaces`
- Creating and pushing Docker images with appropriate tags
- Supporting both debug and release builds

The action focuses solely on building and publishing containers, making it a
modular component in your CI/CD pipeline.

## Usage

### Prerequisites

- A Rust project with a valid Cargo.toml file
- A Dockerfile in your repository
- Docker registry credentials

### Inputs

| Name                | Description                         | Required | Default                    |
| ------------------- | ----------------------------------- | -------- | -------------------------- |
| `mode`              | Build mode: `release` or `debug`    | No       | `release`                  |
| `docker_registry`   | Docker registry hostname            | Yes      | -                          |
| `docker_username`   | Username for Docker registry        | Yes      | -                          |
| `docker_password`   | Password for Docker registry        | Yes      | -                          |
| `docker_image_name` | Name of the Docker image            | No       | `${{ github.repository }}` |
| `docker_context`    | Directory containing the Dockerfile | No       | `.`                        |
| `dockerfile`        | Path to the Dockerfile              | No       | `./Dockerfile`             |
| `git_ref`           | Git reference to build from         | No       | `${{ github.ref }}`        |

### Outputs

| Name      | Description                                    |
| --------- | ---------------------------------------------- |
| `digest`  | The Docker image digest of the built image     |
| `version` | The version of the Rust package that was built |

## Basic Usage

```yaml
name: Build and Publish Container

on:
  push:
    branches:
      - main

jobs:
  publish-container:
    runs-on: ubuntu-latest
    steps:
      - name: Build and publish Docker image
        uses: p6m-dev/actions-rust/publish-containers@v1
        with:
          docker_registry: ${{ vars.DOCKER_REGISTRY }}
          docker_username: ${{ secrets.DOCKER_USERNAME }}
          docker_password: ${{ secrets.DOCKER_PASSWORD }}
```

## Examples

### Building from a Pull Request

```yaml
name: Test Container Build

on:
  pull_request:
    branches:
      - main

jobs:
  build-container:
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker image
        uses: p6m-dev/actions-rust/publish-containers@v1
        with:
          docker_registry: ${{ vars.DOCKER_REGISTRY }}
          docker_username: ${{ secrets.DOCKER_USERNAME }}
          docker_password: ${{ secrets.DOCKER_PASSWORD }}
          git_ref: ${{ github.event.pull_request.head.sha }}
```

### Using Custom Dockerfile and Context

```yaml
name: Build Custom Container

on:
  push:
    branches:
      - main

jobs:
  build-container:
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker image
        uses: p6m-dev/actions-rust/publish-containers@v1
        with:
          docker_registry: ${{ vars.DOCKER_REGISTRY }}
          docker_username: ${{ secrets.DOCKER_USERNAME }}
          docker_password: ${{ secrets.DOCKER_PASSWORD }}
          docker_context: "./docker"
          dockerfile: "./docker/production.Dockerfile"
```

### Building Debug Mode for Testing

```yaml
name: Build Debug Container

on:
  workflow_dispatch:

jobs:
  build-debug:
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker image in debug mode
        uses: p6m-dev/actions-rust/publish-containers@v1
        with:
          docker_registry: ${{ vars.DOCKER_REGISTRY }}
          docker_username: ${{ secrets.DOCKER_USERNAME }}
          docker_password: ${{ secrets.DOCKER_PASSWORD }}
          mode: debug
```

## How It Works

This action:

1. Checks out your code at the specified Git reference
2. Installs the required tools (`cross` and `cargo-workspaces`)
3. Sets up QEMU and Docker Buildx for multi-architecture builds
4. Detects your package version using `cargo-workspaces`
5. Builds your Rust application for both x86_64 and aarch64 architectures
6. Logs in to your Docker registry
7. Builds and pushes the Docker image with appropriate tags

The generated Docker image will be tagged with:

- `latest` (if built from the default branch)
- Your package version (if built from the default branch)
- `[version]-[branch]` (if built from a non-default branch)
- The Git SHA
