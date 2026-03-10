# Standalone -- Single Environment

One docs tree synced to one Confluence space. Run locally or in any CI system.

## Prerequisites

- Python 3.12+
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))

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

4. Initialize the Confluence parent page:

   ```bash
   make init
   ```

5. Preview what will be synced:

   ```bash
   make plan
   ```

6. Deploy to Confluence:

   ```bash
   make deploy
   ```

## Make Targets

| Target        | Description                                      |
|---------------|--------------------------------------------------|
| `install`     | Install ccfm-convert via pip                     |
| `init`        | Create the parent page in Confluence             |
| `plan`        | Dry-run showing what would be created or updated |
| `deploy`      | Sync changed files and archive orphaned pages    |
| `deploy-full` | Full sync (all files, not just changed)          |

---

See [standalone/README.md](../README.md) for an overview of all standalone patterns.
