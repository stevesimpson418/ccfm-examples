# GitHub Actions -- Multi-Source

Deploy multiple doc trees from a single repository to different Confluence spaces. Each source directory has its own `ccfm-*.yaml` config targeting a separate space.

## Prerequisites

- GitHub repository with Actions enabled
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))
- Two Confluence spaces (e.g., `ENG` for API docs, `WIKI` for team wiki)

## Setup

1. Add repository secrets in **Settings > Secrets and variables > Actions**:

   | Secret | Value |
   |--------|-------|
   | `CONFLUENCE_DOMAIN` | `acme-corp.atlassian.net` |
   | `CONFLUENCE_EMAIL` | `you@acme-corp.com` |
   | `CONFLUENCE_TOKEN` | Your Confluence API token |

2. Create the two doc directories:

   ```bash
   cp -r ../../_shared/docs docs/
   mkdir -p docs-wiki && cp -r ../../_shared/docs/* docs-wiki/
   ```

3. Copy `.github/workflows/deploy.yml` to your repository's `.github/workflows/` directory.

## Structure

```
repo/
  ccfm-api.yaml      # config for API docs -> ENG space
  ccfm-wiki.yaml     # config for wiki docs -> WIKI space
  docs/              # API documentation source
  docs-wiki/         # Wiki documentation source
  .github/workflows/
    deploy.yml       # single workflow handles both sources
```

## How It Works

- **Pull request** -- plans both doc sources independently. Each plan step shows what would change in its target space.
- **Push to main** -- deploys both doc sources sequentially. API docs go to the `ENG` space, wiki docs go to the `WIKI` space.

Each ccfm config file (`ccfm-api.yaml`, `ccfm-wiki.yaml`) defines its own `space` and `docs_root`, keeping the sources fully independent.

---

See [github-actions/README.md](../README.md) for an overview of all GitHub Actions patterns.
