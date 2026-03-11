# Docker -- Multi Source

Two separate doc trees in one repository, each synced to a different Confluence space via its own Docker Compose service. Each source has its own config file and volume mount.

## Prerequisites

- Docker ([get Docker](https://docs.docker.com/get-docker/))
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))
- Two Confluence spaces: `ENG` (API docs) and `WIKI` (team wiki)

## Setup

1. Copy the shared sample docs into both directories:

   ```bash
   cp -r ../../_shared/docs docs/
   cp -r ../../_shared/docs docs-wiki/
   ```

2. Create your `.env` file from the example:

   ```bash
   cp .env.example .env
   ```

   Edit `.env` and fill in your Confluence credentials.

3. Initialize both spaces:

   ```bash
   docker compose run --rm api-docs --config /ccfm.yaml init
   docker compose run --rm wiki-docs --config /ccfm.yaml init
   ```

## Usage

Apply API docs:

```bash
docker compose run --rm api-docs --config /ccfm.yaml apply --directory /docs --auto-approve
```

Apply wiki docs:

```bash
docker compose run --rm wiki-docs --config /ccfm.yaml apply --directory /docs-wiki --auto-approve
```

Apply all sources at once:

```bash
docker compose up
```

Preview changes (dry-run):

```bash
docker compose run --rm api-docs --config /ccfm.yaml plan --directory /docs
docker compose run --rm wiki-docs --config /ccfm.yaml plan --directory /docs-wiki
```

## How It Works

Each service in `docker-compose.yml` mounts a different docs directory and config file. The config files target different Confluence spaces:

| Service     | Config            | Docs mount   | Space  |
|-------------|-------------------|--------------|--------|
| `api-docs`  | `ccfm-api.yaml`  | `./docs`     | `ENG`  |
| `wiki-docs` | `ccfm-wiki.yaml` | `./docs-wiki`| `WIKI` |

Both configs are mounted to `/ccfm.yaml` inside their respective containers, keeping the command interface consistent.

---

See [docker/README.md](../README.md) for an overview of all Docker patterns.
