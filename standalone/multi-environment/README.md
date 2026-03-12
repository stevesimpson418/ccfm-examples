# Standalone -- Multi Environment

One docs tree applied to multiple Confluence spaces (e.g. staging and production). The `CONFLUENCE_SPACE` variable controls which space is targeted.

## Prerequisites

- Python 3.12+
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))
- Two Confluence spaces: `ENG-STAGE` (staging) and `ENG` (production)

## Setup

1. Copy the shared sample docs into this directory:

   ```bash
   cp -r ../../_shared/docs docs/
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

## Workflow

Apply to staging first, verify, then promote to production:

```bash
make init-staging
make plan-staging
make apply-staging

# After review...
make init-production
make plan-production
make apply-production
```

## Make Targets

| Target | Space | Description |
| --- | --- | --- |
| `install` | -- | Install ccfm-convert via pip |
| `init-staging` | `ENG-STAGE` | Create parent page in staging space |
| `plan-staging` | `ENG-STAGE` | Dry-run against staging |
| `apply-staging` | `ENG-STAGE` | Sync docs to staging |
| `init-production` | `ENG` | Create parent page in production space |
| `plan-production` | `ENG` | Dry-run against production |
| `apply-production` | `ENG` | Sync docs to production |

## How It Works

The `ccfm.yaml` config uses `${CONFLUENCE_SPACE}` as a variable. Each Make target sets `CONFLUENCE_SPACE` inline, so the same config file and docs tree can target different spaces without duplication.

---

See [standalone/README.md](../README.md) for an overview of all standalone patterns.
