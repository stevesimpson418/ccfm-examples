# GitHub Actions

Deploy Markdown docs to Confluence Cloud using the [`ccfm-convert` GitHub Action](https://github.com/stevesimpson418/ccfm-convert).

## Overview

These examples use the `stevesimpson418/ccfm-convert@v0.5.0` GitHub Action to sync documentation from a Git repository to Confluence. Each pattern demonstrates a different deployment topology while following the same init → plan-on-PR, init → apply-on-push workflow.

## Patterns

| Pattern | Directory | Description |
|---------|-----------|-------------|
| Single Environment | [`single-environment/`](single-environment/) | One docs tree, one Confluence space |
| Multi-Environment | [`multi-environment/`](multi-environment/) | One docs tree, separate staging and production spaces via GitHub Environments |
| Multi-Source | [`multi-source/`](multi-source/) | Multiple doc trees in one repo, each targeting a different space |

## Common Setup

All patterns require the following repository secrets configured in **Settings > Secrets and variables > Actions**:

| Secret | Description |
|--------|-------------|
| `CONFLUENCE_DOMAIN` | Your Confluence Cloud domain (e.g., `acme-corp.atlassian.net`) |
| `CONFLUENCE_EMAIL` | Email address of the Confluence API user |
| `CONFLUENCE_TOKEN` | Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens)) |

## GitHub Action Version

These examples pin to `v0.5.0` of the `ccfm-convert` action. Check the [releases page](https://github.com/stevesimpson418/ccfm-convert/releases) for newer versions.

## Action Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `domain` | Yes | Confluence Cloud domain |
| `email` | Yes | Confluence user email |
| `token` | Yes | Confluence API token |
| `space` | Yes | Confluence space key |
| `directory` | No | Path to the docs directory (not needed for `init`) |
| `version` | Yes | ccfm-convert version to install |
| `args` | No | CLI subcommand and flags (e.g., `init`, `plan`, `apply --auto-approve`) |

---

See the [top-level README](../README.md) for other deployment methods (standalone, Docker).
