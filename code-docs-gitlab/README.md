# Example: Microservice Architecture Docs — GitLab CI

> **Integration method**: Docker image via GitLab CI (`image: ghcr.io/stevesimpson418/ccfm-convert:latest`)

"Order Service" is a fictional microservice. Architecture decision records (ADRs), API
documentation, and runbooks live alongside the service code. The CCFM deploy pipeline
runs inside the service's existing GitLab CI workflow, using the ccfm-convert Docker
image as a native GitLab CI job runner.

## What this demonstrates

- **GitLab CI native pipeline** — no GitHub Actions; ccfm runs directly inside the Docker
  image as a GitLab job
- **Markdown linting** — `lint:markdown` job catches formatting issues before they reach
  Confluence, using `markdownlint-cli2` with CCFM-aware rule overrides
- **Informational plan check** — `plan:confluence` runs on every MR touching `docs/**`;
  shows what Confluence pages would change (exits 0 by default since v0.2.0)
- **Multi-environment deployment** — separate staging and production Confluence spaces,
  driven by GitLab's native Environments feature
- **Per-environment space keys** — `CONFLUENCE_SPACE` is a single CI/CD variable scoped
  to `ENG-STAGE` for staging and `ENG` for production; one `ccfm.yaml`, zero duplication
- **Remote state management** — state is stored in Confluence (v0.3.0+), eliminating
  local state files, artifact handoff, and commit-back jobs

## Prerequisites

1. Two Confluence Cloud spaces: one for staging (e.g. `ENG-STAGE`) and one for production
   (`ENG`)
2. An Atlassian API token with write access to both spaces

## GitLab CI/CD variables required

Configure these in **Settings → CI/CD → Variables**:

| Variable | Scope | Value |
| --- | --- | --- |
| `CONFLUENCE_DOMAIN` | All environments | e.g. `acme.atlassian.net` |
| `CONFLUENCE_EMAIL` | All environments | Your Atlassian account email |
| `CONFLUENCE_TOKEN` | All environments | Atlassian API token (mark as **Masked**) |
| `CONFLUENCE_SPACE` | `staging` environment | e.g. `ENG-STAGE` |
| `CONFLUENCE_SPACE` | `production` environment | e.g. `ENG` |

### How to scope `CONFLUENCE_SPACE` per environment

1. Go to **Settings → CI/CD → Variables → Add variable**
2. Set **Key** to `CONFLUENCE_SPACE`, **Value** to `ENG-STAGE`
3. Under **Environment scope**, select `staging`
4. Repeat for `ENG` scoped to `production`

GitLab injects the correct value automatically based on the `environment:` declared in
each job.

## Pipeline stages

### `lint` — `lint:markdown`

Runs on MRs and default branch pushes when `docs/**` changes. Installs
`markdownlint-cli2` and lints all docs. Configuration in `.markdownlint.yaml` disables
rules that conflict with CCFM syntax (expand blocks, frontmatter, bold in tables).

### `plan` — `plan:confluence`

MR-only. Runs `ccfm --plan` using the ccfm-convert Docker image. Shows what Confluence
pages would be created, updated, or archived if the MR were merged.

**Informational**: since v0.2.0, `--plan` exits 0 by default. The output shows what
Confluence pages would change without blocking the MR pipeline.

### `deploy` — `deploy:staging` + `deploy:production`

- **`deploy:staging`**: Runs automatically on push to the default branch. Deploys to the
  staging Confluence space. State is managed remotely in Confluence (v0.3.0+).
- **`deploy:production`**: `when: manual` — appears as a play button in the pipeline UI
  after staging. Trigger it once you're confident the staging output looks correct.

## Triggering a production deploy

1. Merge a docs change to the default branch
2. `deploy:staging` runs automatically — verify the result in the staging Confluence space
3. Navigate to **CI/CD → Pipelines** and open the pipeline for your merge commit
4. Click the play button next to `deploy:production`

## Running locally (dry run)

```bash
pip install ccfm-convert
export CONFLUENCE_DOMAIN=acme.atlassian.net
export CONFLUENCE_EMAIL=you@acme.com
export CONFLUENCE_TOKEN=your-token
export CONFLUENCE_SPACE=ENG-STAGE

# Preview what would deploy — no Confluence changes made
ccfm --config ccfm.yaml --directory docs --plan

# Deploy to staging
ccfm --config ccfm.yaml --directory docs --changed-only --archive-orphans
```

Or with Docker (matches what CI does exactly):

```bash
docker run --rm \
  -e CONFLUENCE_DOMAIN=acme.atlassian.net \
  -e CONFLUENCE_EMAIL=you@acme.com \
  -e CONFLUENCE_TOKEN=$CONFLUENCE_TOKEN \
  -e CONFLUENCE_SPACE=ENG-STAGE \
  -v $(pwd)/docs:/workspace/docs \
  -v $(pwd)/ccfm.yaml:/workspace/ccfm.yaml \
  -w /workspace \
  ghcr.io/stevesimpson418/ccfm-convert:latest \
  ccfm --config ccfm.yaml --directory docs --plan
```
