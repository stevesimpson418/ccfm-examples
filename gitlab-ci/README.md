# GitLab CI

Sync Markdown docs to Confluence using the `ccfm-convert` Docker image in GitLab CI pipelines.

## Docker Image

All examples use the official image directly:

```yaml
image: ghcr.io/stevesimpson418/ccfm-convert:latest
```

No action wrapper -- the `ccfm` CLI is available in `$PATH` inside the container.

## Examples

| Pattern | Description |
|---------|-------------|
| [`single-environment/`](single-environment/) | One docs tree, one Confluence space. Plan on MR, apply on merge. |
| [`multi-environment/`](multi-environment/) | Staging and production spaces using GitLab Environments and scoped variables. |
| [`multi-source/`](multi-source/) | Two doc trees synced to two spaces from a single pipeline. |

## Common Setup

All examples require three CI/CD variables configured in **Settings > CI/CD > Variables**:

| Variable | Value |
|----------|-------|
| `CONFLUENCE_DOMAIN` | Your Confluence Cloud domain (e.g. `acme-corp.atlassian.net`) |
| `CONFLUENCE_EMAIL` | Confluence user email |
| `CONFLUENCE_TOKEN` | Confluence API token (mark as **Masked**) |

## Markdown Linting

Each pipeline includes a `lint:markdown` job using [`markdownlint-cli2`](https://github.com/DavidAnson/markdownlint-cli2) to catch formatting issues before applying. The lint job runs in a lightweight `node:20-alpine` image and gates the plan/apply stages.

---

See the [top-level README](../README.md) for an overview of all ccfm-convert examples.
