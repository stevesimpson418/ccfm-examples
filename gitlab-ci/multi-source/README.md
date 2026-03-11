# GitLab CI -- Multi-Source

Two doc trees synced to two Confluence spaces from a single pipeline. Each source has its own `ccfm.yaml` config and applies independently based on which files changed.

## Prerequisites

- GitLab project with CI/CD enabled
- Two Confluence spaces (e.g. `ENG` for API docs, `WIKI` for team wiki)
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))

## Setup

1. Copy the shared sample docs into this directory:

   ```bash
   cp -r ../../_shared/docs docs/
   cp -r ../../_shared/docs-wiki docs-wiki/
   ```

2. Add CI/CD variables in **Settings > CI/CD > Variables**:

   | Variable | Value | Options |
   |----------|-------|---------|
   | `CONFLUENCE_DOMAIN` | `acme-corp.atlassian.net` | |
   | `CONFLUENCE_EMAIL` | `you@acme-corp.com` | |
   | `CONFLUENCE_TOKEN` | Your Confluence API token | Masked |

3. Copy `.gitlab-ci.yml`, `ccfm-api.yaml`, and `ccfm-wiki.yaml` to your repository root.

4. Update the `space` values in each config to match your Confluence space keys.

## How It Works

| Source | Config | Space | Plan trigger | Deploy trigger |
|--------|--------|-------|--------------|----------------|
| `docs/` | `ccfm-api.yaml` | `ENG` | MR with `docs/` changes | Default branch with `docs/` changes |
| `docs-wiki/` | `ccfm-wiki.yaml` | `WIKI` | MR with `docs-wiki/` changes | Default branch with `docs-wiki/` changes |

Each doc source is fully independent. Changing a file in `docs/` does not trigger an apply for `docs-wiki/`, and vice versa. The `.ccfm-init` hidden job uses the `$CCFM_CONFIG` variable to run `ccfm init` with the correct config file for each job.

---

See [gitlab-ci/README.md](../README.md) for an overview of all GitLab CI patterns.
