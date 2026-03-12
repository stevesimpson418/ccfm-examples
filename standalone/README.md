# Standalone (pip install)

Run ccfm-convert directly via `pip install`. No Docker or CI platform required. This is the simplest way to get started.

## Patterns

| Pattern | Description |
| --- | --- |
| [single-environment](./single-environment) | One docs tree synced to one Confluence space |
| [multi-environment](./multi-environment) | One docs tree synced to staging and production spaces |
| [multi-source](./multi-source) | Two doc trees in one repo, each targeting a different space |

## Common Setup

All standalone examples share the same prerequisites:

1. **Python 3.12+** installed
2. **Install ccfm-convert:**

   ```bash
   pip install ccfm-convert
   ```

3. **Set environment variables:**

   ```bash
   export CONFLUENCE_DOMAIN="acme-corp.atlassian.net"
   export CONFLUENCE_EMAIL="you@acme-corp.com"
   export CONFLUENCE_TOKEN="your-api-token"
   ```

Each example includes a Makefile that wraps the common ccfm commands. See the individual READMEs for pattern-specific details.
