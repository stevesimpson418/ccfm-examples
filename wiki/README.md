# Example: Company Wiki

> **Integration method**: GitHub Action (`uses: stevesimpson418/ccfm-convert@v0.1.0`)

A fictional company "Acme Corp" maintains their entire internal wiki in Git.
Engineers write Markdown, CI deploys to Confluence automatically on every merge to `main`.

## What this demonstrates

- `ccfm.yaml` config file with `${ENV_VAR}` credential interpolation
- Directory-based page hierarchy with `.page_content.md` container pages
- `--changed-only` for fast CI deploys (only re-deploys what changed)
- `--archive-orphans` to clean up Confluence when pages are deleted from the repo
- Labels, status badges, panels, and expand blocks throughout

## Prerequisites

1. A Confluence Cloud space (e.g. `WIKI`)
2. An Atlassian API token

## GitHub Secrets required

| Secret | Value |
| --- | --- |
| `CONFLUENCE_DOMAIN` | e.g. `acme.atlassian.net` |
| `CONFLUENCE_EMAIL` | Your Atlassian account email |
| `CONFLUENCE_TOKEN` | Atlassian API token |

## Running locally

```bash
pip install ccfm-convert

# Edit ccfm.yaml with your domain/space, then:
export CONFLUENCE_EMAIL=you@acme.com
export CONFLUENCE_TOKEN=your-token

# Preview what would deploy
ccfm --directory docs --plan

# Deploy
ccfm --directory docs
```

## How the CI pipeline works

On every push to `main` that touches `docs/**`, the workflow:
1. Installs ccfm-convert via the GitHub Action
2. Deploys only changed files (`--changed-only`)
3. Links each page back to its source file on GitHub
