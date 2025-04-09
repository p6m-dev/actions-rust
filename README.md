# Rust Github Actions

## General Usage Guidelines

- These actions are designed to work in conjunction with each other compatibly
  when they all use the same version number.
- The main branch is for development purposes only, and should not be used
  directly in production workflows
- The major (@v1, @v2) tags will always point to the latest minor release (@v1.1
  @v2.3) tags. This means that a workflow can stay on the major tag to get all
  backwards-compatible changes.
- These actions are designed to be re-used and composed in larger workflows,
  and most workflows will only use a subset of these.
- Each action has it's own README.md with instructions and examples for how to
  use it.

## Actions

Each of these actions represent a specific task in a typical SDLC. Based on
requirements of a give project, some actions may be useful while others are not.
As an example, not every project produces GraphQL Subgraphs, or publishes
artifacts to an Artifact Management System.

### [Cut Tag (cut-tag)](cut-tag/README.md)

Bumps version numbers and Cuts Tags

### [Cargo Repository Login (cargo-repository-login)](cargo-repository-login/README.md)

Logs into a Cargo Repository to allow publishing
