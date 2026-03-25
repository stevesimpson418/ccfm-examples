# GitHub Actions

Deploy Markdown to Confluence using the
[ccfm-convert GitHub Action](https://github.com/stevesimpson418/ccfm-convert).
Plan on pull requests, deploy on push to main.

## When to use this

- GitHub-hosted repositories
- Teams already using GitHub Actions
- Minimal setup — no Python or Docker knowledge needed

## Prerequisites

- A GitHub repository with Actions enabled
- A Confluence Cloud space with API access
- An [Atlassian API token](https://id.atlassian.com/manage-profile/security/api-tokens)

## Setup

1. Copy the shared example docs into place:

   ```bash
   cp -r ../_shared/docs ./docs
   ```

2. Add repository secrets (Settings > Secrets and variables > Actions):

   | Secret | Value |
   | --- | --- |
   | `CONFLUENCE_DOMAIN` | `your-org.atlassian.net` |
   | `CONFLUENCE_EMAIL` | `you@example.com` |
   | `CONFLUENCE_TOKEN` | Your API token |

3. Copy `ccfm.yaml` to your repo root and `.github/workflows/deploy.yml` into
   your workflows directory.

## What's inside

| File | Purpose |
| --- | --- |
| `ccfm.yaml` | CCFM configuration — credentials, space, docs root |
| `.github/workflows/deploy.yml` | Workflow with plan and deploy jobs |

## Workflow jobs

| Job | Trigger | What it does |
| --- | --- | --- |
| `plan` | Pull request touching `docs/**` | Shows what pages would change |
| `deploy` | Push to `main` touching `docs/**` | Applies changes to Confluence |

Both jobs run `ccfm init` first (idempotent) to ensure the management page exists.

## Multi-environment deployments

To deploy to staging and production spaces, use GitHub Environments with
environment-specific variables. Set `CONFLUENCE_SPACE` per environment and use
`space: ${CONFLUENCE_SPACE}` in your `ccfm.yaml`.

## Further reading

- [ccfm.io](https://ccfm.io) — full documentation
- [Design philosophy](https://ccfm.io/#design-philosophy) — one config per space
