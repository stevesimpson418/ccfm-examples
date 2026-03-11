# GitHub Actions -- Multi-Environment

Deploy docs to separate Confluence spaces per environment using GitHub Environments. Staging deploys automatically on push to `main`; Production requires manual approval.

## Prerequisites

- GitHub repository with Actions enabled
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))
- Two Confluence spaces (one for staging, one for production)

## Setup

### 1. Create GitHub Environments

Go to **Settings > Environments** and create two environments:

| Environment | Purpose |
|-------------|---------|
| `Staging` | Auto-deploys on push to `main` |
| `Production` | Requires reviewer approval before deploy |

For `Production`, enable **Required reviewers** and add the appropriate team members.

### 2. Configure environment variables and secrets

Each environment needs its own `CONFLUENCE_SPACE` variable and can optionally override credentials.

**Repository-level secrets** (shared across environments):

| Secret | Value |
|--------|-------|
| `CONFLUENCE_DOMAIN` | `acme-corp.atlassian.net` |
| `CONFLUENCE_EMAIL` | `you@acme-corp.com` |
| `CONFLUENCE_TOKEN` | Your Confluence API token |

**Per-environment variables** (Settings > Environments > [env] > Variables):

| Variable | Staging | Production |
|----------|---------|------------|
| `CONFLUENCE_SPACE` | `ENG_STAGING` | `ENG` |

### 3. Add docs and workflow

1. Copy the shared sample docs into this directory:

   ```bash
   cp -r ../../_shared/docs docs/
   ```

2. Copy `.github/workflows/deploy.yml` to your repository's `.github/workflows/` directory.

## How It Works

- **Pull request** -- runs `plan` to preview changes. No Confluence modifications.
- **Push to main** -- initializes state and deploys to the Staging environment via `apply --auto-approve`.
- **After staging succeeds** -- the Production job waits for manual approval from a required reviewer, then initializes state and deploys to the production Confluence space.

Each environment resolves `CONFLUENCE_SPACE` from its own variable configuration, so the same workflow targets different spaces depending on the environment.

---

See [github-actions/README.md](../README.md) for an overview of all GitHub Actions patterns.
