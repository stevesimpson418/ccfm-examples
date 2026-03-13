# CCFM Examples

Example projects demonstrating real-world usage of
[ccfm-convert](https://github.com/stevesimpson418/ccfm-convert) — a CLI tool that
converts Markdown to Atlassian Document Format (ADF) and syncs pages to Confluence Cloud.

## Choose your deployment method

| I want to run from... | Example | What's inside |
| --- | --- | --- |
| GitHub Actions | [`github-actions/`](github-actions/) | Workflows using the `ccfm-convert` GitHub Action |
| GitLab CI | [`gitlab-ci/`](gitlab-ci/) | Pipelines using the `ccfm-convert` Docker image natively |
| Docker (any CI or local) | [`docker/`](docker/) | Docker Compose configs — works anywhere Docker runs |
| pip install (any environment) | [`standalone/`](standalone/) | Makefiles wrapping the `ccfm` CLI directly |

## Choose your pattern

Each deployment method includes three patterns:

| Pattern | When to use it | What changes |
| --- | --- | --- |
| **single-environment** | One docs tree, one Confluence space. Most common. | One `ccfm.yaml`, one apply target |
| **multi-environment** | Staging review before production. | Same `ccfm.yaml`, space key varies per environment |
| **multi-source** | Multiple doc trees in one repo targeting different spaces. | Multiple `ccfm.yaml` files, one per source |

## Quick start

The fastest way to try ccfm-convert — no CI or Docker needed:

```bash
git clone https://github.com/stevesimpson418/ccfm-examples.git
cd ccfm-examples/standalone/single-environment

# Copy the example docs into place
cp -r ../../_shared/docs ./docs

# Set your Confluence credentials
export CONFLUENCE_DOMAIN=your-org.atlassian.net
export CONFLUENCE_EMAIL=you@example.com
export CONFLUENCE_TOKEN=your-api-token

# Install and run
make install
make plan      # preview what would change (safe, read-only)
make apply     # apply changes to Confluence
```

## Shared docs

All examples reference the same documentation content in [`_shared/docs/`](_shared/docs/).
Copy this directory into your chosen example as `docs/` before running.

For multi-source examples, a second smaller doc set is available at
[`_shared/docs-wiki/`](_shared/docs-wiki/).

## Installation

```bash
pip install ccfm-convert
ccfm --help
```

Or with Docker:

```bash
docker pull ghcr.io/stevesimpson418/ccfm-convert:latest
docker run ghcr.io/stevesimpson418/ccfm-convert --help
```

## Quick reference

```bash
# Initialize management pages
ccfm --config ccfm.yaml init

# Preview what would change (no modifications made)
ccfm --config ccfm.yaml plan --directory docs

# Apply changes to Confluence
ccfm --config ccfm.yaml apply --directory docs --auto-approve

# Force re-apply all pages regardless of state
ccfm --config ccfm.yaml apply --directory docs --auto-approve --force
```

See [ccfm.io](https://ccfm.io) for full documentation.
