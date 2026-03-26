# Multi-Space (Advanced)

Deploy multiple doc trees from one repository to different Confluence spaces.

> **Note:** The recommended approach is one repository per space. This pattern is
> for cases where you genuinely need multiple spaces managed from a single repo.
> See the [design philosophy](https://ccfm.io/#design-philosophy) for details.

## When to use this

- One repo contains documentation for multiple Confluence spaces
- Each space has its own independent doc tree and config
- Teams share a repo but publish to different spaces

## How it works

Each Confluence space gets its own `ccfm-*.yaml` config file with an independent
`docs_root`. CCFM tracks state per-space, so there are no conflicts between them.

```
multi-space/
├── ccfm-api.yaml       → space: ENG,  docs_root: docs
├── ccfm-wiki.yaml      → space: WIKI, docs_root: docs-wiki
└── Makefile
```

Since config files are not named `ccfm.yaml` (the default), the `--config` flag
is required to select which space to target.

## Prerequisites

- Python 3.12+ with `ccfm-convert` installed
- Two Confluence Cloud spaces with API access
- An [Atlassian API token](https://id.atlassian.com/manage-profile/security/api-tokens)

## Quick start

```bash
# Copy both doc sets into place
cp -r ../_shared/docs ./docs
cp -r ../_shared/docs-wiki ./docs-wiki

# Set your credentials
export CONFLUENCE_DOMAIN=your-org.atlassian.net
export CONFLUENCE_EMAIL=you@example.com
export CONFLUENCE_TOKEN=your-api-token

make install

# Work with API docs (ENG space)
make init-api
make plan-api
make apply-api

# Work with wiki docs (WIKI space)
make init-wiki
make plan-wiki
make apply-wiki

# Or target both at once
make plan-all
make apply-all
```

## What's inside

| File | Purpose |
| --- | --- |
| `ccfm-api.yaml` | Config for ENG space — `docs_root: docs` |
| `ccfm-wiki.yaml` | Config for WIKI space — `docs_root: docs-wiki` |
| `Makefile` | Targets per space plus combined targets |

## CI integration

In CI, run each config as a separate step or job:

```bash
ccfm --config ccfm-api.yaml init
ccfm --config ccfm-api.yaml apply --auto-approve

ccfm --config ccfm-wiki.yaml init
ccfm --config ccfm-wiki.yaml apply --auto-approve
```

## Further reading

- [ccfm.io](https://ccfm.io) — full documentation
- [Design philosophy](https://ccfm.io/#design-philosophy) — one config per space
