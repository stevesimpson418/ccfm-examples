# Standalone -- Multi Source

Two separate doc trees in one repository, each synced to a different Confluence space. Each source has its own config file and Make targets.

## Prerequisites

- Python 3.12+
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))
- Two Confluence spaces: `ENG` (API docs) and `WIKI` (team wiki)

## Setup

1. Copy the shared sample docs into both directories:

   ```bash
   cp -r ../../_shared/docs docs/
   cp -r ../../_shared/docs docs-wiki/
   ```

2. Set the required environment variables:

   ```bash
   export CONFLUENCE_DOMAIN="acme-corp.atlassian.net"
   export CONFLUENCE_EMAIL="you@acme-corp.com"
   export CONFLUENCE_TOKEN="your-api-token"
   ```

3. Install ccfm-convert:

   ```bash
   make install
   ```

## Usage

Apply each source independently or together:

```bash
# Individual sources
make init-api
make plan-api
make apply-api

make init-wiki
make plan-wiki
make apply-wiki

# Both sources at once
make plan-all
make apply-all
```

## Make Targets

| Target | Config | Space | Description |
| --- | --- | --- | --- |
| `install` | -- | -- | Install ccfm-convert via pip |
| `init-api` | `ccfm-api.yaml` | `ENG` | Create parent page for API docs |
| `plan-api` | `ccfm-api.yaml` | `ENG` | Dry-run for API docs |
| `apply-api` | `ccfm-api.yaml` | `ENG` | Sync API docs |
| `init-wiki` | `ccfm-wiki.yaml` | `WIKI` | Create parent page for wiki docs |
| `plan-wiki` | `ccfm-wiki.yaml` | `WIKI` | Dry-run for wiki docs |
| `apply-wiki` | `ccfm-wiki.yaml` | `WIKI` | Sync wiki docs |
| `plan-all` | both | both | Dry-run for all sources |
| `apply-all` | both | both | Sync all sources |

## How It Works

Each doc source gets its own `ccfm-*.yaml` config pointing at a different `docs_root` and Confluence `space`. The Makefile wraps both into unified `plan-all` and `apply-all` targets for convenience.

---

See [standalone/README.md](../README.md) for an overview of all standalone patterns.
