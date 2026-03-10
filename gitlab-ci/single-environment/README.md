# GitLab CI -- Single Environment

One docs tree synced to one Confluence space using the `ccfm-convert` Docker image in GitLab CI. Plans on MR, deploys on merge to the default branch.

## Prerequisites

- GitLab project with CI/CD enabled
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))

## Setup

1. Copy the shared sample docs into this directory:

   ```bash
   cp -r ../../_shared/docs docs/
   ```

2. Add the following CI/CD variables in **Settings > CI/CD > Variables**:

   | Variable | Value | Options |
   |----------|-------|---------|
   | `CONFLUENCE_DOMAIN` | `acme-corp.atlassian.net` | |
   | `CONFLUENCE_EMAIL` | `you@acme-corp.com` | |
   | `CONFLUENCE_TOKEN` | Your Confluence API token | Masked |

3. Copy `.gitlab-ci.yml` to the root of your repository.

4. Update the `space` value in `ccfm.yaml` to match your Confluence space key.

## How It Works

- **Merge request** -- runs markdown linting and `--plan` to preview what would change. No changes are made to Confluence.
- **Push to default branch** -- deploys docs to Confluence with `--changed-only` and `--archive-orphans`.

## Pipeline Stages

| Stage | Job | Trigger |
|-------|-----|---------|
| `lint` | `lint:markdown` | MR and default branch |
| `plan` | `plan:confluence` | MR only |
| `deploy` | `deploy:confluence` | Default branch only |

---

See [gitlab-ci/README.md](../README.md) for an overview of all GitLab CI patterns.
