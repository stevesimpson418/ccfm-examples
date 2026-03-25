# Standalone (pip install)

Deploy Markdown to Confluence using the `ccfm` CLI installed via pip.

## When to use this

- Local development and testing
- Simple CI systems with Python available
- Any environment with Python 3.12+

## Prerequisites

- Python 3.12+
- A Confluence Cloud space with API access
- An [Atlassian API token](https://id.atlassian.com/manage-profile/security/api-tokens)

## Quick start

```bash
# Copy the shared example docs into place
cp -r ../_shared/docs ./docs

# Set your Confluence credentials
export CONFLUENCE_DOMAIN=your-org.atlassian.net
export CONFLUENCE_EMAIL=you@example.com
export CONFLUENCE_TOKEN=your-api-token

# Install and run
make install
make init      # initialize ccfm management page (idempotent)
make plan      # preview what would change (safe, read-only)
make apply     # apply changes to Confluence
```

## What's inside

| File | Purpose |
| --- | --- |
| `ccfm.yaml` | CCFM configuration — credentials, space, docs root |
| `Makefile` | Convenience targets wrapping the `ccfm` CLI |

## Multi-environment deployments

To deploy the same docs to multiple spaces (e.g. staging and production), use
environment variable interpolation in `ccfm.yaml`:

```yaml
space: ${CONFLUENCE_SPACE}
```

Then set the variable at runtime:

```bash
CONFLUENCE_SPACE=ENG-STAGE ccfm plan
CONFLUENCE_SPACE=ENG ccfm apply --auto-approve
```

No separate config files needed — one `ccfm.yaml` handles all environments.

## Further reading

- [ccfm.io](https://ccfm.io) — full documentation
- [Design philosophy](https://ccfm.io/#design-philosophy) — one config per space
