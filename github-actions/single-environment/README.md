# GitHub Actions -- Single Environment

One docs tree synced to one Confluence space via the `ccfm-convert` GitHub Action. Plans on PR, deploys on push to `main`.

## Prerequisites

- GitHub repository with Actions enabled
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))

## Setup

1. Add the following repository secrets in **Settings > Secrets and variables > Actions**:

   | Secret | Value |
   |--------|-------|
   | `CONFLUENCE_DOMAIN` | `acme-corp.atlassian.net` |
   | `CONFLUENCE_EMAIL` | `you@acme-corp.com` |
   | `CONFLUENCE_TOKEN` | Your Confluence API token |

2. Copy the shared sample docs into this directory:

   ```bash
   cp -r ../../_shared/docs docs/
   ```

3. Copy `.github/workflows/deploy.yml` to your repository's `.github/workflows/` directory.

4. Update the `space` input in the workflow to match your Confluence space key.

## How It Works

- **Pull request** -- runs `--plan` to preview what would change. The plan output appears in the Actions log. No changes are made to Confluence.
- **Push to main** -- deploys docs to Confluence, archiving any orphaned pages (pages whose source markdown was deleted).

## Workflow Inputs

| Input | Description |
|-------|-------------|
| `domain` | Confluence Cloud domain |
| `email` | Confluence user email |
| `token` | Confluence API token |
| `space` | Confluence space key |
| `directory` | Path to the docs directory |
| `version` | ccfm-convert version to use |
| `args` | Additional CLI arguments |

---

See [github-actions/README.md](../README.md) for an overview of all GitHub Actions patterns.
