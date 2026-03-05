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
- **Blocking plan gate** — `plan:confluence` runs on every MR touching `docs/**`; exit
  code 2 fails the MR pipeline, forcing intentional doc review before merge
- **Multi-environment deployment** — separate staging and production Confluence spaces,
  driven by GitLab's native Environments feature
- **Per-environment space keys** — `CONFLUENCE_SPACE` is a single CI/CD variable scoped
  to `ENG-STAGE` for staging and `ENG` for production; one `ccfm.yaml`, zero duplication
- **Separate state files** — `.ccfm-state.staging.json` and `.ccfm-state.production.json`
  prevent conflicts between environments
- **Artifact-based state handoff** — the updated state file is passed from deploy job to
  commit-back job via GitLab artifacts (no shared filesystem needed)
- **`[skip ci]` commit-back** — state file updates are committed via `CI_JOB_TOKEN`
  without triggering a new pipeline

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

**Blocking**: exit code 2 (pending changes) fails the MR pipeline. This forces the team
to review the Confluence impact of every doc change before merging.

### `deploy` — `deploy:staging` + `deploy:production`

- **`deploy:staging`**: Runs automatically on push to the default branch. Deploys to the
  staging Confluence space. Updated state file is exported as a GitLab artifact.
- **`deploy:production`**: `when: manual` — appears as a play button in the pipeline UI
  after staging. Trigger it once you're confident the staging output looks correct.

### `update-state` — `update-state:staging` + `update-state:production`

Commits the updated state file back to the repository after each deploy. Uses
`alpine/git` (the ccfm-convert image has no `git`). The state file is downloaded from
the deploy job's artifact automatically via `needs: [{job: ..., artifacts: true}]`.

## Triggering a production deploy

1. Merge a docs change to the default branch
2. `deploy:staging` runs automatically — verify the result in the staging Confluence space
3. Navigate to **CI/CD → Pipelines** and open the pipeline for your merge commit
4. Click the play button next to `deploy:production`
5. `update-state:production` runs automatically after a successful production deploy

## Running locally (dry run)

```bash
pip install ccfm-convert
export CONFLUENCE_DOMAIN=acme.atlassian.net
export CONFLUENCE_EMAIL=you@acme.com
export CONFLUENCE_TOKEN=your-token
export CONFLUENCE_SPACE=ENG-STAGE

# Preview what would deploy — no Confluence changes made
ccfm --config ccfm.yaml --directory docs --state .ccfm-state.staging.json --plan

# Deploy to staging
ccfm --config ccfm.yaml --directory docs --state .ccfm-state.staging.json --changed-only --archive-orphans
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
  -v $(pwd)/.ccfm-state.staging.json:/workspace/.ccfm-state.staging.json \
  -w /workspace \
  ghcr.io/stevesimpson418/ccfm-convert:latest \
  ccfm --config ccfm.yaml --directory docs --state .ccfm-state.staging.json --plan
```
