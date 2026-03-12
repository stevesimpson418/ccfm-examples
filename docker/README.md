# Docker

Run ccfm-convert via Docker. Works anywhere Docker runs -- local machines, CI systems, servers. No Python installation required.

**Docker image:** `ghcr.io/stevesimpson418/ccfm-convert:latest`

## Patterns

| Pattern | Description |
| --- | --- |
| [single-environment](./single-environment) | One docs tree synced to one Confluence space |
| [multi-environment](./multi-environment) | One docs tree synced to staging and production spaces |
| [multi-source](./multi-source) | Two doc trees in one repo, each targeting a different space |

## Common Setup

All Docker examples share the same prerequisites:

1. **Docker** installed ([get Docker](https://docs.docker.com/get-docker/))
2. **Confluence API token** ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))
3. **Create `.env` from `.env.example`** in the example directory and fill in your credentials

Each example includes a `docker-compose.yml` that wraps the container configuration. See the individual READMEs for pattern-specific details.
