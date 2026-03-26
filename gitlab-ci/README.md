# GitLab CI

Deploy Markdown to Confluence using the ccfm-convert Docker image in a GitLab
CI/CD pipeline. Plan on merge requests, apply on push to the default branch.

## When to use this

- GitLab-hosted repositories
- Teams already using GitLab CI/CD
- Environments where the ccfm Docker image runs natively in pipeline jobs

## Prerequisites

- A GitLab project with CI/CD enabled
- A Confluence Cloud space with API access
- An [Atlassian API token](https://id.atlassian.com/manage-profile/security/api-tokens)

## Setup

1. Copy the shared example docs into place:

   ```bash
   cp -r ../_shared/docs ./docs
   ```

2. Set CI/CD variables in your GitLab project (Settings > CI/CD > Variables):

   | Variable | Value |
   | --- | --- |
   | `CONFLUENCE_DOMAIN` | `your-org.atlassian.net` |
   | `CONFLUENCE_EMAIL` | `you@example.com` |
   | `CONFLUENCE_TOKEN` | Your API token (masked) |

3. Copy `ccfm.yaml` and `.gitlab-ci.yml` into your project root.

## What's inside

| File | Purpose |
| --- | --- |
| `ccfm.yaml` | CCFM configuration — credentials, space, docs root |
| `.gitlab-ci.yml` | Pipeline with lint, plan, and apply stages |

## Pipeline stages

| Stage | When | What it does |
| --- | --- | --- |
| `lint` | MRs and default branch | Runs markdownlint on `docs/**/*.md` |
| `plan` | MRs only | Shows what Confluence pages would change |
| `apply` | Default branch only | Applies changes to Confluence |

All plan and apply jobs run `ccfm init` first (idempotent) to ensure the
management page exists.

## Multi-environment deployments

To deploy to staging and production spaces, use CI/CD variable scoping. Set
`CONFLUENCE_SPACE` as an environment-scoped variable and use
`space: ${CONFLUENCE_SPACE}` in your `ccfm.yaml`.

## Further reading

- [ccfm.io](https://ccfm.io) — full documentation
- [Design philosophy](https://ccfm.io/#design-philosophy) — one config per space
