# Example: Microservice Architecture Docs — GitHub Action

> **Integration method**: GitHub Action (`uses: stevesimpson418/ccfm-convert@v0.1.0`)

"Order Service" is a fictional microservice. Architecture decision records (ADRs), API
documentation, and runbooks live alongside the service code. The CCFM deploy pipeline
runs inside the service's existing CI workflow.

## What this demonstrates

- `--plan` as a **blocking** PR gate (teams must consciously approve doc changes)
- `--archive-orphans` to remove Confluence pages when ADRs or runbooks are deleted
- `include_page_metadata: true` to surface author and labels on Confluence pages
- Systematic use of labels (`adr`, `runbook`, `api-reference`)
- ADR format with status badges (`::Accepted::green::`, `::Proposed::yellow::`)
- Multi-job CI workflow: `plan` on PR, `deploy` on merge — both using the GitHub Action

## Prerequisites

1. A Confluence Cloud space (e.g. `ENG`)
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
export CONFLUENCE_EMAIL=you@acme.com
export CONFLUENCE_TOKEN=your-token

# Preview impact of doc changes on a feature branch
ccfm --config ccfm.yaml --directory docs --plan

# Deploy (after merge)
ccfm --config ccfm.yaml --directory docs --archive-orphans
```

## How the CI pipeline works

- **On pull request** (`docs/**` changes): `plan` job runs `--plan`. Exit code 2 **fails
  the check** — forcing the team to review what Confluence pages will change before merging.
- **On merge to main**: `deploy` job does a full deploy with `--archive-orphans`, cleaning
  up any Confluence pages whose source files were deleted in the PR.
