# CCFM Examples

Example projects demonstrating real-world usage of
[ccfm-convert](https://github.com/stevesimpson418/ccfm-convert) — a CLI tool that
converts Markdown to Atlassian Document Format (ADF) and syncs pages to Confluence Cloud.

> Requires **ccfm-convert v2.0.0+**

## Choose your deployment method

| I want to run from... | Example | What's inside |
| --- | --- | --- |
| GitHub Actions | [`github-actions/`](github-actions/) | Workflow using the `ccfm-convert` GitHub Action |
| GitLab CI | [`gitlab-ci/`](gitlab-ci/) | Pipeline using the `ccfm-convert` Docker image natively |
| Docker (any CI or local) | [`docker/`](docker/) | Docker Compose config — works anywhere Docker runs |
| pip install (any environment) | [`standalone/`](standalone/) | Makefile wrapping the `ccfm` CLI directly |

## Advanced patterns

| Pattern | Example | When to use it |
| --- | --- | --- |
| **Multi-space** | [`multi-space/`](multi-space/) | Multiple doc trees in one repo targeting different Confluence spaces |

**Multi-environment** (staging + production) doesn't need a separate example — use
environment variable interpolation in your `ccfm.yaml`:

```yaml
space: ${CONFLUENCE_SPACE}
```

Then set the variable at deploy time:

```bash
CONFLUENCE_SPACE=ENG-STAGE ccfm plan
CONFLUENCE_SPACE=ENG ccfm apply --auto-approve
```

## Quick start

The fastest way to try ccfm-convert — no CI or Docker needed:

```bash
git clone https://github.com/stevesimpson418/ccfm-examples.git
cd ccfm-examples/standalone

# Copy the example docs into place
cp -r ../_shared/docs ./docs

# Set your Confluence credentials
export CONFLUENCE_DOMAIN=your-org.atlassian.net
export CONFLUENCE_EMAIL=you@example.com
export CONFLUENCE_TOKEN=your-api-token

# Install and run
make install
make init      # initialize ccfm management page (idempotent)
make plan      # preview what would change (safe, read-only)
make apply     # apply changes to Confluence
```

## Shared docs

All examples reference the same documentation content in [`_shared/docs/`](_shared/docs/).
Copy this directory into your chosen example as `docs/` before running.

For the multi-space example, a second doc set is available at
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
# Initialize management page (idempotent, run once per space)
ccfm init

# Preview what would change (no modifications made)
ccfm plan

# Apply changes to Confluence
ccfm apply --auto-approve

# Force re-apply all pages regardless of state
ccfm apply --auto-approve --force

# Inspect ADF output for a single file (no API calls)
ccfm plan --debug-file path/to/file.md
```

## Design philosophy

Each Confluence space is managed by exactly one `ccfm.yaml` configuration.
Deployment state is stored in Confluence itself — no local state files to commit.
See [ccfm.io](https://ccfm.io/#design-philosophy) for details.

See [ccfm.io](https://ccfm.io) for full documentation.
