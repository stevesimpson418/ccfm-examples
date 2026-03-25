# Docker -- Single Environment

One docs tree synced to one Confluence space. Run ccfm-convert via Docker Compose -- no Python installation needed.

## Prerequisites

- Docker ([get Docker](https://docs.docker.com/get-docker/))
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))

## Setup

1. Copy the shared sample docs into this directory:

   ```bash
   cp -r ../../_shared/docs docs/
   ```

2. Create your `.env` file from the example:

   ```bash
   cp .env.example .env
   ```

   Edit `.env` and fill in your Confluence credentials.

3. Initialize the Confluence parent page:

   ```bash
   docker compose run --rm ccfm --config /ccfm.yaml init
   ```

4. Preview what will be synced:

   ```bash
   docker compose run --rm ccfm --config /ccfm.yaml plan
   ```

5. Apply changes to Confluence:

   ```bash
   docker compose run --rm ccfm --config /ccfm.yaml apply --auto-approve
   ```

## How It Works

The `docker-compose.yml` mounts your local `docs/` directory and `ccfm.yaml` config into the container. The Docker image's entrypoint is `ccfm`, so `docker compose run` arguments are passed directly as CLI args.

The config uses `docs_root: /docs` because docs are mounted at `/docs` inside the container.

---

See [docker/README.md](../README.md) for an overview of all Docker patterns.
