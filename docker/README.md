# Docker

Deploy Markdown to Confluence using the ccfm-convert Docker image. No Python
installation needed — works anywhere Docker runs.

## When to use this

- Environments without Python
- CI systems with Docker support
- Consistent, reproducible deployments

## Prerequisites

- Docker and Docker Compose
- A Confluence Cloud space with API access
- An [Atlassian API token](https://id.atlassian.com/manage-profile/security/api-tokens)

## Quick start

```bash
# Copy the shared example docs into place
cp -r ../_shared/docs ./docs

# Create a .env file with your credentials
cat > .env <<EOF
CONFLUENCE_DOMAIN=your-org.atlassian.net
CONFLUENCE_EMAIL=you@example.com
CONFLUENCE_TOKEN=your-api-token
EOF

# Initialize, plan, and apply
docker compose run --rm ccfm ccfm init
docker compose run --rm ccfm ccfm plan
docker compose run --rm ccfm ccfm apply --auto-approve
```

## What's inside

| File | Purpose |
| --- | --- |
| `ccfm.yaml` | CCFM configuration — credentials, space, docs root |
| `docker-compose.yml` | Service definition with volume mounts |
| `.env` | Credentials (create this yourself, not committed) |

## How it works

The Docker Compose file mounts your local `docs/` directory and `ccfm.yaml` into
`/workspace` inside the container. The `ccfm.yaml` uses `/workspace/docs` as `docs_root`
to match the container path.

```yaml
volumes:
  - ./docs:/workspace/docs:ro      # your docs (read-only)
  - ./ccfm.yaml:/workspace/ccfm.yaml:ro  # config (read-only)
working_dir: /workspace            # ccfm finds ccfm.yaml here
```

Credentials are injected via the `.env` file and interpolated by `ccfm.yaml` using
`${CONFLUENCE_DOMAIN}` syntax.

## Further reading

- [ccfm.io](https://ccfm.io) — full documentation
- [Design philosophy](https://ccfm.io/#design-philosophy) — one config per space
