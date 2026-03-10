# CCFM Examples

Example projects demonstrating real-world usage of
[ccfm-convert](https://github.com/stevesimpson418/ccfm-convert) — a CLI tool that
converts Markdown to Atlassian Document Format (ADF) and deploys pages to Confluence Cloud.

## Choose your deployment method

| I want to deploy from... | Example | What's inside |
| --- | --- | --- |
| GitHub Actions | [`github-actions/`](github-actions/) | Workflows using the `ccfm-convert` GitHub Action |
| GitLab CI | [`gitlab-ci/`](gitlab-ci/) | Pipelines using the `ccfm-convert` Docker image natively |
| Docker (any CI or local) | [`docker/`](docker/) | Docker Compose configs — works anywhere Docker runs |
| pip install (any environment) | [`standalone/`](standalone/) | Makefiles wrapping the `ccfm` CLI directly |

## Choose your pattern

Each deployment method includes three patterns:

| Pattern | When to use it | What changes |
| --- | --- | --- |
| **single-environment** | One docs tree, one Confluence space. Most common. | One `ccfm.yaml`, one deploy target |
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
make plan      # preview what would deploy (safe, read-only)
make deploy    # deploy for real
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
# Preview what would deploy (no changes made)
ccfm --config ccfm.yaml deploy --directory docs --plan

# Deploy all pages
ccfm --config ccfm.yaml deploy --directory docs

# Only deploy changed files
ccfm --config ccfm.yaml deploy --directory docs --changed-only

# Archive pages for deleted markdown files
ccfm --config ccfm.yaml deploy --directory docs --archive-orphans

# Initialize management pages
ccfm --config ccfm.yaml init
```

See [ccfm-convert](https://github.com/stevesimpson418/ccfm-convert) for full documentation.
