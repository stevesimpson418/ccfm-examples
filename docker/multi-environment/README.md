# Docker -- Multi-Environment

One docs tree deployed to multiple Confluence spaces (e.g. staging and production). Same compose file, different `.env` files per environment.

## Prerequisites

- Docker ([get Docker](https://docs.docker.com/get-docker/))
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))
- Two Confluence spaces: `ENG-STAGE` (staging) and `ENG` (production)

## Setup

1. Copy the shared sample docs into this directory:

   ```bash
   cp -r ../../_shared/docs docs/
   ```

2. Edit `.env.staging` and `.env.production` with your Confluence credentials.

3. Initialize both environments:

   ```bash
   docker compose --env-file .env.staging run --rm ccfm --config /ccfm.yaml init
   docker compose --env-file .env.production run --rm ccfm --config /ccfm.yaml init
   ```

## Usage

Deploy to staging:

```bash
docker compose --env-file .env.staging run --rm ccfm --config /ccfm.yaml deploy --directory /docs --changed-only --archive-orphans
```

Deploy to production:

```bash
docker compose --env-file .env.production run --rm ccfm --config /ccfm.yaml deploy --directory /docs --changed-only --archive-orphans
```

Preview changes (dry-run):

```bash
docker compose --env-file .env.staging run --rm ccfm --config /ccfm.yaml deploy --directory /docs --plan
```

## How It Works

The `ccfm.yaml` config uses `${CONFLUENCE_SPACE}` as a variable, which is resolved from the `.env` file passed to `docker compose`. This lets you target different Confluence spaces with the same compose file and config.

| File              | Space        | Purpose    |
|-------------------|--------------|------------|
| `.env.staging`    | `ENG-STAGE`  | Staging    |
| `.env.production` | `ENG`        | Production |

---

See [docker/README.md](../README.md) for an overview of all Docker patterns.
