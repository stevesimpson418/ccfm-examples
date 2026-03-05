# Example: Customer Product Docs

> **Integration methods**: PyPI (plan on PR) + Docker (deploy on merge)

"Nexus Platform" is a fictional B2B SaaS product. Engineering maintains customer-facing
documentation in Git. A `--plan` check runs on every PR to surface documentation changes;
a full deploy runs on merge using Docker — no Python required in the deploy environment.

## What this demonstrates

- `--plan` as a PR check (exit code 2 = pending changes, informational)
- Docker-based deployment (zero Python dependency in CI)
- PyPI-based plan step (`pip install ccfm-convert` in a lightweight check job)
- Smart page links between docs sections
- `deploy_page: false` for draft pages

## Prerequisites

1. A Confluence Cloud space (e.g. `NEXUS`)
2. An Atlassian API token
3. Docker available in your CI environment

## GitHub Secrets required

| Secret | Value |
| --- | --- |
| `CONFLUENCE_DOMAIN` | e.g. `nexus.atlassian.net` |
| `CONFLUENCE_EMAIL` | Your Atlassian account email |
| `CONFLUENCE_TOKEN` | Atlassian API token |

## Running locally

```bash
# With pip
pip install ccfm-convert
export CONFLUENCE_EMAIL=you@nexus.com
export CONFLUENCE_TOKEN=your-token
ccfm --directory docs --plan    # preview
ccfm --directory docs           # deploy

# With Docker
docker run --rm \
  -v $(pwd)/docs:/docs \
  -v $(pwd)/.ccfm-state.json:/.ccfm-state.json \
  ghcr.io/stevesimpson418/ccfm-convert:latest \
    --domain nexus.atlassian.net \
    --email you@nexus.com \
    --token $CONFLUENCE_TOKEN \
    --space NEXUS \
    --directory /docs \
    --state /.ccfm-state.json
```

## How the CI pipeline works

- **On pull request**: `plan.yml` runs `ccfm --plan` (PyPI install). Exit code 2 means
  documentation changes are pending — visible as an informational check on the PR.
- **On merge to main**: `deploy.yml` runs a full deploy via Docker with `--changed-only`.
